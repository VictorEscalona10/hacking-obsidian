---
title: Hack the box- Management
date: 2026-10-16
platform: Hack the box
difficulty: Easy
os: Linux
tags:
  - 
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.133.107`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.133.107 -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- 22 SSH, 80 HTTP, 443 HTTPS, 4444 krb524, 45851, 50389

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- 

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- 
- 

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. 
2. 
3. 

### Ejecución
```bash

```

**Prueba de acceso (User Flag):**
```bash

```

---

## 4. Privilege Escalation (Escalada de Privilegios)

### Enumeración Interna
- 

### Explotación Local
1. 
2. 
3. 

### Ejecución
```bash

```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash

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