# Reverse Shell

Una **reverse shell** (o *intérprete de comandos inverso*) es una sesión interactiva de terminal en la que **el flujo de conexión de red se invierte**: la máquina objetivo (la víctima) inicia una conexión saliente hacia una máquina receptora (el atacante u operador) y redirige los flujos estándar de su shell (`stdin`, `stdout`, `stderr`) a través de ese socket de red.

---

## ¿Cómo funciona a nivel de sockets y procesos?

En un sistema operativo (como Linux o Windows), una terminal normal interactúa con tres descriptores de archivo básicos:

- `stdin` (0): Entrada de datos (lo que escribes en el teclado).
- `stdout` (1): Salida estándar (lo que muestra la pantalla).
- `stderr` (2): Salida de errores.

### Flujo en una Reverse Shell

1. **Creación del socket:** El proceso en la máquina víctima abre un socket de red TCP hacia la dirección IP y puerto del oyente.
2. **Duplicación de descriptores (`dup2`):** El código redirige `stdin`, `stdout` y `stderr` directamente hacia el socket de red TCP.
3. **Invocación del binario:** Se ejecuta `/bin/sh`, `/bin/bash` o `cmd.exe`.
4. **Interacción remota:** Cada carácter que envías desde tu consola viaja por la red y alimenta el `stdin` del bash remoto; toda respuesta del sistema viaja de vuelta por el socket y se imprime en tu pantalla.

---

## ¿Por qué existe y por qué se utiliza?

La reverse shell existe principalmente para **evadir las restricciones de red perimetrales (firewalls y NAT)**:

- **El problema de la bind shell:** Si intentas conectarte directamente a la máquina objetivo, el firewall del servidor casi siempre bloqueará cualquier conexión entrante a un puerto arbitrario. Además, si la víctima está detrás de un router o NAT, ni siquiera tiene una IP pública alcanzable directamente.
- **La ventaja de la reverse shell:** La mayoría de los cortafuegos y políticas corporativas son estrictos con el tráfico que **entra**, pero permisivos con el tráfico que **sale** (para permitir que los servidores descarguen actualizaciones o naveguen). Como la conexión sale de adentro hacia afuera, el firewall generalmente la autoriza sin sospechas.