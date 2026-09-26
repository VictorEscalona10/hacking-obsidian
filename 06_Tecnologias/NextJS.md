Next.js es un **framework de React** desarrollado por Vercel. Permite renderizar las páginas tanto en el **cliente** (como React normal) como en el **servidor** (SSR / SSG), lo que lo hace muy usado para aplicaciones web modernas.

## Como saber si el sitio usa Next.js

En la consola del navegador colocamos esto para conocer su versión:

```bash
window.next.version
```

Otras formas de detectarlo:

- En el **HTML fuente** (`Ctrl+U`) aparecen rutas `/ _next/static/...` (sin el espacio).
- Archivos como `/_next/static/chunks/pages/...js`.
- Cabecera `x-nextjs-cache` o `x-powered-by: Next.js` en la respuesta HTTP.

```bash
curl -sI http://IP | grep -i next
```

## Estructura típica

```
/app            -> rutas (App Router, Next 13+)
/pages          -> rutas (Pages Router, legacy)
/next.config.js -> configuración
/.next          -> build generado (si se filtra, hay info útil)
```

## Reconocimiento

```bash
# Ver que páginas/rutas existen en el bundle
curl -s http://IP/ | grep -o '/_next/static/[^"]*'

# Directorio build expuesto (mala configuración)
curl -sI http://IP/.next/

# Source maps expuestos -> código fuente completo del servidor
curl -s http://IP/_next/static/chunks/pages/index.js.map
```

Si encuentras un `.map` (source maps), puedes recuperar el **código fuente original**, incluyendo variables de entorno, tokens, rutas internas y lógica del servidor. Es uno de los hallazgos más comunes en máquinas CTF.

## Rutas interesantes

| Ruta | Descripcion |
| :--- | :--- |
| `/_next/static/...` | JS, CSS y chunks del bundle |
| `/_next/data/{buildId}/...` | Datos JSON que consume el SSR (buildId en el HTML) |
| `/api/*` | Endpoints del backend (App Router / pages/api) |
| `/.next/` | Build completo si quedó expuesto |
| `/robots.txt`, `/sitemap.xml` | Rutas y endpoints ocultos |

## Vulnerabilidades comunes

- **Source maps / bundle expuestos** -> fuga de código fuente y secretos.
- **SSRF en el servidor** (las funciones server-side hacen fetchs a URLs controladas por el usuario).
- **CRLF / header injection** en redirects de `next.config.js`.
- **Exposición de `NEXT_PUBLIC_*`**: todo lo que empieza con `NEXT_PUBLIC_` viaja al navegador, no es secreto.
- **Errores de depuración** en producción (`/_next/...` con stack traces).

```bash
# Buscar endpoints y palabras clave en los bundles
gobuster dir -u http://IP -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt
```

## Versiones

| Version | Router |
| :--- | :--- |
| Next 9 - 12 | `pages/` (Pages Router) |
| Next 13+ | `app/` (App Router), se puede mezclar con `pages/` |

Saber la versión ayuda a decidir que rutas probar y que CVEs/busquedas aplicar.
