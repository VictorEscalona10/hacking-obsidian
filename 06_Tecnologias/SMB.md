SMB (Server Message Block) es el protocolo que usan las Windows para **compartir carpetas, archivos e impresoras** en red. Es como una carpeta compartida, pero a nivel de red.

Tambien lo implementa **Samba** en Linux/Unix, asi que no es exclusivo de Windows: un servidor Linux puede exponer recursos SMB igual que un Windows.

## Como funciona

- Es **cliente-servidor**: el cliente pide un recurso (una carpeta, un archivo, una impresora) y el servidor lo sirve despues de autenticar la sesion.
- La autenticacion puede ser **NTLM** (usuario/contraseña o hash) o **Kerberos** (en dominios Active Directory).
- Los permisos se aplican en dos capas: **permisos del share** (SMB) y **permisos NTFS** (sistema de archivos). Puedes ver un share pero no poder leer todo su contenido.
- Puerto principal: **445/TCP** (SMB directo). El **139/TCP** es SMB sobre NetBIOS (legacy) y **137/138 UDP** se usan para resolucion de nombres.

| Version         | Donde aparece                  | Notas                                          |
| :-------------- | :----------------------------- | :--------------------------------------------- |
| **SMB1 / CIFS** | Windows XP/2003, NAS viejos    | Inseguro, origen de **EternalBlue (MS17-010)** |
| **SMB2**        | Windows Vista/2008 en adelante | Mejor rendimiento, menos chatter               |
| **SMB2.1**      | Windows 7/2008 R2              |                                                |
| **SMB3**        | Windows 8/2012 en adelante     | Cifrado y signing                              |
| **SMB3.1.1**    | Windows 10/2016 en adelante    | AES-GCM, preauth integrity                     |

## Como saber que hay SMB

```bash
# Puertos y version
nmap -p139,445 -sV -sC IP

# Info del servicio, dialectos y firma
nmap -p445 --script smb-os-discovery,smb-protocols,smb-security-mode,smb2-security-mode IP
```

Indicios:

- **445 abierto** (y normalmente tambien 139) en un host Windows.
- Suele venir acompañado de **135** (RPC), **5985** (WinRM) y puertos altos de RPC.
- `netexec smb IP` o `smbclient -L //IP -N` responden con nombre de equipo, dominio y shares.
- En Linux, un **Samba** expuesto se ve igual: 139/445 y shares tipo `print$`, `homes`, `shared`.

## Herramientas

### smbclient

Listar recursos compartidos:

```bash
smbclient -L //<IP> -N
```

- `-L` = listar recursos compartidos.
- `-N` = sin contraseña (null session).

Con credenciales:

```bash
smbclient -L //IP -U 'usuario%contraseña'
smbclient -L //IP -U 'DOMINIO/usuario%contraseña'
```

Conectarse a un share:

```bash
smbclient //IP/share -N
smbclient //IP/share -U 'usuario%contraseña'
smbclient //IP/share -U 'DOMINIO/usuario%contraseña'
```

Dentro del share:

| Comando         | Descripcion                        |
| :-------------- | :--------------------------------- |
| `ls`            | Listar archivos y carpetas         |
| `cd <carpeta>`  | Cambiar de directorio remoto       |
| `pwd`           | Ver la ruta actual                 |
| `get <archivo>` | Descargar un archivo               |
| `put <archivo>` | Subir un archivo                   |
| `recurse ON`    | Activar recursion en `ls`/`mget`   |
| `prompt OFF`    | No preguntar por cada archivo      |
| `mget *`        | Descargar todo el contenido actual |
| `exit`          | Salir                              |

Descargar todo de una sola vez:

```bash
smbclient //IP/share -N -c 'recurse ON; prompt OFF; mget *'
```

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

Variantes utiles:

```bash
# Null session / invitado
smbmap -H IP -u '' -p ''
smbmap -H IP -u 'guest' -p ''

# Listar de forma recursiva todos los shares
smbmap -H IP -u 'usuario' -p 'contrasena' -R

# Subir un archivo (si hay permiso de escritura)
smbmap -H IP -u 'usuario' -p 'contrasena' --upload local.txt 'carpeta/remoto.txt'

# Ejecutar un comando (requiere privilegios de administrador)
smbmap -H IP -u 'administrador' -p 'contrasena' -x 'whoami'
```

La salida marca cada share como **READ ONLY** o **READ, WRITE**: lo importante no es solo listar, sino ver donde puedes **leer** y donde puedes **escribir**.

### netexec / crackmapexec

