---
title: Server-Side Request Forgery
tags:
  - vulnerabilidad
  - teoria
aliases:
  - SSRF
puntuacion: " ~ 6.5 / 9.0"
---

---

# ¿Qué es?

Un SSRF es una falla en la que el atacante **fuerza al servidor a realizar peticiones de red hacia destinos que él elige** (internos, de loopback, de la nube o incluso con protocolos raros). El servidor actúa como un "proxy involuntario": lo que responda ese destino interno, que el atacante nunca podría tocar directamente, se lo devuelve o lo procesa por él.

---

## ¿Cómo funciona?
*La lógica conceptual detrás del fallo.*

1. **El escenario normal:** Una app necesita consumir una URL externa (importar una imagen, un webhook, un RSS, generar un PDF) y la pide al servidor: `GET /fetch?url=https://cdn.com/logo.png`.
2. **La manipulación:** El atacante cambia la URL por un destino interno: `url=http://127.0.0.1:8080/admin` o `url=http://169.254.169.254/latest/meta-data/`.
3. **La falla del sistema:** El servidor hace la petición **desde su propia posición de red** (donde sí tiene acceso a esas redes) y devuelve o usa la respuesta, **sin validar a qué está pidiendo**.

---

## Vectores Comunes

*¿Dónde o cómo suele presentarse este fallo en la vida real?*

- **Parámetros que aceptan una URL:** `?url=`, `?next=`, `?dest=`, `?redirect=`, `?image=`, `?webhook=`, `?callback=`, `?proxy=`, `?load_url=`. Cualquier "importar desde URL", generador de PDF, visor de imágenes o integración con APIs externas.
- **Cabeceras HTTP:** `X-Forwarded-Host`, `X-Original-URL`, `Forwarded`, `Referer` cuando la app construye URLs internas con ellas (combinable con [[NGINX]] mal configurado).
- **Subida de SVG/HTML/XML (XEE):** un SVG o XML con `<!ENTITY xxe SYSTEM "file:///etc/passwd">` hace que el servidor pida por ti.
- **Redirecciones abiertas:** la app valida que la URL inicial sea "legítima", pero ese sitio te redirige (302) a `http://127.0.0.1` y el servidor sigue la redirección.
- **Protocolos alternativos:** ademas de `http(s)://`, probar `file://`, `gopher://`, `dict://`, `ftp://`, `sftp://`, `ldap://`, `jar://`, `netdoc://` según la librería usada (Java, Python `requests`, PHP cURL).
- **Obfuscación de IPs/hostnames:** `127.0.0.1` en decimal (`2130706433`), octal (`0177.0.0.1`), hex (`0x7f.0.0.1`), IPv6 (`[::1]`, `[::ffff:127.0.0.1]`), localhost con DNS propio, `localtest.me`, o **DNS rebinding** (el DNS resuelve primero a un IP válido y después a `127.0.0.1` al revalidar).
- **SSRF a ciegas (blind/OOB):** no ves la respuesta, pero el servidor hace la petición => confirmas con un servidor colaborador (`webhook.site`, interactsh, Burp Collaborator) y por timing.
- **Endpoints "de infraestructura":** metadata de la nube `http://169.254.169.254/` (AWS/Azure/GCP), APIs internas, paneles de administración, `redis:6379`, `docker:2375`, `elasticsearch:9200`, `kubernetes:10250`, `[::1]:2375`.

```bash
# Detección rapida con curl
curl -s "http://IP/vuln.php?url=http://127.0.0.1"
curl -s "http://IP/vuln.php?url=http://169.254.169.254/latest/meta-data/"
curl -s "http://IP/vuln.php?url=file:///etc/passwd"

# Codificaciones alternativas de loopback
curl -s "http://IP/vuln.php?url=http://2130706433"
curl -s "http://IP/vuln.php?url=http://0177.0.0.1"
curl -s "http://IP/vuln.php?url=http://[::1]"

# Confirmacion OOB
curl -s "http://IP/vuln.php?url=https://TU-SUBDOMAIN.webhook.site"
```

---

## Impacto y Riesgos

*¿Qué puede lograr un atacante si explota esto exitosamente?*

- **Confidencialidad:**
  - **Metadata de la nube** (`169.254.169.254`): credenciales de la instancia AWS/GCP/Azure => account takeover de la cuenta en la nube.
  - Lectura de archivos locales con `file:///etc/passwd`, `file:///proc/self/environ`, `.env`, configs con credenciales.
  - Mapeo de la **red interna**: puertos, servicios, vhosts ocultos (`http://127.0.0.1/status`, `Host:` interno => ver [[VHOST]]) y apps que solo escuchan en `localhost`.
  - Robo de tokens de sesión o datos de APIs internas que consumió el servidor.
- **Integridad:**
  - Con `gopher://` se pueden **escribir** en servicios internos: mandar comandos a Redis/Memcached/SMTP/RabbitMQ => a partir de ahi suele venir un [[RCE]].
  - Cambiar configuraciones de APIs internas, disparar acciones administrativas (crear usuario, borrar datos) "en nombre" del servidor.
- **Disponibilidad:**
  - Escaneo/agotamiento de la red interna, DoS contra servicios internos, o amplificación hacia terceros.

> [!NOTE]
> El SSRF es el **puente clásico de pivoteo** en CTF y en la vida real: desde una web pública llegas a servicios que solo viven en `127.0.0.1` o en la VLAN de gestión. En [[Cohort]] el encadenamiento fue parecido: descubrir lo interno (`/status` -> vhost -> [[MARIMO]]) y atacarlo desde fuera.

---

## ¿Cómo se previene?
*Medidas defensivas y buenas prácticas de desarrollo.*

- **Lista blanca de destinos:** solo permitir dominios/IPs conocidos; nada de "cualquier URL que mande el usuario".
- **Bloquear rangos peligrosos:** `127.0.0.0/8`, `10/8`, `172.16/12`, `192.168/16`, `169.254.0.0/16`, `::1`, y IPv4-encoded/IPv6 alternativos. Validar **después** de resolver el DNS (anti-DNS rebinding).
- **Restringir esquemas:** solo `http` y `https`; deshabilitar `file`, `gopher`, `dict`, `ftp`.
- **No seguir redirecciones externas** (o re-validar el destino en cada salto).
- **Metadato de la nube endurecido:** IMDSv2 en AWS (`PUT http://169.254.169.254/latest/api/token` obligatorio con TTL corto).
- **Segmentación de red:** el servidor de la app no debería poder hablar con redes de gestión/base de datos.
- **No devolver la respuesta cruda** al usuario (si es posible, solo confirmar el resultado).
- **Timeouts y límites** de tamaño/conexión para mitigar el uso como proxy o DoS.

---
