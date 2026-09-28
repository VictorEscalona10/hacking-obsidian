**git-dumper** es una herramienta de código abierto escrita en **Python** que sirve para **recuperar el repositorio Git completo de un sitio web** cuando la carpeta `.git/` quedó expuesta públicamente en el servidor. Con eso se reconstruye el historial del proyecto y se obtiene el **código fuente**, incluyendo ramas, commits borrados y archivos eliminados.

---

## Para que sirve en hacking

- Obtener el **código fuente** de la aplicación sin tener acceso al servidor.
- Encontrar **credenciales, API keys, tokens y secretos** olvidados en commits viejos (muchas veces se borran del código actual pero quedan en el historial).
- Descubrir **rutas, endpoints y parámetros** internos que no aparecen en el front-end.
- Localizar **archivos eliminados** (`.env`, `config.php`, `backup.sql`, etc.) que siguen existiendo en el historial.
- Detectar **credenciales hardcodeadas** comparando versiones anteriores del código.

Es una técnica de **reconocimiento pasivo sobre información mal configurada**, no explota el servidor: solo descarga lo que ya está público.

---

## Instalación

```bash
git clone https://github.com/arthaud/git-dumper
cd git-dumper
pip3 install -r requirements.txt
```

---

## Sintaxis y parámetros

```bash
python3 git_dumper.py <URL_del_target> <directorio_de_salida>
```

| Parámetro | Descripción |
| :--- | :--- |
| `<URL>` | URL base donde está expuesta la carpeta `.git` (ej: `http://10.10.10.10/`) |
| `<output_dir>` | Carpeta local donde se guarda el repositorio reconstruido |
| `-j <n>` / `--jobs <n>` | Número de hilos/hilos de descarga (más rápido) |
| `--debug` | Muestra peticiones y errores detallados |
| `--repair` | Repara el `.git` corrupto o incompleto (mezcla versiones de objetos) |
| `-q` / `--quiet` | Modo silencioso |

---

## Flujo de trabajo típico

### 1. Detectar que `.git` está expuesta

```bash
curl -sI http://IP/.git/ | head -5
curl -s http://IP/.git/HEAD
curl -s http://IP/.git/config
curl -s http://IP/.git/index
```

Si `HEAD` devuelve `ref: refs/heads/main` (o `master`) en vez de un 403/404, el repo está expuesto.

También se detecta con fuzzing de rutas:

```bash
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://IP/FUZZ -mc 200,403 -e .git
```

### 2. Descargar el repositorio

```bash
python3 git_dumper.py http://IP/ git-repo
```

Con hilos para acelerar:

```bash
python3 git_dumper.py -j 10 http://IP/ git-repo
```

### 3. Reparar si el repositorio quedó corrupto

```bash
python3 git_dumper.py --repair http://IP/ git-repo
```

### 4. Analizar el contenido descargado

```bash
cd git-repo
git log --oneline --all          # historial completo de commits
git status
ls -la
```

---

## Qué buscar después de dumpear

### Historial de commits y secretos

```bash
git log --all --oneline --stat                # todos los commits con archivos cambiados
git log --all -p -- .env                       # versiones anteriores del .env
git log --all -p -- '*.php' '*.js' '*.py'      # cambios en código
git log --all --diff-filter=D --name-only      # archivos borrados
git grep -i "password" $(git rev-list --all)   # buscar passwords en todo el historial
git grep -i -E "api[_-]?key|token|secret" $(git rev-list --all)
```

### Ver una versión antigua de un archivo

```bash
git show <commit>:ruta/archivo.php
git checkout <commit> -- ruta/archivo.php
```

### Recuperar archivos eliminados

```bash
git log --all --full-history -- "ruta/archivo.borrado"
git checkout <commit>^ -- ruta/archivo.borrado
```

### Diferencia entre dos commits

```bash
git diff <commit1> <commit2>
git diff <commit1> <commit2> -- ruta/archivo
```

### Información útil del repo

```bash
git branch -a                       # ramas (suelen tener código experimental)
git tag                             # etiquetas
git config --list                   # email del autor, URLs internas
git shortlog -sn                    # autores y nº de commits
```

---

## Herramientas complementarias

| Herramienta | Uso |
| :--- | :--- |
| `git-dumper` | Descarga el `.git` expuesto |
| `git-robber` / `trufflehog` | Buscan secretos en el historial automáticamente |
| `GitTools` (`DFINDER.sh`) | Encuenta repositorios `.git` expuestos |
| `gitleaks` | Detección de secretos en commits |
| `git log -p` | Revisión manual de cambios |

```bash
# Búsqueda automatizada de secretos en todo el historial
trufflehog git file://./git-repo
```

---

## Errores comunes

- **403 Forbidden en `.git/`**: el directorio existe pero está bloqueado; se pueden probar rutas sueltas (`/.git/config`, `/.git/objects/...`).
- **Repo incompleto/corrupto**: usar `--repair` o reconstruir los objetos a mano con `git fsck --full`.
- **`.git` desactivada pero quedó un volcado**: buscar `git.tar.gz`, `.git.bak`, `backup.zip` con fuzzing.
- **Rama `master` vs `main`**: probar ambas al clonar manualmente.