`netexec` (comando corto `nxc`) es el sucesor de `crackmapexec`; la sintaxis es practicamente la misma.

```bash
netexec smb IP
nxc smb IP
```

Enumeracion sin credenciales:

```bash
netexec smb IP -u '' -p '' --shares
netexec smb IP -u 'guest' -p '' --shares
```

Con credenciales:

```bash
netexec smb IP -u 'usuario' -p 'contraseña' --shares
netexec smb IP -u 'usuario' -p 'contraseña' --users
netexec smb IP -u 'usuario' -p 'contraseña' --groups
netexec smb IP -u 'usuario' -p 'contraseña' --pass-pol
netexec smb IP -u 'usuario' -p 'contraseña' --sessions
netexec smb IP -u 'usuario' -p 'contraseña' --loggedon-users
netexec smb IP -u 'usuario' -p 'contraseña' --disks
```

- `--shares` lista shares y permisos.
- `--users`, `--groups`, `--pass-pol` y `--sessions` pueden requerir privilegios altos.
- `--loggedon-users` muestra quien tiene sesion abierta (util para buscar objetivos).
- `--disks` lista las unidades locales del host.

Ejecucion y reutilizacion de credenciales:

```bash
# Ejecutar un comando (requiere admin)
netexec smb IP -u 'administrador' -p 'contraseña' -x 'whoami'

# Pass-the-Hash: usar el hash NT en vez de la contraseña
netexec smb IP -u 'administrador' -H 'NTHASH' --shares

# Probar muchas credenciales sin parar en el primer acierto
netexec smb IP -u usuarios.txt -p contraseñas.txt --continue-on-success
```

### enum4linux-ng

Enumera de golpe sistema operativo, usuarios, grupos, shares, politica de contraseñas e impresoras:

```bash
enum4linux-ng -A IP
enum4linux-ng -A IP -u 'usuario' -p 'contraseña'
enum4linux-ng -A IP -u 'usuario' -p 'contraseña' -oY salida.yaml
```

- `-A` = toda la enumeracion simple.
- `-oY` / `-oJ` = guardar la salida en YAML/JSON.

La version antigua sigue existiendo:

```bash
enum4linux -a IP
```

### rpcclient

Se conecta por RPC sobre SMB y permite consultar el dominio/usuario con comandos interactivos:

```bash
rpcclient -U '' -N IP
rpcclient -U 'usuario%contraseña' IP
```

Comandos utiles dentro de la shell:

| Comando | Descripcion |
| :--- | :--- |
| `srvinfo` | Informacion del servidor |
| `enumdomusers` | Listar usuarios del dominio (incluye RID) |
| `enumdomgroups` | Listar grupos del dominio |
| `querydominfo` | Informacion del dominio |
| `getdompwinfo` | Politica de contraseñas (longitud, bloqueo, caducidad) |
| `netshareenumall` | Listar todos los shares |
| `netsharegetinfo <share>` | Detalle y permisos de un share |
| `lookupnames <usuario>` | Resolver un nombre a SID |
| `queryuser <RID>` | Detalle de un usuario por RID |
| `querygroup <RID>` | Detalle de un grupo por RID |
| `enumalsgroups builtin` | Grupos locales integrados |

Ejemplo: si `enumdomusers` devuelve `usuario 0x1f4`, se consulta con `queryuser 0x1f4`.

### Impacket

```bash
# Cliente SMB interactivo
impacket-smbclient usuario:contraseña@IP
impacket-smbclient -hashes :NTHASH usuario@IP

# Enumerar usuarios y grupos por SID
impacket-lookupsid usuario:contraseña@IP
impacket-lookupsid -no-pass usuario@IP

# Servidor SMB temporal (para compartir archivos o capturar hashes)
impacket-smbserver share . -smb2support
```

Ejecucion remota con credenciales validas:

```bash
impacket-psexec usuario:contraseña@IP
impacket-wmiexec usuario:contraseña@IP
impacket-smbexec usuario:contraseña@IP
```

> En instalaciones antiguas los scripts aparecen como `smbclient.py`, `lookupsid.py`, `psexec.py`, etc.; el prefijo `impacket-` es el actual.

### Nmap

