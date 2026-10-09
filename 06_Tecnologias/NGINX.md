NGINX (pronunciado "engine-x") es un servidor web de código abierto, uno de los más usados del mundo. Además de servir páginas estáticas, funciona como **reverse proxy**, **load balancer** y **proxy de correo**. Es el "cara frontal" de casi todo: detrás de él casi siempre hay una app (Node, Python, PHP, Java) o algo interno que no debería exponerse directamente.

Su virtud (y su peligro) es que **toda la lógica de ruteo** (que URL va a qué backend, que dominios atiende, que archivos sirve) vive en texto plano en su configuración.

## Como funciona por dentro

- Es **event-driven / síncrono no bloqueante**: poca memoria por conexión, ideal para servir muchos clientes simultáneos (a diferencia de Apache, que gasta un proceso/hilo por conexión).
- Cada petición pasa por un pipeline: `HTTP modules -> filas -> filtro de contenido -> respuesta`.
- Lo interesante para nosotros: **`proxy_pass`**: nginx recibe la petición, la transforma (headers, rewrite, auth) y la reenvía a un backend interno.

## Estructura de configuración

```
/etc/nginx/nginx.conf              -> config principal
/etc/nginx/conf.d/*.conf           -> configs incluidas (sites nuevos)
/etc/nginx/sites-enabled/*.conf    -> vhosts habilitados (symlinks de sites-available)
/etc/nginx/sites-available/        -> todos los vhosts (aunque no estén activos)
/var/log/nginx/access.log          -> peticiones
/var/log/nginx/error.log           -> errores (suele filtrar rutas internas)
/etc/nginx/mime.types              -> asociación extensión -> tipo
```

Distro-dependent: Debian/Ubuntu usan `sites-enabled`, RHEL/CentOS/Fedora usan `/etc/nginx/conf.d/` y `/usr/share/nginx/html` como root por defecto.

Ejemplo de un `server block` mínimo:

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name www.ejemplo.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:3000/;   # backend interno
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Directivas clave:

| Directiva | Que hace |
| :--- | :--- |
| `server_name` | Que dominios/Host header atiende este bloque (=> vhost, ver [[VHOST]]) |
| `listen` | Puerto y si es `ssl`, `default_server` |
| `root` | Directorio base de los archivos estáticos |
| `location` | Matcheo por prefijo/regex de la URI |
| `proxy_pass` | Reenvía la petición a otro servicio (el corazón del reverse proxy) |
| `try_files` | Busca el archivo o devuelve otro (SPA suelen usar `try_files $uri /index.html`) |
| `return` / `rewrite` | Redirecciones y saltos |
| `autoindex` | Si está `on`, genera **directory listing** de ese path |
| `include` | Trae otros archivos de config (aqui es donde se esconden los vhosts) |
| `add_header` | Headers extra en la respuesta (a veces filtran versiones/Info) |

## Como identificar que hay NGINX

```bash
curl -sI http://IP/
# Server: nginx/1.18.0 (Ubuntu)   -> version + SO
# Server: nginx                   -> version oculta

nmap -sV -p80,443 --script http-headers IP

# 404 por defecto de nginx (sospechoso de config "virgen"):
curl -s http://IP/no_existe | head
```

La página 404 por defecto de nginx (`<center>nginx</center>`) indica que **no hay app detrás de ese vhost** o que nadie configuró `error_page`.

## Endpoints y rutas que siempre hay que probar

| Ruta | Por que |
| :--- | :--- |
| `/status`, `/stub_status`, `/nginx_status` | Módulo `stub_status`: conexiones activas, aceptadas, manejadas => **info leak** (y en CTF a veces esconde algo más) |
| `/server-status`, `/server-info` | Esto es de **Apache** (mod_status/mod_info), no de nginx; si responde, hay Apache |
| `/` con directory listing | `autoindex on` mal puesto |
| `/.git/`, `/.env`, `nginx.conf.bak` | Config/archivos filtrados |
| `/` con Host header distinto | Otro vhost oculto, ver [[VHOST]] |

```bash
# Probar el status de nginx
curl -s http://IP/status
curl -s http://IP/nginx_status
curl -sI http://IP/stub_status

# Directory listing
ffuf -u http://IP/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```

En [[Cohort]] (HTB) fue justamente un `GET /status` lo que terminó devolviendo el nombre de un vhost oculto: ver [[Cohort]].

## Acceso a la configuración (lo más valioso)

Si logras leer la config de nginx tienes **el mapa completo**: dominios internos, backends, credenciales de base de datos, rutas protegidas.

```bash
# Desde una shell en la maquina
cat /etc/nginx/nginx.conf
ls -R /etc/nginx/
grep -rn "proxy_pass\|server_name\|root\|auth_basic" /etc/nginx/
cat /var/log/nginx/access.log | tail -50

# Externos típicos: backups y archivos mal servidos
curl -s http://IP/nginx.conf
curl -s http://IP/../etc/nginx/nginx.conf      # traversal
ffuf -u http://IP/FUZZ -w wordlist -x conf,bak,old,txt,zip -t 50

# Git filtrado (vuelca el repo entero)
githacker --url http://IP/.git/ --output-folder out/   # o git-dumper
```

Truco: si tienes `proxy_pass` a `127.0.0.1:XXXX` y tienes SSRF, puedes saltar al backend interno que nginx estaba protegiendo.

## Vulnerabilidades y malas configuraciones

### 1. `proxy_pass` mal construido (bypass / SSRF / acceso a lo interno)

