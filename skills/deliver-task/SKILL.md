---
name: "deliver-task"
description: "Entrega una tarea en 4Geeks: busca por nombre, muestra resumen, pide confirmación y ejecuta PUT + deliver"
---

# Deliver Task

Entrega una tarea de 4Geeks Academy: busca el task_id por nombre, confirma con el usuario, y ejecuta la entrega (PUT URLs + POST deliver).

## When to use

- "Entrega X" / "Entregar proyecto X" / "Entrega el proyecto"
- "Ya está listo X, entrégalo" / "Sube X a 4geeks"
- "Deliver X"
- Cualquier instrucción que implique entregar una tarea/proyecto en 4Geeks

## API Details

- **Base URL:** `https://breathecode.herokuapp.com`
- **Auth header:** `Authorization: Token <TOKEN_4GEEKS>`
- **Token source:** `.env` file in workspace root, variable `TOKEN_4GEEKS`

### Endpoints

1. `GET /v1/assignment/user/me/task?limit=200` — all tasks (filtra por task_status, task_type)
2. `PUT /v1/assignment/task/{task_id}` — update task URLs (body: `github_url`, `live_url`)
3. `POST /v1/assignment/task/{task_id}/deliver` — mark task as delivered for review

## Workflow

### Fase 1: Buscar la tarea

1. Read `TOKEN_4GEEKS` from `.env`.
2. Call `GET /v1/assignment/user/me/task?limit=200` to get all tasks.
3. Deduplicate by `associated_slug`. Priority: `APPROVED (4) > REJECTED (3) > DONE (2) > PENDING (0)`.
4. Search for tasks matching the user's query against `title` and `associated_slug` (case-insensitive substring match).
5. **Match results:**
   - **0 matches:** Tell the user no task was found matching that name. Suggest they check the exact name or list pending tasks.
   - **1 match:** Proceed to phase 2.
   - **2+ matches:** Show all matches with `task_id`, `title`, `task_type`, `task_status`, and `cohort name`. Ask the user which one they mean. **Do NOT proceed until a unique match is confirmed.**

### Fase 2: Mostrar resumen y pedir confirmación

Show a clear summary **before any write call**:

```
📋 Resumen de entrega

- **Tarea:** {title} (task_id: {task_id})
- **Tipo:** {task_type}
- **Estado actual:** {task_status} / {revision_status}
- **Cohorte:** {cohort name}
- **github_url:** {url provided by user, or "no especificada"}
- **live_url:** {url provided by user, or "no especificada"}

⚠️ Esto marcará la tarea como entregada para revisión.
¿Confirmas la entrega? (responde "sí" o "confirmo")
```

**STOP HERE.** Do not proceed without explicit user confirmation ("sí", "confirmo", "dale", "go", "yes", "ok" or similar). Any ambiguous answer means "wait for confirmation".

### Fase 3: Ejecutar entrega (solo si confirmado)

1. **PUT /v1/assignment/task/{task_id}** — send the URLs in the body:
   ```json
   {
     "github_url": "https://github.com/user/repo",
     "live_url": "https://example.com"
   }
   ```
   - Only include URLs that the user provided. If they gave only one, send only that one.
   - Wait for successful response (200/201).

2. **POST /v1/assignment/task/{task_id}/deliver** — mark as delivered.
   - Wait for successful response.

3. Report the result clearly:
   - ✅ if both calls succeeded
   - ❌ if either failed, with the specific error

### Phase 2b: URL ambiguity

If the user provided URLs but didn't specify which is which:
- If they gave one URL that looks like a GitHub repo → treat as `github_url`
- If they gave one URL that doesn't look like GitHub → treat as `live_url`
- If they gave two URLs → first is typically `github_url`, second `live_url`
- If truly ambiguous → ask them to clarify which is which before proceeding

## Output format

### Before confirmation
```
📋 Resumen de entrega
- **Tarea:** {title} (id: {task_id})
- **Tipo:** {task_type}
- **Estado:** {task_status}
- **Cohorte:** {cohort}
- **URLs a enviar:**
  - github_url: {url}
  - live_url: {url}

⚠️ Esto marcará la tarea como entregada para revisión.
¿Confirmas la entrega?
```

### After success
```
✅ Entrega completada
- **Tarea:** {title} (id: {task_id})
- **URLs actualizadas:** github_url + live_url
- **Estado:** entregada para revisión
```

### On error
```
❌ Error en la entrega
- **Tarea:** {title}
- **Error:** {detail from API}
```

## Limitación conocida: POST /deliver

El endpoint `POST /v1/assignment/task/{task_id}/deliver` devuelve **405 (Method Not Allowed)** en la instancia actual de la API (breathecode.herokuapp.com). Se probaron POST, PUT y PATCH — ninguno funciona.

Esto puede deberse a:
- El endpoint no está habilitado para el rol de estudiante.
- Requiere un paso adicional en la plataforma web.
- La entrega se marca automáticamente cuando el instructor revisa el repo vinculado.

**Workaround:** Si el deliver falla, la skill igual ha actualizado las URLs vía PUT. Informa al usuario que puede necesitar completar la entrega desde la web de 4Geeks, o que la revisión se activará al vincular el repo.

## Constraints

- Never log or echo the raw token value.
- Never hardcode the token; always read from `.env`.
- **NEVER execute PUT or POST without explicit user confirmation.** This is the most important constraint.
- Always deduplicate by `associated_slug` to avoid phantom entries from macro-cohorts.
- If API call fails, report the specific error and continue with remaining calls.
- Only send URLs the user actually provided. Don't fabricate URLs.
- If the task already has URLs from a previous delivery, show them and ask if the user wants to update them.