```bash
# Version y scripts por defecto
nmap -p139,445 -sV -sC IP

# Info del servicio, dialectos y firma
nmap -p445 --script smb-os-discovery,smb-protocols,smb-security-mode,smb2-security-mode IP

# Enumeracion de shares, usuarios, grupos, sesiones y dominio
nmap -p445 --script smb-enum-shares,smb-enum-users,smb-enum-groups,smb-enum-sessions,smb-enum-domains IP

# Enumeracion autenticada
nmap -p445 --script smb-enum-shares,smb-enum-users \
  --script-args smbusername='usuario',smbpassword='contraseña' IP

# Vulnerabilidades conocidas
nmap -p445 --script smb-vuln-ms17-010,smb-vuln-ms08-067 IP

# Fuerza bruta (usar con cuidado: puede bloquear cuentas)
nmap -p445 --script smb-brute --script-args userdb=usuarios.txt,passdb=contraseñas.txt IP
```

`smb-enum-users` puede generar bastante ruido y bloquear cuentas; mejor probarlo primero en un laboratorio.

### Metasploit

| Modulo | Uso |
| :--- | :--- |
| `auxiliary/scanner/smb/smb_version` | Version y sistema operativo |
| `auxiliary/scanner/smb/smb_enumshares` | Enumerar shares |
| `auxiliary/scanner/smb/smb_enumusers` | Enumerar usuarios |
| `auxiliary/scanner/smb/smb_login` | Probar credenciales |
| `auxiliary/scanner/smb/smb_lookupsid` | Enumerar SIDs |
| `exploit/windows/smb/ms17_010_eternalblue` | Exploit de EternalBlue (solo si es vulnerable) |

```bash
msfconsole -q
use auxiliary/scanner/smb/smb_version
set RHOSTS IP
run
```

## Que enumerar (resumen)

| Que buscar | Herramienta |
| :--- | :--- |
| Nombre de equipo, dominio y SO | `nmap --script smb-os-discovery`, `netexec smb IP` |
| Version/dialecto de SMB | `nmap --script smb-protocols`, `netexec smb IP` |
| Shares y permisos | `smbclient -L`, `smbmap`, `netexec --shares`, `enum4linux-ng` |
| Usuarios y grupos | `rpcclient`, `enum4linux-ng`, `netexec --users/--groups` |
| Politica de contraseñas | `rpcclient getdompwinfo`, `netexec --pass-pol` |
| Sesiones abiertas | `netexec --sessions`, `nmap --script smb-enum-sessions` |
| Contenido de los shares | `smbclient`, `smbmap --download`, montaje CIFS |
| Vulnerabilidades | `nmap --script smb-vuln-*`, `searchsploit` |

## Shares comunes

| Share | Que suele contener |
| :--- | :--- |
| `IPC$` | Pipes con nombre; se usa para enumeracion y sesiones nulas |
| `ADMIN$` | `C:\Windows`; solo administradores |
| `C$` | La unidad `C:` completa; solo administradores |
| `NETLOGON` | Scripts de inicio de sesion del dominio |
| `SYSVOL` | GPOs y politicas del dominio (aqui aparecen credenciales en GPP) |
| `Users`, `Shared`, `Data`, `Backup`, `IT` | Datos de usuarios, backups y documentacion |

Los shares `ADMIN$`, `C$` y `IPC$` existen por defecto en Windows y aparecen en `net share`, pero normalmente solo los administradores pueden entrar. En la enumeracion SMB se listan aunque el usuario no tenga permiso de acceso.

## Acceso, descarga y montaje

```bash
# Descargar un archivo con smbclient
smbclient //IP/share -U 'usuario%contraseña' -c 'get archivo.txt'

# Descargar todo el share
smbclient //IP/share -U 'usuario%contraseña' -c 'recurse ON; prompt OFF; mget *'

# Descargar con smbmap
smbmap -H IP -u 'usuario' -p 'contraseña' --download 'carpeta/archivo.txt'
```

Montar el share en Linux:

```bash
sudo mkdir -p /mnt/share
sudo mount -t cifs //IP/share /mnt/share \
  -o username='usuario',password='contraseña',domain=WORKGROUP,vers=3.0

# Como invitado
sudo mount -t cifs //IP/share /mnt/share -o guest,uid=1000
```

En Windows:

```powershell
net use \\IP\share /user:usuario contraseña
```

## Fuerza bruta y reutilizacion de credenciales

Antes de lanzar un ataque, revisar la politica de contraseñas (`netexec --pass-pol`) para no bloquear cuentas.

```bash
# Hydra contra SMB
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt smb://IP -t 4 -f

# netexec probando usuario/contraseña en paralelo
netexec smb IP -u usuarios.txt -p /usr/share/wordlists/rockyou.txt --continue-on-success
```

Con credenciales validas, volver a enumerar: **cada usuario puede ver shares distintos**. Tambien se puede probar **Pass-the-Hash** si se obtuvo el hash NT.

Ver [[Hydra]] para el resto de opciones y [[MD5]] para crackeo offline de hashes.

