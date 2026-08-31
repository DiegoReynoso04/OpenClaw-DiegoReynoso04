---
name: "get-certificates"
description: "Shows certificates obtained from 4Geeks and their associated cohorts"
---

# Get Certificates

Lists all certificates the user has earned on 4Geeks Academy, with cohort details and verification links.

## When to use

- "¿Qué certificados tengo?" / "Mis certificados" / "¿De qué cohortes me certifiqué?"
- "¿Tengo certificado de X cohorte?"
- "Dame mis certificados"

## API Details

- **Base URL:** `https://breathecode.herokuapp.com`
- **Auth header:** `Authorization: Token <TOKEN_4GEEKS>`
- **Token source:** `.env` file in workspace root, variable `TOKEN_4GEEKS`

### Endpoints

1. `GET /v1/certificate/` — all certificates of the authenticated user
2. `GET /v1/admissions/academy/cohort/me` — cohort enrollments (for cohort context like name, slug, educational_status)
3. `GET /v1/admissions/user/me` — user profile (for name and email)

## Workflow

1. Read `TOKEN_4GEEKS` from `.env` in workspace root.
2. Call `GET /v1/admissions/user/me` to get user info (first_name, last_name, email) and cohort enrollments with completion data.
3. Call `GET /v1/certificate/` to get all certificates.
   - **If 403 (missing `read_certificate` permission):** skip and use fallback.
   - **Fallback:** derive "completed" cohorts from `user/me` response: any cohort where `completion.is_complete == true` counts as certified.
4. Call `GET /v1/admissions/academy/cohort/me` (with `Academy` header) for additional cohort context. If it fails or returns empty, rely on the cohort data already in `user/me`.
5. Sort completed cohorts by `created_at` descending (most recent first).
6. Present results clearly.

### Fallback logic

If the certificate endpoint returns 403 or any error, the skill uses this heuristic:
- A cohort is considered **completed** when `completion.is_complete == true`.
- A cohort is **in progress** when `completion.is_complete == false` and `educational_status == "ACTIVE"`.
- All cohort info (name, slug, academy, kickoff_date, completion stats) comes from the `cohorts[]` array in the `user/me` response.
- This is noted in the output with a ⚠️ indicator.

### Limitación conocida del fallback

El endpoint `GET /v1/certificate/` devuelve **403** con el mensaje:
`"You (user: XXXX) don't have this capability: read_certificate for academy X"`

Esto significa que el token actual no tiene el permiso `read_certificate` en la academia. Es un permiso de role en el backend de 4Geeks, no un problema del token en sí.

Por eso, cuando se usa el fallback, el dato de "cohortes completadas" es un **proxy**: indica que todas las tareas obligatorias están marcadas como hechas, pero **no confirma que el certificado haya sido emitido**. La generación de certificados es un proceso separado a cargo de la academia que puede tardar o requerir pasos adicionales (verificación manual, graduación oficial, etc.).

En la salida se debe usar la etiqueta:
`🎓 Cohortes con 100% de tareas completadas (no confirma que el certificado esté emitido — la generación de certificado es un paso aparte a cargo de la academia)`

Si en el futuro el endpoint de certificados devuelve 200 con datos reales, se usa la sección "🏆 Certificados obtenidos" con la info directa de la API.

## Output format

### 👤 Perfil
- Nombre, email

### 🏆 Certificados obtenidos
For each certificate:
- **Cohorte** — cohort name/slug
- **Fecha** — `created_at` (formatted)
- **Academia** — academy name if available
- **Estado** — `educational_status` from the cohort (GRADUATED, ACTIVE, etc.)
- **Link de verificación** — public URL if `token` field is present: `https://breathecode.herokuapp.com/v1/certificate/{token}`

### 📊 Resumen
- Total de certificados: X (si el endpoint principal respondió)
- O bien: Total de cohortes con tareas completadas: X (si se usó fallback — no implica certificados emitidos)
- Lista de cohortes relevantes

## Constraints

- Never log or echo the raw token value.
- Never hardcode the token; always read from `.env`.
- If API call fails, report the specific error and continue with remaining calls.
- If no certificates found, say so clearly — don't pretend there are none silently.
- The public verification link (`/v1/certificate/{token}`) requires no auth and can be shared.
