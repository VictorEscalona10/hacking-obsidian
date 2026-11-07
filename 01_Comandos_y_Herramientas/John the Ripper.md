**John the Ripper** (JTR) es una herramienta de línea de comandos de código abierto para **crackear contraseñas de forma offline**. Recibe uno o varios hashes, prueba candidatos de un diccionario (o generados por reglas, máscaras o fuerza bruta) y compara el resultado hasta encontrar la contraseña original.

A diferencia de [[Hydra]], John **no ataca servicios en red**: trabaja sobre el hash que ya tienes en tu máquina. Eso lo hace mucho más rápido y sigiloso, porque no genera tráfico ni intentos de login.

---

## Para que sirve en hacking

- Recuperar la contraseña a partir de un hash obtenido en una brecha, una base de datos o un CMS.
- Crackear los hashes de `/etc/shadow` tras conseguir acceso a un sistema Linux.
- Descifrar la **passphrase de una clave privada** de [[SSH]] (por ejemplo `id_rsa`) con `ssh2john`.
- Abrir archivos protegidos con contraseña (ZIP, RAR, PDF, KeePass, documentos de Office).
- Reutilizar credenciales encontradas en un equipo para pivotar a otros servicios.

> Solo debe usarse contra sistemas propios o con autorización explícita por escrito.

---

## Instalación

```bash
sudo apt update
sudo apt install john
```

En Kali viene preinstalada la versión **Jumbo**, que incluye los scripts `*2john` y muchísimos formatos. Para comprobarlo:

```bash
john --list=build-info
john --list=formats | head
```

Si el comando `ssh2john` no existe, en algunas distribuciones el script se llama `ssh2john.py` o está en `/usr/share/john/`.

---

## Como funciona

1. John lee el hash (o varios) desde un archivo, una línea por hash.
2. Identifica el **formato** del hash o se lo indicas con `--format=`.
3. Genera candidatos según el modo elegido: diccionario, reglas, máscara o incremental.
4. Calcula el hash de cada candidato y lo compara con el objetivo.
5. Cuando coincide, guarda el resultado en `john.pot` y lo muestra con `--show`.

La contraseña **no se descifra**: se adivina probando candidatos. Por eso la velocidad y el diccionario importan tanto.

---

## Diccionarios

El diccionario más usado en laboratorios es `rockyou.txt`:

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
ls -lh /usr/share/wordlists/rockyou.txt
```

También se pueden usar listas de **SecLists**, las mismas que se usan con [[FFUF]]:

```bash
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt
/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt.tar.gz
```

---

## Sintaxis y parametros

```bash
john [opciones] <archivo_con_hashes>
john --wordlist=<diccionario> <archivo_con_hashes>
```

### Opciones principales

| Parámetro | Descripción |
| :--- | :--- |
| `--wordlist=<archivo>` | Diccionario de contraseñas (una por línea) |
| `--format=<tipo>` | Fuerza el formato del hash (ver tabla abajo) |
| `--rules[=<conjunto>]` | Aplica reglas de mutación al diccionario |
| `--incremental[=<modo>]` | Fuerza bruta por conjuntos de caracteres |
| `--mask=<patrón>` | Fuerza bruta con un patrón concreto |
| `--single` | Prueba variaciones del nombre de usuario |
| `--show` | Muestra las contraseñas ya crackeadas |
| `--list=formats` | Lista todos los formatos soportados |
| `--list=build-info` | Muestra la versión y las capacidades de John |
| `--session=<nombre>` | Guarda la sesión para poder retomarla |
| `--restore=<nombre>` | Restaura una sesión anterior |
| `--pot=<archivo>` | Usa un archivo `.pot` distinto del habitual |
| `--fork=<n>` | Usa varios procesos en paralelo |
| `--test` | Benchmark de velocidad de cracking |
| `--stdout` | Imprime los candidatos generados en vez de crackear |

### Archivo de hashes

El archivo puede contener un hash por línea:

```text
5f4dcc3b5aa765d61d8327deb882cf99
```

O con nombre de usuario delante, lo que ayuda a identificar cada resultado:

```text
mitch:0c01f4468bd75d7a84c7eb73846e8d96
```

---

## Formatos de hash

Si John no detecta el formato o muestra `No password hashes loaded`, hay que indicarlo manualmente. Primero se puede identificar el hash con `hash-identifier` o siguiendo [[MD5#Identificar un hash|la guía de MD5]].

| Formato | Uso típico |
| :--- | :--- |
| `raw-md5` | MD5 sin sal (32 caracteres hex) |
| `raw-sha1` | SHA1 sin sal (40 caracteres hex) |
| `raw-sha256` | SHA256 sin sal (64 caracteres hex) |
| `dynamic_4` | MD5 con sal en formato `md5($salt.$pass)` |
| `nt` | Hash NTLM de Windows |
| `sha512crypt` | Hashes de `/etc/shadow` en Linux moderno (`$6$...`) |
| `bcrypt` | Hashes `$2a$`, `$2b$`, `$2y$` (muy lentos) |
| `ssh` | Clave privada de [[SSH]] convertida con `ssh2john` |

Ver todos los formatos disponibles:

```bash
john --list=formats | tr ',' '\n' | grep -i md5
john --list=formats | tr ',' '\n' | grep -i ssh
```

---

## Uso basico

### Diccionario

El caso más habitual: hash + `rockyou.txt`.

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Ver el resultado:

```bash
john --show --format=raw-md5 hash.txt
```

### Diccionario con reglas

Las reglas mutan cada palabra del diccionario (mayúsculas, números al final, sustituciones tipo `a` → `@`, etc.):

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=best64 hash.txt
```

