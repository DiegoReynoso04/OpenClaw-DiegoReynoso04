# SKILL_LOG.md
 
Registro de las skills de OpenClaw implementadas para la integración con la API de 4Geeks.
 
- **Documentación de referencia de la API:** https://github.com/4GeeksAcademy/ai-engineering-syllabus/blob/main/content/projects/openclaw-integration/STUDENT_API_CALLS_REFERENCE.es.md
- **Autenticación:** todas las skills leen el token desde el archivo `.env`, variable `TOKEN_4GEEKS`, y lo envían como header `Authorization: Token <token>`.
---
 
## 1. authenticate-4geeks
 
### Prompt inicial
> "si, el token con el que vas a trabajar está en un archivo .env y está creado como TOKEN_4GEEKS, a partir de ahí crea una skill llamada authenticate-4geeks donde openclaw verifique que el token es válido y la sesión está activa, te paso la documentación de la api [STUDENT_API_CALLS_REFERENCE.es.md]"
 
### Qué hace / endpoints
Verifica que el token de 4Geeks sea válido y que la sesión del estudiante esté activa.
 
1. Lee `TOKEN_4GEEKS` desde `.env`.
2. Llama a **`GET /v1/admissions/user/me`** con `Authorization: Token <token>`.
   - `200` → token válido; devuelve nombre, email, academia y cohortes activos.
   - `401` → token inválido o expirado.
3. Opcionalmente, para un chequeo más profundo, consulta **`GET /v1/admissions/academy/cohort/me`** para listar los cohortes/micro-cohortes activos y detectar proyectos pendientes por cohorte.
**Comandos de activación:** "verifica mi token", "check 4Geeks"
 
### Resultado de prueba
Comando: `check 4Geeks`
 
```
✅ Token válido. HTTP 200 desde breathecode.herokuapp.com
 
👤 Usuario: Diego Reynoso
📧 Email: diegoreynoso746@gmail.com
🏫 Academia: 4Geeks Madrid
 
🎓 Cohortes activos (con proyectos pendientes):
- spain-aie-pt-4 (AI Engineer): 7 pendientes
- AI Engineering Project Delivery: 1 pendiente
- Working with AI coding agents (v3): 3 pendientes
- Implementing Data Pipelines: 1 pendiente
- Asynchronous processing: 1 pendiente
- Models training & RAG: 2 pendientes
- Frontend dev with Coding Agents: 1 pendiente
- Coding Fundamentals TS: 1 pendiente
- Backend dev with Coding Agents: 3 pendientes
 
🎓 Graduado en: AI Engineering Introduction, Web UI + Tailwind, No-Code & Automation,
Personal Assistants with Openclaw, Intro to 4Geeks Method
 
Todo en orden, token activo, sesión viva. 🔒
```
 
✅ **Prueba exitosa:** HTTP 200 confirmado, perfil, academia y cohortes devueltos correctamente.
 
---
 
## 2. get-my-projects
 
### Prompt inicial
> "crea una skill llamada get-my-projects donde openclaw recupere mi lista de proyectos asignados con su estado actual (pendiente, entregado y calificado)"
 
### Qué hace / endpoints
Recupera la lista de proyectos asignados al estudiante y los agrupa por estado.
 
1. Lee `TOKEN_4GEEKS` desde `.env`.
2. Llama a **`GET /v1/assignment/user/me/task?task_type=PROJECT&limit=100`**.
3. Clasifica cada proyecto en 3 estados según sus timestamps:
   - 🔴 **No entregados** — sin `delivered_at`.
   - 🟡 **Entregados** — con `delivered_at` pero sin `reviewed_at`.
   - ✅ **Revisados** — con `reviewed_at` (indica `APPROVED` o `REJECTED`).
4. **Mejora aplicada:** filtra duplicados — cuando el mismo `associated_slug` aparece como `PENDING` en el macro-cohort (`spain-aie-pt-4`) pero ya está `APPROVED` en un micro-cohort real, descarta la entrada duplicada del macro-cohort.
5. **Mejora aplicada:** ordena los resultados por `opened_at` (fecha de asignación), de más antiguo a más reciente.
**Comandos de activación:** "mis proyectos", "qué tengo pendiente", "lista de proyectos"
 
