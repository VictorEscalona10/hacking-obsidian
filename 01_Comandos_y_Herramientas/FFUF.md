**FFUF** (_Fuzz Faster U Fool_) es una herramienta de línea de comandos de código abierto escrita en el lenguaje Go, diseñada para realizar **fuzzing web** y enumeración de alta velocidad en servidores y aplicaciones para encontrar subdominios: 

---
## Parametros para los comandos

- **-w:** indicar la ruta del diccionario
- **-u:** indicar el dominio al que queremos hacer fuzzing
- **-mc:** filtrar si solo mostrar el codigo que le indiquemos
- **-H:** sirve para inyectar o modificar **cabeceras HTTP (Headers)** personalizadas en cada petición que envía la herramienta al servidor.
- 

### Diccionarios utilizados

+  subdomains-top1million-5000.txt

```bash
ffuf -w {ruta} -u http://dominio.htb -H "Host: FUZZ.dominio.htb" -mc 200
```

