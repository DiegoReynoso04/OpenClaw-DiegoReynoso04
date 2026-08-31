---
name: "get-pending-work"
description: "Shows specific pending work: tasks not yet delivered or approved from the 4Geeks API"
---

# Get Pending Work

Shows exactly what's left to do: tasks not yet approved, grouped by type, with enough detail to act on each one.

## When to use

- "¿Qué me falta?" / "Trabajos pendientes" / "¿Qué tengo que hacer?"
- "¿Qué no he entregado?" / "¿Qué está rechazado?"
- Quick triage of what needs attention

## API Details

- **Base URL:** `https://breathecode.herokuapp.com`
- **Auth header:** `Authorization: Token <TOKEN_4GEEKS>`
- **Token source:** `.env` file in workspace root, variable `TOKEN_4GEEKS`

### Endpoints

1. `GET /v1/assignment/user/me/task?limit=200` — all tasks assigned to the user
2. `GET /v1/admissions/user/me` — user profile + active cohort info (for context)

## Workflow

1. Read `TOKEN_4GEEKS` from `.env` in workspace root.
2. Call `GET /v1/assignment/user/me/task?limit=200` to get all tasks.
3. Call `GET /v1/admissions/user/me` to get active cohort(s).
4. Deduplicate by `associated_slug`. Priority: `APPROVED (4) > REJECTED (3) > DONE (2) > PENDING (0)`.
5. Filter: keep only tasks where final status is NOT `APPROVED` AND `task_type` is `PROJECT` (skip LESSON, QUIZ, and EXERCISE).
6. Classify each remaining task:
   - **🔴 Rechazados:** `revision_status == "REJECTED"` — need corrections and re-delivery
   - **🟡 Entregados (esperando revisión):** `task_status == "DONE"` but no `reviewed_at`
   - **⚪ No entregados:** `task_status == "PENDING"`, no `delivered_at`
7. Sort within each group by `opened_at` ascending (oldest first = assigned earliest = deliver first).
8. Present as a single ordered list from most urgent to least, grouped by status: Rechazados → No entregados → Entregados sin revisar.

## Output format

### 🔴 Rechazados (corregir y re-entregar)
For each:
- **Título** — `title`
- Tipo — `task_type` (PROJECT / EXERCISE / LESSON)
- Entregado el — `delivered_at` (fecha)
- Feedback: if available from last review

### ⚪ No entregados
For each:
- **Título** — `title`
- Tipo — `task_type`
- Asignado el — `opened_at`

### 🟡 Entregados (esperando revisión)
For each:
- **Título** — `title`
- Tipo — `task_type`
- Entregado el — `delivered_at`

### 📊 Resumen rápido
- Total pendientes: X (Y rechazados, Z sin entregar, W en revisión)
- Cohorte(s): name(s)

## Constraints

- Never log or echo the raw token value.
- Never hardcode the token; always read from `.env`.
- Always deduplicate by `associated_slug` to avoid phantom entries from macro-cohorts.
- If API call fails, report the specific error and continue with remaining calls.
- Keep output concise — this is a triage view, not a full report.