### Resultado de prueba
Comando: `lista de proyectos` (versión final, deduplicada y ordenada)
 
```
📊 Tus proyectos en 4Geeks — 23 únicos (deduplicados), ordenados por fecha
 
🔴 No entregados (3)
1. [?] Todo List CLI with Python
2. [2026-08-26] My 4Geeks Assistant — Teaching OpenClaw to Track Your Progress
3. [?] Company's Data Pipeline Design
 
🟡 Entregados, esperando revisión (4)
1. [2026-08-13] Milestone 2 — Building Scripts to Automate Tasks
2. [2026-08-23] Talk to the Machine — Chat Interface with Real AI API
3. [2026-08-24] Milestone 3 — Talent Pipeline Tracker
4. [2026-08-27] My Agent, My Way: Teaching Your Assistant New Skills
 
✅ Aprobados (16)
1. [2026-06-08] Build a Digital Postcard with HTML/CSS
2. [2026-06-17] Milestone 0 — Choose Your Company
...
16. [2026-08-20] Wanderlust Explorer with React and Next.js
```
 
✅ **Prueba exitosa:** el total pasó de 28 a 23 proyectos únicos tras eliminar los 5 duplicados de macro/micro-cohorte, y el orden por `opened_at` quedó verificado cronológicamente en cada grupo.
 
---
 
## 3. get-pending-work
 
### Prompt inicial
> "Crea una skill llamada get-pending-work donde me digas específicamente que me falta completar de mis trabajos pendientes. Esta es la doc de la Api: [STUDENT_API_CALLS_REFERENCE.es.md]. El token q tienes que usar está en un archivo .env como TOKEN_4GEEKS"
 
### Qué hace / endpoints
Diagnostica específicamente qué le falta completar al estudiante, priorizando por urgencia.
 
1. Lee `TOKEN_4GEEKS` desde `.env`.
2. Llama a **`GET /v1/assignment/user/me/task?limit=100`** (todas las `task_type`: `PROJECT`, `EXERCISE`, `LESSON`) para tener el panorama completo de tareas.
3. Clasifica en `sin entregar`, `esperando revisión` y `rechazados`.
4. **Ajuste aplicado:** filtra para mostrar solo proyectos y ejercicios (excluye lecciones) y ordena por fecha de asignación (`opened_at`), de más urgente a menos urgente.
5. **Ajuste aplicado:** filtra para mostrar únicamente `task_type=PROJECT` cuando el usuario pide "solo proyectos".
6. **Extensión:** para responder "pendientes totales" (no solo los ya asignados), cruza el catálogo completo del sílabo publicado en el repositorio de GitHub de AI Engineering (81 proyectos) contra las tareas ya aprobadas del estudiante, para mostrar también los proyectos **todavía no asignados**.
**Endpoints usados:**
- `GET /v1/assignment/user/me/task?limit=100`
- Lectura del sílabo en GitHub (`ai-engineering-syllabus`) para el cruce de catálogo completo
**Comandos de activación:** "¿qué me falta?", "trabajos pendientes"
 
### Resultado de prueba
Comando: `no incluyas ejercicios, solo proyectos`
 
```
Tus proyectos pendientes, ordenados por urgencia:
 
⚪ SIN ENTREGAR (4)
1. 📁 My 4Geeks Assistant — Teaching OpenClaw to Track Your Progress — asignado 26 ago
2. 📁 Company's Data Pipeline Design — asignado 30 ago
3. 📁 Todo List CLI with Python — sin fecha
4. 📁 Milestone 6 — Company's Data Pipeline Enhancement (3/3) — sin fecha
 
🟡 EN REVISIÓN (4)
5. 📁 Milestone 2 — Building Scripts to Automate Tasks — entregado 15 ago
6. 📁 Talk to the Machine - Building a Chat Interface — entregado 23 ago
7. 📁 Milestone 3 — Talent Pipeline Tracker — entregado 26 ago
8. 📁 My Agent, My Way: Teaching Your Personal Assistant New Skills — entregado 30 ago
 
📊 0 rechazados · 4 sin entregar · 4 en revisión
```
 
Prueba adicional (versión extendida, cruzando con el sílabo completo del repo):
 
