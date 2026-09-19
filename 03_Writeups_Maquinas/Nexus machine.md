---
title: Hack the box- Nexus
date: 2026-09-17
platform: Hack the box
difficulty: Easy
os: Linux
tags:
  - RCE
  - ReverseShell
status: ""
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.113.87`
**Descripción breve:** 

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv [IP] -n -Pn -oN escaneo
```

**Puertos Abiertos:**
- 22, 80

### Enumeración de Servicios / Fuzzing

[[FFUF]]
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://nexus.htb -H "Host: FUZZ.nexus.htb"
```

**Descubrimientos clave:**
- se descubrio un subdominio git y nos llevo a una pagina de gitea, al revisar la pagina pudimos ver la documentacion de la api y llegar a un repositorio de github con unas credenciales

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
-  en la pagina principal esta este correo: j.mattew@nexus.htb
-  en el env que esta en el repositorio de github tenia esta contraseña: N27xh!!2ucY04
- en la ruta de [http://billing.nexus.htb/admin/dashboard](http://billing.nexus.htb/admin/login se colocan las credenciales y se llega al dashboard de la aplicacion

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. utilizar la vulnerabilidad CVE-2026-38526 para intentar hacer un [[RCE]]
2. intentar subir un archivo al servidor que me de la reverse shell y utilizar netcat para escuchar por el puerto 
3.  interceptamos la subida del archivo por la parte de email con burp suite y le cambiamos la extension png a php y vamos a la url vulnerable segun el CVE para ejecutar el reverse shell
4. Despues de ejecutar la reverse shell, nos da una consola y tenemos que movernos entre los usuarios para sacar las credenciales de jones, al final se encuentra un .env con la contraseña y probamos por ssh

### Ejecución
```bash
jones@10.129.113.87 passwd:y27xb3ha!!74GbR
```

**Prueba de acceso (User Flag):**
```bash 
 05eeee5ad2e82a3a5831bb7bfa2d4294
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