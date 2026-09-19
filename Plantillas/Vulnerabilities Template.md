---
title:
tags:
aliases:
puntuacion:
---

---

# ¿Qué es?

(Escribe aquí una definición directa de 2 a 3 líneas. Ejemplo: Un IDOR es una falla de control de acceso que ocurre cuando un sistema muestra datos privados al cambiar un simple parámetro, sin verificar los permisos del usuario).

---

## ¿Cómo funciona?
*La lógica conceptual detrás del fallo.*

1. **El escenario normal:** (Ej: Un usuario entra a su perfil en `?id=50`).
2. **La manipulación:** (Ej: El atacante cambia el parámetro a `?id=51`).
3. **La falla del sistema:** (Ej: El servidor procesa la petición basándose solo en la autenticación, ignorando la autorización, y devuelve los datos).

---

## Vectores Comunes

*¿Dónde o cómo suele presentarse este fallo en la vida real?*
- **Vector 1:** (Ej: Parámetros numéricos predecibles en la URL).
- **Vector 2:** (Ej: APIs que confían ciegamente en el input del lado del cliente).
- **Vector 3:** (Ej: Subida de archivos sin validación de extensión).

---

## Impacto y Riesgos

*¿Qué puede lograr un atacante si explota esto exitosamente?*
- **Confidencialidad:** (Ej: Robo de historias médicas, facturas o datos de clientes).
- **Integridad:** (Ej: Modificación o eliminación de registros ajenos).
- **Disponibilidad:** (Ej: En caso de un RCE, caída total del servicio).

---

## ¿Cómo se previene?
*Medidas defensivas y buenas prácticas de desarrollo.*
- **Validación del lado del servidor:** (Ej: Comprobar siempre que el usuario en sesión es el dueño legítimo del recurso).
- **Diseño seguro:** (Ej: Usar identificadores únicos universales (UUID) que sean imposibles de adivinar en lugar de números secuenciales).

---

## 🔗 Referencias de Estudio
- [Enlace a un artículo o documentación oficial, ej: OWASP]()