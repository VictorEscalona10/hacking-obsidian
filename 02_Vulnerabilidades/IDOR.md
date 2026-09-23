---
title: Insecure Direct Object Reference
tags:
  - vulerabilidad
  - teoria
aliases:
  - IDOR
puntuacion: " ~ 6.0 / 9.0"
---

---

# ¿Qué es?

Un IDOR es una falla de control de acceso que ocurre cuando un sistema muestra datos privados al cambiar un simple parámetro, sin verificar los permisos del usuario

---

## ¿Cómo funciona?
*La lógica conceptual detrás del fallo.*

1. **El escenario normal:** Un usuario entra a su perfil en `?id=50`
2. **La manipulación:** El atacante cambia el parámetro a `?id=51`
3. **La falla del sistema:** El servidor procesa la petición basándose solo en la autenticación, ignorando la autorización, y devuelve los datos

---

## Vectores Comunes

*¿Dónde o cómo suele presentarse este fallo en la vida real?*
- **Vector 1:** Parámetros numéricos predecibles en la URL
- **Vector 2:** APIs que confían ciegamente en el input del lado del cliente
- **Vector 3:** Subida de archivos sin validación de extensión

---

## Impacto y Riesgos

*¿Qué puede lograr un atacante si explota esto exitosamente?*
- **Confidencialidad:** Robo de historias médicas, facturas o datos de clientes
- **Integridad:** Modificación o eliminación de registros ajenos
- **Disponibilidad:** En caso de un [[RCE]], caída total del servicio

---

## ¿Cómo se previene?
*Medidas defensivas y buenas prácticas de desarrollo.*
- **Validación del lado del servidor:** Comprobar siempre que el usuario en sesión es el dueño legítimo del recurso
- **Diseño seguro:** Usar identificadores únicos universales (UUID) que sean imposibles de adivinar en lugar de números secuenciales

