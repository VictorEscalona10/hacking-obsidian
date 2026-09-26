SQLite es una base de datos **completa pero embebida en un solo archivo**: no hay servidor corriendo, todo (tablas, datos, índices) vive dentro de un archivo `.db`, `.sqlite`, `.sqlite3` o a veces sin extensión. Es la base de datos por defecto de muchísimas apps (PHP, Python, Android, Node.js).

Para obtener una shell para interactuar con un archivo sqlite utilizamos

```bash
sqlite3 nombre_del_archivo
```

## Comandos dentro de la shell

| Comando | Descripcion |
| :--- | :--- |
| `.tables` | Listar todas las tablas |
| `.schema [tabla]` | Ver la estructura (CREATE TABLE) de una tabla |
| `.headers on` | Mostrar los nombres de las columnas en los resultados |
| `.mode column` | Formatear la salida en columnas legibles |
| `.dump` | Volcar toda la base a SQL (texto legible) |
| `.dump tabla` | Volcar solo esa tabla |
| `.help` | Ayuda |
| `.quit` o `.exit` | Salir |

## Consultas SQL

```sql
.tables
.schema
.headers on
.mode column

SELECT name FROM sqlite_master WHERE type='table';
SELECT * FROM usuarios LIMIT 10;
.quit
```

- `sqlite_master` es la tabla interna donde SQLite guarda el esquema de todo.

## Consejo de examen (CTF / pentest)

Siempre activa headers y modo columnas **antes** de consultar, o la salida es un desastre:

```bash
sqlite3 archivo.db
sqlite> .headers on
sqlite> .mode column
sqlite> .tables
sqlite> SELECT * FROM users;
```

## SQLite fuera de la shell

```bash
# Consulta directa desde la terminal
sqlite3 archivo.db "SELECT * FROM users;"

# Volcar todo a un archivo .sql legible
sqlite3 archivo.db .dump > dump.sql

# Ver que contiene sin sqlite3 instalado
strings archivo.db | head -50
```

`strings` es útil cuando no tienes la herramienta: SQLite guarda el texto en claro dentro del archivo, así que contraseñas y datos suelen salir con solo extraer cadenas.

## Donde buscar archivos de base de datos

```bash
# Enumeracion web
gobuster dir -u http://IP -w wordlist -x db,sqlite,sqlite3,bak

# En la maquina
find / -name "*.db" -o -name "*.sqlite*" 2>/dev/null
```

Rutas típicas: `/var/www/html/database.db`, `app.db`, `users.db`, `/data/app.db`.

## Extras

- **Modo CSV**: `.mode csv` y `.output archivo.csv` para exportar datos.
- **Cifrado**: SQLite no viene cifrado por defecto. Si necesitas cifrar se usa *SQLCipher*.
- Un archivo `.db` es una **copia completa de la base**: si lo consigues, ya tienes todos los datos, sin necesidad de explotar nada más.
