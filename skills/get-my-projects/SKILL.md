---
name: "get-my-projects"
description: "Retrieve assigned projects list with status from 4Geeks API"
---

# Get My Projects

Fetches all projects assigned to the user from the 4Geeks BreatheCode API, deduplicates, sorts chronologically, and displays their status.

## When to use

- "¿Qué proyectos tengo?" / "Mis proyectos" / "Lista de proyectos"
- "¿Qué me queda por entregar?"
- Before checking specific project details

## API Details

- **Endpoint:** `GET /v1/assignment/user/me/task`
- **Base URL:** `https://breathecode.herokuapp.com`
- **Auth header:** `Authorization: Token <TOKEN_4GEEKS>`
- **Token source:** `.env` file in workspace root, variable `TOKEN_4GEEKS`
- **Query params:** `task_type=PROJECT&limit=100`

## Response fields per project

- `title` — project name
- `associated_slug` — unique project identifier (key for deduplication)
- `task_status` — `PENDING` or `DONE`
- `revision_status` — `APPROVED`, `REJECTED`, or `PENDING`
- `opened_at` — when the task was assigned (used for sorting)
- `delivered_at` — ISO timestamp if delivered, null otherwise
- `reviewed_at` — ISO timestamp if reviewed, null otherwise
- `cohort.name` / `cohort.id` — cohort info

## Known issue: macro-cohort duplicates

The 4Geeks API assigns the same project to both the macro-cohort (e.g. `spain-aie-pt-4`, id:1727) and each individual micro-cohort. This creates phantom PENDING entries for projects already APPROVED elsewhere.

**Fix:** Deduplicate by `associated_slug`. For each slug, keep the entry with the best status priority:
`APPROVED (4) > REJECTED (3) > DONE+reviewed (2) > DONE (1) > PENDING (0)`

## Workflow

1. Read `TOKEN_4GEEKS` from `.env`.
2. Call `GET /v1/assignment/user/me/task?task_type=PROJECT&limit=100`.
3. Deduplicate by `associated_slug`, keeping highest-priority status per slug.
4. Sort each group by `opened_at` ascending (oldest first = chronological order).
5. Group deduplicated results:
   - **No entregados:** `task_status == "PENDING"`, no `delivered_at`
   - **Entregados (esperando revisión):** `task_status == "DONE"`, no `reviewed_at`
   - **Aprobados:** `revision_status == "APPROVED"`
   - **Rechazados:** `revision_status == "REJECTED"`
6. Display grouped by status, each group sorted by `opened_at` ascending.

## Constraints

- Never log or echo the raw token.
- Always filter `task_type=PROJECT`.
- One paginated call is enough.
