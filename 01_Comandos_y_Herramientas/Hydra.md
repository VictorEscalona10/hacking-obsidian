**Hydra** (THC-Hydra) es una herramienta de línea de comandos de código abierto que sirve para hacer **ataques de fuerza bruta y de diccionario** contra servicios de autenticación en red. Prueba combinaciones de usuario y contraseña a gran velocidad y soporta más de 50 protocolos (SSH, FTP, HTTP, SMB, RDP, MySQL, etc.).

---

## Para que sirve en hacking

- Recuperar **credenciales débiles** cuando ya conocemos un usuario válido (por ejemplo, enumerado en [[FTP - SFTP|FTP]] o en un writeup como [[Obsession]]).
- Probar **usuarios por defecto** (`admin`, `root`, `administrator`) contra paneles web.
- Validar la **política de contraseñas** de un servicio tras el escaneo con [[Nmap]].
- Automatizar el login en formularios **HTTP POST/GET** con mensajes de error conocidos.

No explota ninguna vulnerabilidad: solo prueba credenciales, así que depende de una **mala política de contraseñas** o de usuarios por defecto.

> Solo debe usarse contra sistemas propios o con autorización explícita por escrito.

---

## Instalación

```bash
sudo apt update
sudo apt install hydra
```

Versión con interfaz gráfica (opcional):

```bash
sudo apt install hydra-gtk
```

Comprobar que quedó instalada y ver los servicios soportados:

```bash
hydra -h
hydra -h | grep -A 20 "Supported services"
```

---

## Diccionarios

El diccionario más usado en laboratorios es `rockyou.txt`:

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
ls /usr/share/wordlists/
```

También se puede usar cualquier lista de **SecLists** (las mismas que se usan en [[FFUF]]):

```bash
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

---

## Sintaxis y parámetros

```bash
hydra [opciones] <objetivo> <servicio>
hydra -l <usuario> -P <diccionario> ssh://<IP>
```

### Usuarios y contraseñas

| Parámetro      | Descripción                                                             |
| :------------- | :---------------------------------------------------------------------- |
| `-l <user>`    | Un solo usuario conocido                                                |
| `-L <archivo>` | Lista de usuarios (uno por línea)                                       |
| `-p <pass>`    | Una sola contraseña                                                     |
| `-P <archivo>` | Diccionario de contraseñas                                              |
| `-C <archivo>` | Lista combinada con formato `usuario:contraseña`                        |
| `-e nsr`       | Prueba contraseña vacía (`n`), igual al usuario (`s`) e invertida (`r`) |

> `-l`/`-L` y `-p`/`-P` no se pueden mezclar entre sí: se elige una opción por cada lado.

### Comportamiento y velocidad

| Parámetro | Descripción |
| :--- | :--- |
| `-t <n>` | Hilos en paralelo (por defecto 16; bajar a 4 si el servicio se cae) |
| `-f` | Parar al encontrar el **primer** par válido (global) |
| `-F` | Parar al encontrar el primer par válido **en ese host** |
| `-V` | Mostrar cada intento (útil para depurar) |
| `-vV` | Modo muy verboso |
| `-q` | Modo silencioso |
| `-w <seg>` | Tiempo máximo de espera por respuesta |
| `-W <seg>` | Espera entre conexiones (evita saturar el servicio) |
| `-c <seg>` | Espera entre intentos de login (por hilo) |
| `-u` | Recorrer primero todos los usuarios (en vez de todas las contraseñas) |

### Red y salida

| Parámetro | Descripción |
| :--- | :--- |
| `-s <puerto>` | Puerto no estándar |
| `-S` | Conectar usando SSL/TLS |
| `-4` / `-6` | Forzar IPv4 / IPv6 |
| `-M <archivo>` | Lista de objetivos (una IP por línea) |
| `-o <archivo>` | Guardar el resultado en un archivo |
| `-b <formato>` | Formato de salida: `text`, `json`, `jsonv1` |
| `-R` | Restaurar una sesión anterior |
| `-I` | Ignorar el archivo de restauración y empezar de cero |
| `-d` | Modo debug |
| `-x` | Usar un proxy (`-x host:puerto`) |

---

## Servicios y ejemplos

### SSH

El caso más típico: ya tenemos el usuario y solo falta la contraseña.

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -f
```

Con varios usuarios:

```bash
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt ssh://<IP> -t 4 -f
```

Al terminar, entrar con lo aprendido en [[SSH]]:

```bash
ssh russoski@172.17.0.2
```

### FTP

```bash
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt ftp://<IP> -f
```

Si además hay sesión anónima, probarla antes de gastar tiempo (ver [[FTP - SFTP]]).

### SMB

```bash
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt smb://<IP> -t 4 -f
```

Credenciales válidas de [[SMB]] suelen abrir la puerta a `smbclient`, `psexec` o `netexec`.

### HTTP Basic Auth

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt http-get://<IP>/ -f
hydra -l admin -P /usr/share/wordlists/rockyou.txt https-get://<IP>/ -S -f
```

### Formulario HTTP POST

