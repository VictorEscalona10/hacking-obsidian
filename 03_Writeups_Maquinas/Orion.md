---
title: Hack the box- Orion
date: 2026-09-26
platform: Hack the box
difficulty: Easy
os: Linux
tags:
  - 
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.244.146`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash

```

**Puertos Abiertos:**
- 22 (SSH), 80 (HHTP)

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- endpoint /admin/login donde hay un formulario

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Es craftcms 5.6.16 esta version tiene un fallo de seguridad [[RCE]] 
- CVE-2025-32432

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