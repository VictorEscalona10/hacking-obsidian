script /dev/null -c /bin/bash

El comando `script /dev/null -c /bin/bash` es una técnica muy conocida en sistemas Linux, especialmente en el ámbito de la ciberseguridad y el _pentesting_ (pruebas de penetración). Se utiliza principalmente para **"actualizar" una shell básica (dumb shell) a una terminal completamente interactiva (TTY)**.

Aquí tienes el desglose exacto de lo que hace cada parte del comando:

- **`script`**: Es un programa de Linux diseñado originalmente para grabar todo lo que ocurre en una sesión de terminal y guardarlo en un archivo de texto. Al ejecutarse, `script` crea un pseudo-terminal (PTY) de forma nativa.
    
- **`/dev/null`**: Es el archivo de destino donde `script` intentará guardar la grabación. `/dev/null` es conocido como el "agujero negro" de Linux; todo lo que se envía allí se descarta inmediatamente. Esto se hace para no dejar un archivo de registro (log) en el sistema.
    
- **`-c /bin/bash`**: La bandera `-c` le indica a `script` que ejecute un comando específico en lugar de abrir la shell por defecto. En este caso, ejecuta `/bin/bash`, que es un intérprete de comandos más avanzado y amigable.
    

### ¿Por qué se utiliza?

Cuando un auditor de seguridad o administrador obtiene acceso remoto a una máquina (por ejemplo, a través de una [[Reverse Shell]] usando `netcat`), la terminal que recibe suele ser muy limitada. Estas "dumb shells" tienen varios problemas:

1. **No hay control de trabajos:** Si presionas `Ctrl + C` para detener un proceso (como un `ping`), se cierra toda la conexión y pierdes el acceso.
    
2. **Sin historial ni navegación:** Las flechas del teclado no funcionan (imprimen caracteres raros como `^[[A` en lugar de subir al comando anterior).
    
3. **Comandos interactivos rotos:** Programas que requieren una terminal real, como `su`, `sudo`, `nano`, `vim` o incluso `clear`, no funcionan o muestran errores.
    

Al ejecutar `script /dev/null -c /bin/bash`, el programa `script` envuelve la nueva sesión de `bash` dentro del pseudo-terminal (PTY) que acaba de crear. Esto engaña al sistema haciéndole creer que hay un usuario real sentado frente a una terminal física, devolviéndote el formato correcto, el uso de comandos interactivos y una experiencia de uso normal.