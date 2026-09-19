**FFUF** (_Fuzz Faster U Fool_) es una herramienta de línea de comandos de código abierto escrita en el lenguaje Go, diseñada para realizar **fuzzing web** y enumeración de alta velocidad en servidores y aplicaciones para encontrar subdominios: 

---
## Parametros para los comandos

- `-w:` indicar la ruta del diccionario
- `-u:` indicar el dominio al que queremos hacer fuzzing
- `-e:` sirve para añadir extensiones de archivo específicas a cada palabra de tu diccionario
- `-mc 200,301` : Muestra únicamente estos códigos de estado (Match Code).
- `-H:` sirve para inyectar o modificar **cabeceras HTTP (Headers)** personalizadas en cada petición que envía la herramienta al servidor.
- `-c:` Activa colores para identificar códigos HTTP rápidamente.
- `-t 100:` activa los hilos de ejecucion para mayor velocidad

### Diccionarios utilizados

+  subdomains-top1million-5000.txt

```bash
ffuf -w {ruta} -u http://dominio.htb -H "Host: FUZZ.dominio.htb" -mc 200
```

### Búsqueda con extensiones específicas

Ideal para encontrar archivos de lógica de backend, rutas de API o respaldos olvidados en el servidor.

```bash
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://<IP>/FUZZ -e .php,.txt,.html,.bak,.js
```