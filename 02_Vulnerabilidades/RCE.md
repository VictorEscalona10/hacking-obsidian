---
title: Remote Code Execution
tags:
  - vulnerabilidad
  - teoria
aliases:
  - RCE
puntuacion: "9.0 - 10.0 (Crítica)"
---

---

# ¿Qué es?

Consiste en un fallo en el diseño o implementación de una aplicación o servicio que permite a un usuario remoto (sin acceso físico a la máquina y, a menudo, sin credenciales válidas) **forzar al servidor o dispositivo a ejecutar código arbitrario o comandos del sistema operativo**.

---

## ¿Cómo funciona?
*La lógica conceptual detrás del fallo.*

1. **El escenario normal:** Una aplicación recibe datos del exterior (a través de un formulario web, una API, una cabecera HTTP, etc.) para procesar una solicitud de usuario.
2. **La manipulación:** El atacante envía entradas especialmente manipuladas que contienen código arbitrario o comandos del sistema.
3. **La falla del sistema:** La aplicación pasa los datos directamente al motor de ejecución del sistema o a un intérprete de código **sin sanitizarlos ni validarlos adecuadamente**, forzando su ejecución.

---

## Vectores Comunes

*¿Dónde o cómo suele presentarse este fallo en la vida real?*
- **Inyección de comandos (*Command Injection*):** Concatenar entradas del usuario directamente en llamadas al sistema (`system()`, `exec()`, `shell_exec()`).
- **Subida de archivos sin validar (*Unrestricted File Upload*):** Subir un archivo con código ejecutable (como `.php`, `.jsp`, `.aspx`) al servidor web y solicitar su URL en el navegador para que el servidor lo interprete.
- **Deserialización insegura:** Procesar objetos serializados manipulados por el atacante (muy común en Java, Python o PHP), lo que altera el flujo del programa al reconstruir el objeto y fuerza la ejecución de código.
- **Desbordamiento de búfer (*Buffer Overflow*):** En programas escritos en lenguajes de bajo nivel como C o C++, escribir más datos de los que caben en la memoria para sobreescribir punteros de instrucción (como el registro `RIP`/`EIP`) y redirigir la ejecución hacia un *shellcode*.
- **Fallos en bibliotecas o servicios desactualizados:** Vulnerabilidades públicas conocidas (como la famosa falla *Log4Shell* en Log4j o vulnerabilidades en versiones viejas de servidores como Apache o SMB).

---

## Impacto y Riesgos

*¿Qué puede lograr un atacante si explota esto exitosamente?*
En la escala CVSS, los RCE casi siempre obtienen una puntuación de **9.0 a 10.0 (Crítica)** porque rompen por completo los tres pilares de la seguridad:
- **Confidencialidad:** El atacante puede leer cualquier archivo accesible para el usuario del servicio (códigos fuente, credenciales de bases de datos, variables de entorno, claves SSH).
- **Integridad:** Puede modificar o borrar bases de datos, alterar la lógica de la aplicación o plantar *backdoors*.
- **Disponibilidad:** Puede tumbar servicios, cifrar la máquina con ransomware o consumir todos los recursos de hardware.

> [!NOTE]
> Una vez que consigues RCE, el siguiente paso natural en una auditoría o ataque suele ser pivotar hacia la escalada de privilegios o **desplegar una reverse shell** para interactuar cómodamente con el sistema operativo en tiempo real.

---

## ¿Cómo se previene?
*Medidas defensivas y buenas prácticas de desarrollo.*
- **Validación y sanitización estricta:** Validar y escapar todas las entradas del usuario con listas blancas (*allowlists*) antes de procesarlas.
- **Evitar llamadas directas al intérprete/shell:** Usar funciones seguras y parametrizadas en vez de APIs de ejecución de comandos directos del sistema.
- **Control estricto de subida de archivos:** Validar extensiones, tipos MIME y contenido; almacenar archivos fuera de la raíz web y sin permisos de ejecución.
- **Principio de mínimo privilegio:** Ejecutar las aplicaciones y servicios con cuentas de usuario que tengan los menores privilegios posibles para mitigar el impacto.
- **Gestión de dependencias y parches:** Mantener bibliotecas, servidores y sistemas actualizados frente a vulnerabilidades conocidas.

---

## Referencias de Estudio
- [OWASP - Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [MITRE ATT&CK - Command and Scripting Interpreter (T1059)](https://attack.mitre.org/techniques/T1059/)