### Fuerza bruta incremental

Prueba todas las combinaciones de un conjunto de caracteres:

```bash
john --incremental hash.txt
john --incremental=Digits hash.txt
```

### Máscara

Más rápido que incremental cuando se conoce la estructura de la contraseña:

```bash
john --mask='?u?l?l?l?l?d?d' hash.txt
```

| Símbolo | Conjunto |
| :--- | :--- |
| `?l` | Letras minúsculas |
| `?u` | Letras mayúsculas |
| `?d` | Dígitos |
| `?s` | Símbolos |
| `?a` | Todo lo anterior |

### Single

Prueba variaciones del nombre de usuario y datos del `GECOS` de `/etc/passwd`:

```bash
john --single hashes.txt
```

### Sesiones y reanudación

Para ataques largos conviene guardar la sesión:

```bash
john --session=shadow1 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --restore=shadow1
```

### Generar candidatos sin crackear

Útil para ver qué está probando John o para pasarlos a otra herramienta:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules --stdout | head
```

---

## Preparar hashes con los scripts *2john

John no puede leer directamente archivos protegidos: primero hay que extraer el hash con un script auxiliar.

### Clave privada de SSH

```bash
ssh2john id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

Es el flujo que se usa en [[Basic Pentesting]] para descifrar la passphrase de la clave de `kay` y entrar por [[SSH]].

### `/etc/shadow`

Se combinan `passwd` y `shadow` con `unshadow`:

```bash
unshadow /etc/passwd /etc/shadow > hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt
```

Normalmente John detecta solo el formato (`sha512crypt` en distribuciones modernas), pero se puede forzar:

```bash
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

### Archivos comprimidos y documentos

```bash
zip2john archivo.zip > zip.hash
rar2john archivo.rar > rar.hash
pdf2john archivo.pdf > pdf.hash
keepass2john database.kdbx > keepass.hash
office2john documento.docx > office.hash
```

Después se crackean igual que cualquier otro hash:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash
```

### Hashes de Windows

```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm.txt
```

---

## Casos practicos del vault

- **[[Simple CTF]]**: el exploit del CMS devuelve un hash MD5 con salt; se crackea con `--format=dynamic_4` y la contraseña resulta ser `secret`.
- **[[Basic Pentesting]]**: se copia el `id_rsa` de `kay`, se convierte con `ssh2john` y se descifra la passphrase con `rockyou.txt`.

---

## Rendimiento y GPU

- John Jumbo puede usar **OpenCL** para aprovechar la GPU. Comprobar si está disponible:

```bash
john --list=build-info | grep -i opencl
```

- Para repartir la carga entre varios núcleos de CPU:

```bash
john --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

- Para ver una estimación de velocidad antes de lanzar el ataque:

```bash
john --test
```

- En general, **hashcat** es más rápido con GPU, pero John es más cómodo para formatos raros y scripts `*2john`.

---

## Errores y problemas comunes

- **`No password hashes loaded`**: el formato no se detectó o el archivo está mal formado. Probar `--format=` con el tipo correcto.
- **`No password hashes left to crack`**: esa contraseña ya está en `john.pot`. Usar `--show` para verla.
- **Va muy lento**: el formato puede ser costoso (`bcrypt`, `sha512crypt`) o el ataque es demasiado amplio. Acotar con `--mask`, reglas o un diccionario mejor.
- **`command not found: ssh2john`**: usar `ssh2john.py` o la ruta completa dentro de `/usr/share/john/`.
- **Resultado inesperado**: revisar que el hash no tenga salt ni campos extra; en ese caso hay que ajustar el `--format` (`dynamic_4`, `dynamic_10`, etc.).

---

## Alternativas y herramientas complementarias

| Herramienta | Uso |
| :--- | :--- |
| `john` | Crackeo offline, muchos formatos y scripts `*2john` |
| `hashcat` | Crackeo offline muy rápido con GPU |
| [[Hydra]] | Fuerza bruta **online** contra servicios de red |
| `hash-identifier` | Identificar el tipo de hash |
| `unshadow` | Combinar `passwd` y `shadow` para John |

```bash
# Mismo ataque con hashcat (MD5 sin sal)
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

---

## Ver también

- [[MD5]] - Formato de hash más común y cómo identificarlo.
- [[SSH]] - Servicio cuyas claves privadas se descifran con `ssh2john`.
- [[Hydra]] - Fuerza bruta online contra servicios, complemento de John.
- [[Nmap]] - Descubrir servicios antes de buscar hashes o credenciales.
- [[Basic Pentesting]] - Caso práctico con `id_rsa` y `ssh2john`.
- [[Simple CTF]] - Caso práctico con hash MD5 con salt (`dynamic_4`).