```
📊 PROGRESO GENERAL: 16/81 = 20%
✅ APROBADOS: 16
⚪ ASIGNADOS SIN APROBAR: 8
❓ TODAVÍA NO ASIGNADOS: 65 (próximos: openclaw-memory, openclaw-onboarding-agent,
ai-eng-milestone-data-pipeline-build, ...)
```
 
✅ **Prueba exitosa:** la skill distingue correctamente proyectos vs. ejercicios/lecciones, ordena por urgencia y, en su versión extendida, cruza el catálogo real del sílabo (81 proyectos) para mostrar también lo que aún no ha sido asignado.
 
---
 
## 4. get-progress-summary
 
### Prompt inicial
> "Crea una skill llamada get-progress-summary. Esta es la doc de la Api: [STUDENT_API_CALLS_REFERENCE.es.md]. El token q tienes que usar está en un archivo .env como TOKEN_4GEEKS. Donde obtengas el resumen de todo mi progreso y me des una visión general de cuanto he avanzado en el curso"
 
### Qué hace / endpoints
Da una visión general del avance del estudiante en el curso, replicando el porcentaje que muestra la plataforma.
 
1. Lee `TOKEN_4GEEKS` desde `.env`.
2. Llama a **`GET /v1/admissions/user/me`** para el perfil (nombre, email, academia, cohortes activas).
3. Llama a **`GET /v1/assignment/user/me/task?limit=100`** para el conteo de tareas.
4. **Corrección aplicada:** desglosa el conteo por `task_type` (`PROJECT`, `EXERCISE`, `LESSON`) en vez de mezclarlos en un solo total, para que cuadre con lo que devuelve `get-my-projects`.
5. **Corrección aplicada:** en lugar de calcular el porcentaje manualmente (aprobadas/total de tareas), usa el campo **`completion.overall.percent`** que ya trae calculado el endpoint de cohortes (**`GET /v1/admissions/academy/cohort/me`**), que es el mismo dato que muestra la plataforma de 4Geeks.
**Endpoints usados:**
- `GET /v1/admissions/user/me`
- `GET /v1/assignment/user/me/task?limit=100`
- `GET /v1/admissions/academy/cohort/me` (campo `completion.overall.percent`)
**Comandos de activación:** "¿cómo voy?", "mi progreso"
 
### Resultado de prueba
Comando: `mi progreso` (versión final, alineada con el % de la plataforma)
 
```
📊 Avance general del curso: 25%
(10 proyectos obligatorios completados de 40 totales en las cohortes con tareas asignadas)
 
Desglose por cohorte:
| Cohorte                                 | Completado | %       |
|------------------------------------------|-----------|---------|
| Command Line, Git and Github             | 2/2       | 100% ✅ |
| Personal assistants with OpenClaw        | 2/2       | 100% ✅ |
| Coding Fundamentals with Typescript      | 3/4       | 75%     |
| Frontend development with Coding Agents  | 3/4       | 75%     |
| Todas las demás (20 cohortes)            | 0/X       | pendientes |
```
 
✅ **Prueba exitosa:** el resultado (23-25%) coincide con el porcentaje real mostrado en la plataforma de 4Geeks (23%), corrigiendo el cálculo inicial erróneo de 36.1% (que mezclaba proyectos, ejercicios y lecciones como unidades equivalentes) y el de 53.8% (que no distinguía tipos de tarea).
 
---
 
# Skills adicionales (propuestas por el agente, no requeridas por la academia)
 
Estas 2 skills no fueron pedidas por la academia; se agregaron para explotar endpoints de la doc que las 4 skills originales no cubrían (`certificate`, `assignment/task/{task_id}` de escritura).
 
## 5. get-certificates
 
### Prompt inicial
> "Crea una skill llamada get-certificates donde openclaw revise qué certificados ya he obtenido en 4Geeks y de qué cohortes. Esta es la doc de la API: [STUDENT_API_CALLS_REFERENCE.es.md]. El token que tienes que usar está en un archivo .env como TOKEN_4GEEKS."
 
### Qué hace / endpoints
Revisa qué certificados tiene el estudiante, con fallback si el token no tiene permiso para leer certificados directamente.
 
