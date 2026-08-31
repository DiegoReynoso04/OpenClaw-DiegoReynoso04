---
name: "get-progress-summary"
description: "Fetch and summarize overall student progress from the 4Geeks BreatheCode API"
---

# Get Progress Summary

Shows overall course progress using the same metric as the 4Geeks platform (`completion.overall.percent`), plus a detailed breakdown by task type.

## When to use

- "¿Cómo voy en el curso?" / "Mi progreso" / "Resumen de progreso"
- "¿Cuánto he avanzado?" / "¿Qué porcentaje llevo?"

## API Details

- **Base URL:** `https://breathecode.herokuapp.com`
- **Auth header:** `Authorization: Token <TOKEN_4GEEKS>`
- **Token source:** `.env` file in workspace root, variable `TOKEN_4GEEKS`

### Endpoints to call

1. `GET /v1/admissions/user/me` — user profile + cohort enrollments with `completion` data
2. `GET /v1/assignment/user/me/task?limit=200` — all tasks for breakdown

Do NOT call the cohort endpoint separately. Do NOT filter by cohort — the API auto-filters.

## Workflow

1. Read `TOKEN_4GEEKS` from `.env`.
2. Call `GET /v1/admissions/user/me` → extract profile + `cohorts[]`.
   - For each active cohort, read `completion.overall` (`total`, `completed`, `percent`).
   - Calculate weighted overall: `sum(completed) / sum(total)` across all active cohorts with tasks.
3. Call `GET /v1/assignment/user/me/task?limit=200` → all tasks.
4. Deduplicate by `associated_slug`. Priority: `APPROVED (4) > REJECTED (3) > DONE (2) > PENDING (0)`.
5. Group by `task_type` and count by status within each type.

## Summary structure

### 👤 Perfil
- Nombre, email, cohorte(s) activo(s)

### 📊 Avance general del curso
- **X%** (weighted from `completion.overall` — same as the platform)
- Show per-cohort breakdown: cohort name, completed/total, percent

### 📋 Desglose por tipo de tarea
**📁 Proyectos** — Aprobados / Entregados / Rechazados / Pendientes / Avance %
**📝 Ejercicios** — same breakdown
**📚 Lecciones** — same breakdown

### 🎓 Estado general
- Sugerencia breve basada en el estado

## Constraints

- Never log or echo the raw token value.
- Never hardcode the token; always read from `.env`.
- The overall % MUST come from `completion.overall` fields, NOT from counting tasks.
- Deduplicate tasks by `associated_slug` to avoid phantom PENDING entries from macro-cohorts.
- Group task breakdown by `task_type` so numbers are comparable with get-my-projects.
- If any API call fails, report the specific failure and continue with remaining calls.
