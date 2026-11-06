---
title: Plataforma - Nombre de la Máquina
date: YYYY-MM-DD
platform: Try Hack Me
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
- 21, 80

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- /simple y nos da a un cms

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Revisando la plataforma CMS made simple 2.2.8, es vulnerable a un SQLInyection y ejecutamos un explot de internet, pero nos da la siguiente salida
```bash
[+] Salt for password: 1dac0d92e9fa6bb2

[+] Username: mitch

[+] Email: admin@admin.com

[+]Password hash: 0c01f4468bd75d7a84c7eb73846e8d96
```
- Entonces usamos john the ripper para descifrar la contrase;a
```bash
john --format=dynamic_4 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

- luego lo consultamos y nos damos cuenta que la contrase;a es "secret"
- entramos al panel de administracion y vemos que hay una partado donde podemos subir arhcivos
---

## 3. Exploitation (Acceso Inicial)

### Metodología
1.  Al intentar subir un archivo php, nos bloquea, entonces lo subimos como .phtm
2. insertamos un payload para ejecutar comandos remotamente
3. Introcducimos como usuario www-data, pero este usuario no tiene permisos de casi nada

### Ejecución
```bash
G00d j0b, keep up!
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