---
title: Try hack me- Room 404
date: YYYY-MM-DD
platform: Try Hack Me
difficulty: Very Easy
os: Linux
tags:
  - Git
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** `10.64.170.29`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.234.54 -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- 22 ([[SSH]]), 8080 (HTTP)

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- hay una ruta /.git oculta 

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Podemos utilizar herramientas como git-dumper para ver la estructura de la pagina que esta en git
- 

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. utilizamos la herramienta git-dumper 
2. nos metemos en los archivos y revisamos cada uno
3. Buscamos algun dato o brecha que nos sirva

### Ejecución
```bash
git-dumper http://10.64.170.29:8080/.git/ . 
```

**Prueba de acceso (User Flag):**
```bash
THM{byt3_l0tus_n3v3r_f0rg3ts}
```

---