```nginx
# el / final importa MUCHO
location /api { proxy_pass http://backend; }        # pasa la URI tal cual
location /api { proxy_pass http://backend/; }       # recorta el prefijo /api
```

- Si el backend se expone en otro puerto/host y nginx solo hace de "cortina", un path mal matcheado puede dejar **pasar directo al backend**.
- Un `proxy_pass` con variable (`proxy_pass $backend;`) habilita **SSRF** si controlas la variable (Host header, parámetro, etc.).
- Acceso a "solo-localhost": apps que solo confían en `127.0.0.1`. Si tienes un endpoint `proxy_pass` a ellas, entras por la puerta.

### 2. Alias traversal / "off-by-slash"

```nginx
location /img { alias /var/www/img/; }
```

`/img../etc/passwd` (o `/img../`) queda fuera del directorio esperado. Clásico CVE-2009-3899 y sigue apareciendo en configs viejas hechas a mano.

```bash
ffuf -u "http://IP/img../FUZZ" -w wordlist
```

### 3. Directory listing y archivos servidos

```nginx
location /backup { autoindex on; alias /var/backups/; }
```

- `autoindex on` en un path = inventario completo.
- nginx sirve **cualquier extensión** que coincida con `root`: `web.config.bak`, `.git`, `id_rsa`, `.sql`, `dump.zip`.
- Si `root` apunta a `/` o a `/etc` por error: lectura de ficheros del sistema directa.

```bash
ffuf -u "http://IP/FUZZ" -w wordlist -e .bak,.old,~, .swp, .sql, .zip
```

### 4. `stub_status` / headers que filtran

`stub_status` solo da números (conexiones), pero en CTF a veces está combinado con un `return 200` que devuelve info. También: `Server:` con versión permite buscar CVEs.

### 5. Manipulación de IP real (X-Forwarded-For)

Cuando nginx hace `proxy_set_header X-Real-IP $remote_addr;` o la app confía en `X-Forwarded-For`, puedes **falsificar tu IP** para pasar filtros tipo "solo admin desde localhost" o IP blanqueada.

```bash
curl -H "X-Forwarded-For: 127.0.0.1" -H "X-Real-IP: 127.0.0.1" http://IP/admin
curl -H "X-Forwarded-For: 10.10.14.5" http://IP/
```

Nota: solo funciona si la app/nginx no usan `set_real_ip_from` + `real_ip_header` bien configurado (o si lo confías a ciegas).

### 6. HTTP Request Smuggling (nginx como frontend)

Con `proxy_http_version 1.1` + keepalive hacia atrás, mal manejo de `Content-Length`/`Transfer-Encoding` permite **CL.TE / TE.CL smuggling**: "colar" una petición dentro de otra y secuestrar conexiones de otros usuarios (hitting de credenciales, saltar auth del proxy).

```bash
smuggleTest.py -u https://IP
python3 smuggler.py -u http://IP
```

### 7. CVEs históricas de nginx (según versión)

| CVE | Versión | Impacto |
| :--- | :--- | :--- |
| CVE-2017-7529 | < 1.13.3 | Integer overflow en `Range` header => info leak / DoS |
| CVE-2013-4547 | 1.5.6-1.5.7 | Bypass de seguridad con espacio en la URI (`/file\ \ /..`) |
| CVE-2021-23017 | < 1.20.0 | Resolver DNS: write de 1 byte => posible RCE (UDP spoofing) |
| CVE-2014-3616 | < 1.7.5 | SSI con `merge_slashes off` |
| CVE-2009-3899 | < 0.8.14 | Alias traversal |
| CVE-2022-41741/41742 | < 1.22.1 | Memoria corrupta en mp4 module |

```bash
# Sacar version exacta
nmap -sV -p80,443 IP
whatweb http://IP
```

### 8. Cache poisoning / host header

Si hay un `proxy_cache` o CDN delante, un `Host` malo o un `X-Forwarded-Host` cacheado puede **envenenar** la cache y servir contenido tuyo a todos (incluso XSS almacenado).

## Flujo de trabajo en una CTF / pentest con nginx

1. **Detectar**: `curl -sI` → `Server: nginx`.
2. **Version**: si sale versión → buscar CVE / malformaciones conocidas.
3. **Endpoints**: `/status`, `/nginx_status`, directory listing, backups.
4. **Vhosts**: fuzz de `Host` header contra el mismo IP → ver [[VHOST]] y [[Cohort]].
5. **Paths internos**: probar los `location` típicos que nginx protege (`/admin`, `/internal`, `/api`, `/backend`, `/actuator`, `/.git`).
6. **Headers falsificados**: `X-Forwarded-For: 127.0.0.1`, `X-Original-URL`, `X-Rewrite-URL`.
7. **Si obtienes shell**: leer `/etc/nginx/nginx.conf` + `sites-enabled/*` para mapear **todos** los servicios y backends internos → base para la escalada.

```bash
# Rápido y completo
curl -sI http://IP/
curl -s http://IP/status
ffuf -u http://IP/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -mc all -fs 0
ffuf -u http://IP -H "Host: FUZZ.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 0
```

## Cuando tienes shell: nginx como aliado

```bash
grep -rn "proxy_pass\|server_name" /etc/nginx/ | grep -v "#"
ls /etc/nginx/sites-enabled/
# Cada server_name = un dominio/vhost interno nuevo
# Cada proxy_pass = un servicio interno que ahora puedes tocar por 127.0.0.1
```

Los vhosts suelen tener nombres tipo `interno.cohort.htb`, `backend.local`, `admin.empresa.htb`: pruébalos en `/etc/hosts` y atácalos directamente, saltándote la capa de nginx.