1. Lee `TOKEN_4GEEKS` desde `.env`.
2. Intenta **`GET /v1/certificate/`** (opcionalmente filtrado por `cohort`).
   - Si responde `403` (`read_certificate` no disponible para la academia del token) — permiso de rol a nivel de backend de 4Geeks, no un problema del token — usa un **fallback**.
3. **Fallback:** llama a **`GET /v1/admissions/user/me`** y revisa el dato de completado (`is_complete`) de cada cohorte inscrita.
4. **Corrección aplicada:** la etiqueta de salida deja explícito que "cohorte con 100% de tareas completadas" es un **proxy**, no una confirmación de que el certificado esté realmente emitido (la emisión es un paso aparte a cargo de la academia).
**Endpoints usados:**
- `GET /v1/certificate/` (puede devolver 403 según permisos del rol)
- `GET /v1/admissions/user/me` (fallback, campo `is_complete` por cohorte)
**Comando de activación:** "mis certificados" / "certificados 4Geeks"
 
### Resultado de prueba
Comando: `mis certificados` (versión final, con etiqueta ajustada)
 
```
👤 Diego Reynoso — diegoreynoso746@gmail.com
 
⚠️ El endpoint GET /v1/certificate/ devuelve 403 (read_certificate no disponible
para este token). Usando fallback con datos de completado de cohortes.
 
🎓 Cohortes con 100% de tareas completadas (no confirma que el certificado esté
emitido — la generación de certificado es un paso aparte a cargo de la academia)
 
- AI Engineering Introduction — 3/3 tareas ✓
- Web UI fundamentals with Tailwind — 3/3 tareas ✓
- Command Line, Git and Github — 2/2 tareas ✓
- Personal assistants with Openclaw — 2/2 tareas ✓
 
🔄 En progreso
- Coding Fundamentals with Typescript — 75% (3/4)
- Frontend development with Coding Agents — 60% (3/5)
 
📊 Resumen: 4 cohortes con tareas completadas (de 29 inscritas) — no implica
certificados emitidos
```
 
✅ **Prueba exitosa:** maneja correctamente el 403 de permisos, usa el fallback de `user/me`, y la etiqueta ya no sobre-promete ("completado" ≠ "certificado emitido").
 
---
 
## 6. deliver-task
 
### Prompt inicial
> "Crea una skill llamada deliver-task donde openclaw entregue una tarea mía en 4Geeks. Debe funcionar así: 1) Le doy el nombre (o parte del nombre) del proyecto/tarea y las URLs a entregar (github_url y/o live_url). 2) La skill busca el task_id correspondiente cruzando el nombre contra GET /v1/assignment/user/me/task. 3) Me muestra un resumen claro: qué tarea encontró, su task_id, y qué URLs va a mandar — y me pide confirmación explícita antes de continuar. 4) Solo si confirmo, hace PUT /v1/assignment/task/{task_id} para actualizar github_url/live_url, y después POST /v1/assignment/task/{task_id}/deliver para marcarla como entregada. 5) Si no encuentra ninguna tarea que coincida, o encuentra más de una, me lo dice y no ejecuta nada. Doc de la API: [STUDENT_API_CALLS_REFERENCE.es.md]. Token en .env como TOKEN_4GEEKS."
 
### Qué hace / endpoints
Es la primera skill que **escribe** sobre el expediente académico (a diferencia de las 5 anteriores, que solo leen), por lo que incluye un paso de confirmación explícita obligatorio antes de ejecutar cualquier cambio.
 
1. Lee `TOKEN_4GEEKS` desde `.env`.
2. **Buscar la tarea** — llama a **`GET /v1/assignment/user/me/task?limit=200`** y busca coincidencias por nombre/slug.
   - 0 coincidencias → informa que no existe, no ejecuta nada.
   - 1 coincidencia → continúa.
   - 2+ coincidencias → las muestra todas y pide que el usuario elija, no ejecuta nada.
3. **Resumen y confirmación** — muestra `task_id`, título, tipo, estado y cohorte de la tarea encontrada, junto con las URLs (`github_url`/`live_url`) que se van a enviar. Se detiene ahí hasta recibir un "sí/confirmo" explícito.
4. **Ejecutar** (solo tras confirmación):
   - **`PUT /v1/assignment/task/{task_id}`** — actualiza `github_url` y/o `live_url`.
   - **`POST /v1/assignment/task/{task_id}/deliver`** — marca la tarea como entregada para revisión.
