
## 1. Máquinas & Writeups (Vista General)

### Todas las Máquinas
```dataview
TABLE
  platform AS "Plataforma",
  difficulty AS "Dificultad",
  os AS "SO",
  tags AS "Vulnerabilidades / Tags",
  choice(status = "Completada", "Completada", "⏳ " + default(status, "Pendiente")) AS "Estado",
  date AS "Fecha"
FROM "03_Writeups_Maquinas"
SORT date DESC
```

---

## 2. Filtros por Estado

### Máquinas Completadas
```dataview
TABLE
  platform AS "Plataforma",
  difficulty AS "Dificultad",
  os AS "SO",
  tags AS "Tags",
  date AS "Fecha"
FROM "03_Writeups_Maquinas"
WHERE status = "Completada" OR contains(lower(status), "complet")
SORT date DESC
```

### Máquinas En Progreso / Pendientes
```dataview
TABLE
  platform AS "Plataforma",
  difficulty AS "Dificultad",
  os AS "SO",
  tags AS "Tags"
FROM "03_Writeups_Maquinas"
WHERE status != "Completada" AND !contains(lower(default(status, "")), "complet")
SORT file.name ASC
```

---

## 3. Filtros por Plataforma

### Hack The Box
```dataview
TABLE
  difficulty AS "Dificultad",
  os AS "SO",
  tags AS "Tags",
  choice(status = "Completada", "✅ Completada", "⏳ " + default(status, "Pendiente")) AS "Estado"
FROM "03_Writeups_Maquinas"
WHERE contains(lower(default(platform, "")), "hack") OR contains(lower(file.name), "htb")
SORT difficulty ASC
```

### TryHackMe
```dataview
TABLE
  difficulty AS "Dificultad",
  os AS "SO",
  tags AS "Tags",
  choice(status = "Completada", "✅ Completada", "⏳ " + default(status, "Pendiente")) AS "Estado"
FROM "03_Writeups_Maquinas"
WHERE contains(lower(default(platform, "")), "try") OR contains(lower(file.name), "thm")
SORT difficulty ASC
```

---

## 4. Filtros por Sistema Operativo

### Máquinas Linux
```dataview
TABLE
  platform AS "Plataforma",
  difficulty AS "Dificultad",
  tags AS "Tags",
  choice(status = "Completada", "✅ Completada", "⏳ " + default(status, "Pendiente")) AS "Estado"
FROM "03_Writeups_Maquinas"
WHERE contains(lower(default(os, "")), "linux")
SORT platform ASC
```

### Máquinas Windows
```dataview
TABLE
  platform AS "Plataforma",
  difficulty AS "Dificultad",
  tags AS "Tags",
  choice(status = "Completada", "✅ Completada", "⏳ " + default(status, "Pendiente")) AS "Estado"
FROM "03_Writeups_Maquinas"
WHERE contains(lower(default(os, "")), "windows")
SORT platform ASC
```

---

## 5. Búsqueda por Etiquetas (Tags Globales)
```dataview
TABLE
  file.folder AS "Carpeta",
  file.tags AS "Tags"
FROM ""
WHERE length(file.tags) > 0 AND file.name != "Dashboard HTB"
SORT file.name ASC
```