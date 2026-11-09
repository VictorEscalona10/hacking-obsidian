---
title: Try Hack Me- Pickle Rick
date: 2026-11-08
platform: Try Hack Me
difficulty: Easy
os: Linux
tags:
  - robots
  - ReverseShell
  - RCE
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** ``
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash

```

**Puertos Abiertos:**
- 22, 80

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- nos encontramos con un login.php y un robots.txt

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- nos damos cuenta que en ell codigo de la pagina pricipal nos dan un usuaio en un comentario
- en el robots.txt simplemente nos dan unos caracteres, y llso probamos como contrase;a
- Nos lleva a un panel donde literalmente podemos ejecutar comandos

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Ejecutar comandos para intentar tener una shell en nuestra maquina
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
- si ejecutamos sudo -l nos dice que tenemos permiso de usarlo sin contrase;a ni nda

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