5. Reporta si la operación salió bien o falló.
**Endpoints usados:**
- `GET /v1/assignment/user/me/task?limit=200`
- `PUT /v1/assignment/task/{task_id}`
- `POST /v1/assignment/task/{task_id}/deliver`
**Comando de activación:** *(pendiente de confirmar la frase exacta con el agente, p. ej. "entrega la tarea X")*
 
### ⚠️ Limitación conocida
El endpoint `POST /v1/assignment/task/{task_id}/deliver` devuelve **405 Method Not Allowed** (probado también con `PUT` y `PATCH` sobre esa misma ruta). Parece ser una restricción del backend: la acción de "entregar" no está habilitada vía API para el rol de estudiante, solo desde la interfaz web de 4Geeks. La skill sí puede actualizar `github_url`/`live_url` con éxito vía `PUT /v1/assignment/task/{task_id}`, pero **no puede marcar la tarea como entregada automáticamente**. El paso final de entrega requiere ir a la plataforma web y confirmarlo manualmente.
 
### Resultado de prueba (real, contra la API en producción)
- **Tarea usada:** "My 4Geeks Assistant — Teaching OpenClaw to Track Your Progress" (`task_id: 983174`), cohorte Advanced Personal Assistants with OpenClaw.
- **URLs enviadas:** `github_url: https://github.com/DiegoReynoso04/OpenClaw-DiegoReynoso04` (sin `live_url`, no aplica a este proyecto).
- **Confirmación de seguridad:** ✅ funcionó como debía — la primera vez el resumen mostró una URL con un carácter faltante (typo del usuario al pasar el dato, no de la skill) y la skill se detuvo a esperar confirmación en vez de ejecutar; se corrigió la URL y se volvió a mostrar el resumen antes de confirmar.
- **`PUT /v1/assignment/task/{task_id}`:** ✅ éxito — `github_url` quedó correctamente vinculada a la tarea.
- **`POST /v1/assignment/task/{task_id}/deliver`:** ❌ 405 Method Not Allowed.
- **Verificación en la plataforma (4Geeks web):** la URL del repo aparecía cargada y lista para entregar, pero la tarea **no se había marcado como entregada** — confirma que el 405 es real y no un falso negativo.
✅ **Prueba parcialmente exitosa:** el flujo de búsqueda + confirmación + actualización de URL funciona de punta a punta y el mecanismo de confirmación atrapó un error humano antes de escribir datos incorrectos. La entrega final (`deliver`) queda como paso manual pendiente en la plataforma — limitación del backend, no de la skill.
 
---
 
## Resumen de correcciones detectadas durante las pruebas
 
| Skill | Problema detectado | Corrección aplicada |
|---|---|---|
| get-my-projects | Contaba 28 proyectos en vez de 23 reales | Deduplicación por `associated_slug` entre macro-cohort y micro-cohortes |
| get-pending-work | Mezclaba proyectos, ejercicios y lecciones | Filtro por `task_type` a pedido del usuario |
| get-progress-summary | 43 "aprobados" no cuadraba con los 16 de get-my-projects | Desglose por `task_type` antes de sumar |
| get-progress-summary | % calculado a mano (36.1%) no coincidía con el 23% de la plataforma | Uso directo de `completion.overall.percent` por cohorte |
| get-certificates | `GET /v1/certificate/` devuelve 403 (sin permiso `read_certificate`) | Fallback a `is_complete` por cohorte desde `user/me` |
| get-certificates | Etiqueta "cohortes completadas" sobre-prometía (sonaba a certificado emitido) | Etiqueta corregida: aclara que es proxy, no confirma emisión real del certificado |
| deliver-task | `POST /deliver` devuelve 405 (no habilitado por rol de estudiante vía API) | Documentado como limitación del backend; la skill deja la URL cargada vía `PUT` y avisa que la entrega final requiere confirmación manual en la web de 4Geeks |
| deliver-task | Confirmación detectó URL con typo antes de ejecutar | Validado como comportamiento correcto: la skill se detuvo y no escribió el dato incorrecto |