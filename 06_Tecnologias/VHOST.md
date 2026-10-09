Un **Virtual Host (vhost)** es un sitio web completo (dominio, config, archivos, backend) servido desde **el mismo servidor y la misma IP** que otros. El servidor decide cual devolver mirando el header **`Host:`** de la petición (y en HTTPS, el **SNI**).

Es la razón por la que `http://IP/` puede mostrar "it works" y `http://IP/otro.dominio.htb` muestre una app totalmente distinta.

## Tipos

| Tipo | Como decide | Ejemplo |
| :--- | :--- | :--- |
| **Name-based** | Header `Host:` | Lo estándar hoy; la mayoría de nginx/apache |
| **IP-based** | IP de destino | Un IP por sitio (poco usado, requiere muchas IP) |
| **SNI-based** | Campo SNI en el handshake TLS | Necesario para elegir cert en HTTPS multip sitio |

```nginx
# nginx: cada server block = un vhost
server { listen 80; server_name www.empresa.htb; root /var/www/web; }
server { listen 80; server_name api.empresa.htb; proxy_pass http://127.0.0.1:3000; }
server { listen 80 default_server; return 404; }   # "catch-all"
```

```apache
# apache: VirtualHost por IP:puerto, el Host elige
<VirtualHost *:80>
    ServerName admin.empresa.htb
    DocumentRoot /var/www/admin
</VirtualHost>
```

## Por que es un vector de ataque

- **El vhost por defecto** a menudo sirve el sitio "de mentira" y el real está oculto.
- Muchos vhosts **internos** (admin, api, staging, panel) quedan montados pero nunca se anuncian.
- Suelen tener **menos seguridad** que el sitio público (solo nginx/apache como barrera).
- Acceder a un vhost interno puede significar: saltarte el WAF, entrar a un panel de admin, llegar a un backend que solo confía en ese Host.
- Un `server_name` mal configurado con `default_server` puede **devolver el contenido de otro sitio** a quien manda un Host aleatorio (fuga de información).

## Como descubrir que hay un vhost

### Señales en la respuesta

```bash
curl -sI http://IP/
# - Host distinto => mismo contenido (ignora el Host) o 404/301 distinto (lo usa)
# - Headers raros: X-VHost, X-Backend-Server, X-Served-By, Via
# - Certificate errors / "no server available"

# Comparar respuestas con Host distintos
curl -s -H "Host: prueba" http://IP/ | wc -c
curl -s http://IP/ | wc -c
```

Indicios de que usa vhosts:

- Cambia el status code (200 vs 404 vs 302) según el `Host`.
- El cuerpo cambia de tamaño / de contenido.
- Aparece un mensaje tipo *"Unknown domain"*, *"No such virtual host"* o una web por defecto distinta.
- `Server: nginx` con un `Location` a un dominio concreto.

También se filtran en:

- **Certificados TLS**: el `Subject Alternative Name` lista todos los dominios del cert.

```bash
echo | openssl s_client -connect IP:443 -servername x 2>/dev/null | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"

# o en el navegador: candado -> certificado -> SAN
```

- **crt.sh / CT logs**: `https://crt.sh/?q=%.target.htb`
- **JS/CSS/HTML**: enlaces absolutos a `https://sub.dominio.htb`
- **Respuestas de la propia app** (aquí entra [[Cohort]]: un endpoint devolvió el nombre del vhost, ver [[Cohort]]).

### Fuzzing del header Host

```bash
# ffuf - el clásico
ffuf -u http://IP -H "Host: FUZZ.cohort.htb" \
  -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs 0 -mc all

# con dominios reales del certificado
ffuf -u http://IP -H "Host: FUZZ.target.htb" -w vhosts.txt -fs <tamaño_falso>

# gobuster
gobuster vhost -u http://IP -d target.htb -w wordlist --append-domain -t 50

# wfuzz
wfuzz -u http://IP -H "Host: FUZZ.target.htb" -w wordlist --hh 0

# curl a mano
curl -s -H "Host: admin.target.htb" http://IP/
```

Tips de fuzzing:

