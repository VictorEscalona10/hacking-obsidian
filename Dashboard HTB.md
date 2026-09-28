```dataview
TABLE difficulty AS "Dificultad", os AS "Sistema", tags AS "Vulnerabilidades", status AS "Estado"
FROM "03_Writeups_Maquinas"
WHERE status = "Completada"
SORT date DESC
```