El módulo `http-post-form` se usa para paneles de login. Formato:

```text
"<ruta>:<parámetros>:<condición>"
```

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> \
  http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid credentials"
```

- `^USER^` y `^PASS^` son los marcadores donde Hydra inyecta cada credencial.
- `F=` contiene el texto que aparece cuando **falla** el login.
- `S=` contiene el texto (o el código) que aparece cuando **acierta**; se usa cuando el error no es claro.
- Si el formulario tiene más campos ocultos, hay que añadirlos a la cadena de parámetros.

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> \
  http-post-form "/login:user=^USER^&pass=^PASS^&csrf=abc123:F=Usuario incorrecto"
```

### Formulario HTTP GET

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> \
  http-get-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"
```

### Otros servicios habituales

| Servicio | Ejemplo |
| :--- | :--- |
| MySQL | `hydra -l root -P rockyou.txt mysql://<IP>` |
| PostgreSQL | `hydra -l postgres -P rockyou.txt postgres://<IP>` |
| MSSQL | `hydra -l sa -P rockyou.txt mssql://<IP>` |
| RDP | `hydra -l administrator -P rockyou.txt rdp://<IP> -t 1 -W 3` |
| VNC | `hydra -P rockyou.txt vnc://<IP>` |
| Telnet | `hydra -l root -P rockyou.txt telnet://<IP>` |
| SMTP | `hydra -l user -P rockyou.txt smtp://<IP>` |
| POP3 | `hydra -l user -P rockyou.txt pop3://<IP>` |
| IMAP | `hydra -l user -P rockyou.txt imap://<IP>` |
| LDAP | `hydra -l admin -P rockyou.txt ldap2://<IP>` |
| Redis | `hydra -P rockyou.txt redis://<IP>` |

### Ver la ayuda de un módulo

Antes de pelear con un formulario, consultar la sintaxis exacta del módulo:

```bash
hydra -U http-post-form
hydra -U http-get-form
hydra -U smb
```

---

## Flujo de trabajo típico

### 1. Enumerar el servicio

```bash
nmap -p- -sV -sC <IP>
```

Ver [[Nmap]] para las variantes de escaneo.

### 2. Saber si es viable

- ¿El servicio permite muchos intentos o hay **bloqueo de cuenta / fail2ban**?
- ¿Tenemos un **usuario válido**? Probar primero usuarios por defecto.
- ¿El login web tiene **CSRF token** o captcha? Hydra no lo resuelve solo.

### 3. Elegir diccionario y lanzar

```bash
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt ssh://<IP> -t 4 -f -V
```

### 4. Guardar el resultado

```bash
hydra -l <usuario> -P rockyou.txt ssh://<IP> -o hydra_ssh.txt -b json
```

### 5. Validar la credencial a mano

Nunca confiar solo en la salida de Hydra: probar el login y confirmar el acceso.

---

## Errores y problemas comunes

- **`[ERROR] target ssh://IP:22 does not support password authentication`**: el servidor solo acepta claves; Hydra no sirve aquí.
- **`Connection refused` o el servicio se cae**: bajar `-t` (a 1-4) y añadir `-W 3`.
- **`max connect retries exceeded`**: el servicio está filtrando o bloqueando; probar `-W` más alto o cambiar de estrategia.
- **Siempre devuelve la misma contraseña**: la condición `F=`/`S=` está mal elegida, o el formulario devuelve 200 con el error dentro. Usar `-V` y revisar la respuesta real.
- **Formulario con token CSRF**: Hydra no lo regenera; usar Burp Intruder o un script a medida.
- **Cuenta bloqueada / IP baneada**: reducir velocidad (`-t 1 -W 5 -c 10`) o usar `-u`; en entornos reales esto puede dejar el servicio inutilizable.

---

## Alternativas y herramientas complementarias

| Herramienta | Uso |
| :--- | :--- |
| `hydra` | Fuerza bruta en línea contra servicios |
| `medusa` | Alternativa a Hydra, modular y paralela |
| `ncrack` | Fuerza bruta de credenciales de red (Nmap suite) |
| `patator` | Framework modular para ataques de autenticación |
| `netexec` / `crackmapexec` | Validar y reutilizar credenciales en [[SMB]] |
| [[John the Ripper]] / `hashcat` | Crackeo **offline** de hashes (ver [[MD5]]) |
| `wpscan` | Fuerza bruta específica de WordPress |
| [[FFUF]] | Fuzzing de rutas y parámetros, no de credenciales |

```bash
# Ejemplo con medusa, mismo ataque contra SSH
medusa -h <IP> -u russoski -P /usr/share/wordlists/rockyou.txt -M ssh -t 4
```

---

## Ver también

- [[Nmap]] - Descubrir qué servicio y versión está expuesto antes de atacar.
- [[SSH]] - Conectarse con las credenciales obtenidas.
- [[FTP - SFTP]] - Servicio habitual donde aparece el usuario válido.
- [[SMB]] - Reutilizar credenciales en recursos compartidos.
- [[FFUF]] - Diccionarios y enumeración web complementaria.
