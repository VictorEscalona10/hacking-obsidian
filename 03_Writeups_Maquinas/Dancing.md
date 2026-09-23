---
title: Hack the box- Dancing
date: 2026-09-22
platform: Hack the box
difficulty: Very Easy
os: Windows
tags:
  - SMB
  - Windows
status: Completada
---

# Resumen Ejecutivo
**IP Objetivo:** `10.129.227.216`
**Descripción breve:** Una maquina Windows 10

---

## 1. Information Gathering (Reconocimiento)

### Escaneo de Puertos
```bash

```

**Puertos Abiertos:**
- 135, 139, 445, 5985, 47001, 49664, 49665, 49666, 49667, 49668, 49669

### Enumeración de Servicios / Fuzzing
```bash

```

**Descubrimientos clave:**
- 

---

## 2. Vulnerability Assessment (Análisis de Vulnerabilidades)
- El protocolo [[SMB]] esta corriendo en esta maquina, quiere decir que tal vez podriamos meternos
- revisar si podria ser un recurso compartido mal configurado, Eso se llama **Null Session** o sesión nula.

---

## 3. Exploitation (Acceso Inicial)

### Metodología
1. queremos revisar si hay una null sesion en [[SMB]]

### Ejecución
```bash
smbclient -L //10.129.227.216 -N
```

Eso devuelve 4 shares, probamos con WorkShare

```bash
smbclient //10.129.227.216/WorkShares -N
```


**Prueba de acceso (User Flag):**
```bash
5f61c10dffbc77a704d76016a22f1664
```

---
