FTP (File Transfer Protocol) es el estándar clásico diseñado específicamente para subir, descargar y gestionar archivos entre una computadora y un servidor a través de una red. Importante destacar que la información no va cifrada por este protocolo. Por defecto utiliza el puerto 21

## Como conectarse por FTP

Debemos conocer la ip de la maquina a la que queremos conectarnos

```bash
ftp {IP}
```

Nos pedira un usuario y una contraseña, **Siempre hay que probar** si colocamos como usuario `anonymous` y de contraseña lo dejamos vacio solo presionamos `ENTER` a ver que sucede

## Comandos mas comunes

| Comando | Descripcion |
| :--- | :--- |
| `ls` o `dir` | Listar archivos y directorios en el servidor remoto |
| `cd <directorio>` | Cambiar de directorio remoto |
| `pwd` | Ver la ruta actual en el servidor remoto |
| `binary` | Cambiar a modo binario (obligatorio para descargar ejecutables, zips, imágenes sin corromperlos) |
| `ascii` | Cambiar a modo texto |
| `get <archivo>` | Descargar un archivo del servidor a tu máquina local |
| `mget *` | Descargar múltiples archivos |
| `put <archivo>` | Subir un archivo local al servidor remoto |
| `mput *` | Subir múltiples archivos |
| `bye` o `exit` | Cerrar la sesión FTP |

