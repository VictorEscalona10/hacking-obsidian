---
title: Try Hack Me-Dreaming
date: 2026-11-10
platform: TryHackMe
difficulty: Easy
os: Linux
tags:
  - 
status: ""
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
- /app que nos lleva a un cms llamado pluck con una version super explicita 4.7.13

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Nos damos cuenta que podemos llegar a un apartado de adminisitrador, probando contrase;as, nos deja entrar con `password`, al entrar podemos ver un apartado de subida de archivos
- intentamos subir un archivo malicioso, pero este nos lo pone como .txt, entonces buscamos la vulnerabilidad para subir archivos maliciosos y nos descargamos un exploit

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Ejecutamos el exploit de CaelumIsMe con `python3 exploit.py 10.65.158.50 80 password /app/pluck-4.7.13`
2. nos deja un archivo subido al servidor con la extencion .phar y podemos ejecutar comandos
3. hacemos una reverse shell a nuestra maquina

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