## Vulnerabilidades comunes

### 1. Null session / invitado

Si el servidor permite sesiones anonimas o la cuenta `guest`, se puede enumerar sin credenciales:

```bash
smbclient -L //IP -N
smbmap -H IP -u '' -p ''
enum4linux-ng -A IP
rpcclient -U '' -N IP
```

### 2. SMB signing deshabilitado (NTLM relay)

Si la firma no es obligatoria, un atacante puede retransmitir la autenticacion NTLM de otra maquina y hacerse pasar por ella:

```bash
nmap -p445 --script smb2-security-mode IP
# "message signing enabled but not required" -> relay posible

impacket-ntlmrelayx -tf objetivos.txt -smb2support
```

Se suele combinar con **Responder** para capturar la autenticacion. No ejecutar Responder y `ntlmrelayx` escuchando SMB a la vez en la misma interfaz.

### 3. SMBv1 y EternalBlue (MS17-010)

Solo afecta si **SMBv1 esta habilitado**:

```bash
nmap -p445 --script smb-vuln-ms17-010 IP
searchsploit ms17-010
```

### 4. SMBGhost (CVE-2020-0796)

Vulnerabilidad en la compresion de **SMBv3.1.1** (Windows 10 1903/1909 y Server equivalentes). Se detecta por version y con scanners especificos; no confundir con EternalBlue (ese es SMBv1).

### 5. Shares con permiso de escritura

Si `smbmap` marca **WRITE**, se pueden subir archivos. En entornos Windows esto permite ataques de captura de hashes (archivos `.scf`, `.lnk`, `.url`) o reemplazo de contenido que otro usuario ejecutara. Revisar siempre los shares escribibles.

### 6. SYSVOL / GPP en dominios

En `\\IP\SYSVOL` pueden aparecer `Groups.xml` con contraseñas `cpassword`:

```bash
netexec smb IP -u 'usuario' -p 'contraseña' -M gpp_password
netexec smb IP -u 'usuario' -p 'contraseña' -M gpp_autologin
```

### 7. Informacion sensible en los shares

Backups, scripts, configuraciones, claves SSH, bases de datos y documentacion interna. Un share de backups suele ser el camino mas rapido a credenciales. Si aparece una base de datos, revisar [[SQLITE 3]].

## Cuando tienes shell

En un servidor **Samba**:

```bash
cat /etc/samba/smb.conf
testparm -s
smbstatus
```

En un **Windows**:

```powershell
net share
Get-SmbShare
Get-SmbShareAccess -Name share
Get-SmbSession
```

Ahi se ven los shares reales, sus rutas y quien tiene acceso: informacion directa para moverte lateralmente o escalar privilegios.

## Flujo rapido (checklist)

1. **Detectar**: `nmap -p139,445 -sV -sC IP`.
2. **Probar sesion nula/invitado**: `smbclient -L //IP -N`, `netexec smb IP -u '' -p '' --shares`.
3. **Listar shares y permisos**: `smbmap -H IP -R`, `netexec smb IP ... --shares`.
4. **Enumerar usuarios, grupos y politica**: `enum4linux-ng -A IP`, `rpcclient`, `netexec --users --groups --pass-pol`.
5. **Entrar a los shares interesantes** y descargar todo lo que se pueda leer.
6. **Repetir con credenciales validas** (cada usuario ve cosas distintas) y probar Pass-the-Hash.
7. **Revisar vulnerabilidades**: MS17-010, SMBGhost, signing deshabilitado, shares con escritura.
8. **Fuerza bruta solo si esta justificado** y la politica de bloqueo lo permite.

```bash
# Ruta rapida
nmap -p139,445 -sV -sC IP
smbclient -L //IP -N
smbmap -H IP -u '' -p '' -R
enum4linux-ng -A IP
netexec smb IP -u '' -p '' --shares
nmap -p445 --script smb-vuln-ms17-010,smb2-security-mode IP
```

## Relacionados

- [[Nmap]] - Descubrir puertos, version y scripts de enumeracion SMB.
- [[Hydra]] - Fuerza bruta de credenciales SMB.
- [[Dancing]] - Ejemplo de null session y acceso a un share en HTB.
- [[Basic Pentesting]] - Maquina con 139/445 para practicar enumeracion.
- [[RCE]] - Vulnerabilidades de servicios como SMB que llevan a ejecucion remota.
- [[Escaladas de privilegios]] - Siguiente paso despues de entrar por un share.
- [[SQLITE 3]] - Que hacer si encuentras una base de datos en un share.
