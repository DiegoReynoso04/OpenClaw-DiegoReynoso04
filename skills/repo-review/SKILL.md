---
name: repo-review
description: "Revisar un repositorio de GitHub de Diego y reportar problemas concretos con su arreglo."
metadata: { "openclaw": { "emoji": "🔍" } }
---
# Repo review

Revisa un repo de Diego y devuelve hallazgos accionables, no una descripción
de lo que ya se ve al abrirlo.

## Input
Una URL de GitHub o `owner/repo`. Nada más. No preguntes por contexto adicional:
si algo no está claro, léelo del repo.

## Workflow
1. `Find Repository` — descripción, visibilidad, rama por defecto.
2. `Get File Contents` del README y de los archivos de configuración relevantes.
3. `Find Branch` para ver el estado de las ramas.
4. Reporta en el chat con esta forma:

```
## <repo>  ·  público|privado

**Bien**
- <máx. 3, solo lo no obvio>

**Problemas**
- <hallazgo> → <qué hacer>

**Riesgos**
- <secretos expuestos, config sensible, permisos>
```

## Reglas
- Cada problema lleva su arreglo concreto. Un hallazgo sin acción es ruido.
- Si el repo es público, revisa siempre si hay datos que no deberían estar ahí:
  emails, hostnames, IDs de recursos, rutas internas, endpoints.
- Ordena los hallazgos por impacto real, no por el orden en que los encontraste.
- Verifica antes de afirmar: si no leíste el archivo, no opines sobre él.
  Distingue lo comprobado de lo que supones.
- Si la sección "Riesgos" queda vacía, dilo explícitamente. Un vacío sin
  explicar parece que no miraste.
- Esta skill solo lee. No modifica nada, no crea ramas, no abre issues ni PRs.
  Si un arreglo merece un PR, lo propones y esperas a que Diego lo pida.