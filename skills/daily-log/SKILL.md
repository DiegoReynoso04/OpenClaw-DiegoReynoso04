---
name: daily-log
description: "Registrar el avance diario de Diego en su diario de trabajo de Google Docs."
metadata: { "openclaw": { "emoji": "📓" } }
---
# Daily log

Convierte lo que Diego cuenta de su día en una entrada estructurada dentro de
su diario de trabajo en Drive. No es un resumen literal: es una entrada editada.

## Contexto que ya tienes
Carpeta, zona horaria, formato de fecha y tono están en TOOLS.md, USER.md y
IDENTITY.md. No los preguntes.

## Workflow
1. Si Diego no dio detalles, pregunta **una sola vez**: qué hizo, qué se atascó,
   qué queda pendiente. Una pregunta, no tres por separado.
2. Localiza el doc "Diario de trabajo" con `Find a Document` en la carpeta de trabajo.
   Si no existe, créalo con `Create Document From Text`.
3. Redacta la entrada con esta forma exacta:

```
## YYYY-MM-DD (día de la semana)

**Hecho**
- ...

**Atascos**
- <qué falló y qué lo resolvió, o qué sigue abierto>

**Siguiente**
- ...
```

4. `Append Text` al final del doc.
5. Confirma en el chat con el título y el enlace del doc.

## Reglas
- Editorializa: agrupa lo relacionado, descarta el ruido, ordena por importancia.
  Una lista plana de lo que dijo Diego no aporta nada.
- En "Atascos", registra la causa real, no el síntoma. Diego revisa esto para
  no repetir errores.
- Máximo 8 bullets por entrada. Si hay más, es que no has priorizado.
- Nunca escribir credenciales, tokens ni rutas a secretos en el diario.
- Una entrada por día: si ya existe la de hoy, añade a esa sección en vez de
  crear otra.
- Antes de crear el doc, comprueba con `Find a Document` si ya existe. Si aparece
  más de uno con el mismo nombre, usa el más antiguo y avisa a Diego del duplicado.
  Nunca crear un doc nuevo si ya hay uno.