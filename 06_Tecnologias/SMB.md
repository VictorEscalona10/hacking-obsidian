SMB (Server Message Block) es el protocolo que usan las Windows para **compartir carpetas, archivos e impresoras** en red. Es como una carpeta compartida, pero a nivel de red.

## Herramientas 

### smbmap

```bash
smbmap -H IP
```

```bash
smbmap -H IP -u 'usuario' -p 'contrasena' 
```

```bash
smbmap -H IP -u 'usuario' -p 'contrasena' -r carpeta
```

```bash
smbmap -H IP -u 'usuario' -p 'contrasena' --download carpeta/archivo.txt
```





```bash
smbclient -L //<IP> -N
```

- `-L` = listar recursos compartidos.
- `-N` = sin contraseña (null session).