- Mira el **tamaño** de la respuesta (`-fs` para filtrar la falsa/por-defecto).
- Prueba `FUZZ` sin dominio (`Host: admin`), con y sin el dominio raíz, y con `FUZZ.target.htb`.
- **`default_server`** suele responder igual a todo: esa respuesta es la "falsa" y se filtra con `-fs`.
- Para HTTPS, fuerza el Host **y** el SNI:

```bash
curl -sk --resolve admin.target.htb:443:IP https://admin.target.htb/
curl -k -H "Host: admin.target.htb" https://IP/

# SNI fuzzing
sni-fuzz -t IP -w wordlist.txt
```

### Enumeración por DNS (si tienes resolución)

```bash
# subdominios de verdad (si el DNS del lab resuelve *.htb)
gobuster dns -d target.htb -w subdomains.txt
amass enum -passive -d target.htb
```

## Como acceder a un vhost descubierto

El nombre no resuelve en internet, hay que **mapearlo a la IP**:

```bash
echo "10.129.244.174 nb-1be3782a8afd3ad5.cohort.htb" | sudo tee -a /etc/hosts

# o solo para una petición (sin tocar /etc/hosts)
curl --resolve nb-xxx.cohort.htb:443:10.129.244.174 https://nb-xxx.cohort.htb/

# en Burp: Options -> Project options -> DNS -> Add host override
```

En Windows: `C:\Windows\System32\drivers\etc\hosts`.

Ojo con los **wildcards**: si `*.htb` resuelve a la IP, cualquier nombre "anda" y el fuzzing da falsos positivos.

## Vulnerabilidades comunes ligadas a vhosts

| Vulnerabilidad | Descripcion |
| :--- | :--- |
| **Vhost oculto / internal app** | Un servidor admin/api que solo se accede con el Host correcto, con menor seguridad |
| **Default server mal configurado** | El `default_server` sirve contenido de un sitio interno a cualquiera |
| **Host header injection** | La app usa el `Host` para generar enlaces (reset password, cache, redirecciones) => phishing / poisoning |
| **Cache poisoning** | Si el cache clavea mal el Host, sirves tu contenido a otros (o XSS persistente) |
| **Password reset poisoning** | `Host: attacker.com` hace que el link del email salga hacia tu servidor => robo de token |
| **Bypass de allowlist/auth** | La app solo protege `app.empresa.com`; entras con `Host: internal` o IP directa |
| **SSRF** | Si el backend redirige/llama usando el Host que le pasas |
| **TLS misconfig** | Cert de un sitio que expone otro (`SNI` equivocado), o `default` sin cert |
| **Apache `ServerAlias *`** | Cualquier Host cae en ese sitio (fuga) |

```bash
# Host header injection / reset poisoning
curl -H "Host: atacante.com" http://IP/password-reset?email=victim@x.com

# Bypass de auth por Host
curl -H "Host: localhost" http://IP/admin
curl -H "X-Forwarded-Host: internal" http://IP/admin
curl -H "X-Original-URL: /admin" http://IP/
```

## Flujo de trabajo (checklist)

1. `curl -sI IP` → detectar servidor web y cert/SAN.
2. Extraer **posibles nombres**: SAN del cert, HTML/JS, crt.sh, headers.
3. **Fuzzear Host** con wordlist de subdominios → filtrar por tamaño/status.
4. Probar HTTPS con `--resolve` y SNI.
5. Agregar hallazgos a `/etc/hosts` y re-hacer el **enum completo** contra ese vhost (dirbusting, API, login).
6. Repetir: cada vhost es una app nueva con su propio superficie.

## Ejemplo real: [[Cohort]] (HTB)

```
GET http://0/status            -> la respuesta trae el nombre de un vhost oculto
echo "IP nb-1be3782a8afd3ad5.cohort.htb" >> /etc/hosts
https://nb-1be3782a8afd3ad5.cohort.htb  -> aparece una instancia de MarIMO
```

Vhost descubierto -> app oculta ([[MARIMO]]) -> RCE -> flag. Flujo completo en [[Cohort]].

## Relacionados

- [[NGINX]] → donde se definen los `server_name` y como leerlos.
- [[MARIMO]] → la app que en [[Cohort]] estaba detrás del vhost.
