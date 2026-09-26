MD5 (Message Digest Algorithm 5) es una **función de hash** criptográfica que toma cualquier texto o archivo y produce una huella única de **128 bits (32 caracteres hexadecimales)**.

Ojo: **no es encriptación**. La encriptación es reversible (se puede descifrar), el hash **no**. A partir del hash no se "descifra" nada, se intenta adivinar la contraseña original probando candidatos.

```
"password"  ->  5f4dcc3b5aa765d61d8327deb882cf99
```

Características:

- Siempre sale con el mismo largo (32 caracteres), sin importar el largo del texto.
- Cualquier cambio mínimo produce un hash totalmente distinto.
- Es **muy rápido** de calcular, lo que lo hace débil para contraseñas.

## ¿Dónde lo vas a ver

- Contraseñas guardadas en bases de datos (foros viejos, CMS, CTFs).
- Archivos `passwd`/`shadow` con usuarios `user:hash`.
- Verificación de integridad de archivos (`md5sum` contra el hash publicado por el sitio).

## Identificar un hash

```bash
hash-identifier
```

Pega el hash y te dice si es MD5, SHA1, SHA256, etc. También sirve **Hash Identifier** en **CyberChef**.

También se puede identificar a ojo: MD5 = 32 caracteres hex, SHA1 = 40, SHA256 = 64.

## Calcular MD5

```bash
md5sum archivo.txt
echo -n "password" | md5sum
```

- `-n` es importante: sin él, `echo` agrega un salto de línea y el hash no coincide.

## Crackear MD5

**JTR (John the Ripper):**

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show --format=raw-md5 hash.txt
```

**Hashcat:**

```bash
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

- `-m 0` = modo MD5 raw.
- Si el hash viene con sal (`user:hash` o `hash:salt`), revisa los modos `10`/`20`.

## Ataques comunes

| Ataque | Que hace |
| :--- | :--- |
| **Diccionario** | Prueba palabras de una lista (rockyou, etc.) |
| **Rainbow tables** | Tablas precalculadas de hash -> contraseña, evitan calcular |
| **Fuerza bruta** | Todas las combinaciones posibles |
| **Reversa online** | Sitios como crackstation o md5decrypt que tienen bases de datos de hashes ya resueltos |
| **Colisión** | Encontrar 2 textos distintos con el mismo hash (roto en MD5 desde 2004) |

Truco rápido: antes de romper nada, **pégalo en Google o en un reversor online**. MD5 es tan viejo que la mayoría de hashes de contraseñas comunes ya están resueltos.

## Por que NO usar MD5 para contraseñas

- Es rapidísimo: se pueden probar **miles de millones de hashes por segundo** con GPU.
- No tiene sal incorporada, así que dos usuarios con la misma contraseña tienen el mismo hash.
- Está roto: existen colisiones prácticas.

Alternativas correctas actuales: **bcrypt**, **Argon2**, **scrypt**, **PBKDF2**.

## MD5 vs SHA

| Algoritmo | Largo | Estado |
| :--- | :--- | :--- |
| MD5 | 128 bits (32 hex) | Roto, solo para checksums |
| SHA1 | 160 bits (40 hex) | Roto para colisiones |
| SHA256 | 256 bits (64 hex) | Seguro |

Un checksum MD5 para verificar que un ISO descargó bien sigue siendo válido (no es un escenario adversario), **para contraseñas nunca**.
