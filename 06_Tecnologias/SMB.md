SMB (Server Message Block) es el protocolo que usan las Windows para **compartir carpetas, archivos e impresoras** en red. Es como una carpeta compartida, pero a nivel de red.

```bash
smbclient -L //<IP> -N
```

- `-L` = listar recursos compartidos.
- `-N` = sin contraseña (null session).