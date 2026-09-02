# SKILL_LOG.md

Esto es el registro de las 6 skills que fuimos construyendo para que OpenClaw hable con la API de 4Geeks en mi nombre. Las primeras 4 las pidió la academia como parte del proyecto; las últimas 2 las propuse yo mismo, revisando qué más se podía hacer con la documentación de la API que todavía no estábamos usando.

Todas parten de lo mismo: el token vive en el archivo `.env` como `TOKEN_4GEEKS`, y la API base es `https://breathecode.herokuapp.com`. La doc de referencia es el [STUDENT_API_CALLS_REFERENCE.es.md](https://github.com/4GeeksAcademy/ai-engineering-syllabus/blob/main/content/projects/openclaw-integration/STUDENT_API_CALLS_REFERENCE.es.md) del repo de la academia.

---

## 1. authenticate-4geeks

Esta fue la primera y la más sencilla: solo quería que OpenClaw me confirmara que mi token seguía siendo válido antes de meterme a construir cosas más complicadas encima. El prompt con el que arranqué fue:

> "El token con el que vas a trabajar está en un archivo .env y está creado como TOKEN_4GEEKS, a partir de ahí crea una skill llamada authenticate-4geeks donde openclaw verifique que el token es válido y la sesión está activa, te paso la documentación de la api..."

Lo que hace es simple: toma el token, llama a `GET /v1/admissions/user/me`, y si la respuesta es 200 significa que todo está en orden — me devuelve mi nombre, mi email, mi academia y mis cohortes activos. Si me devolviera un 401 sabría que el token expiró. También puede apoyarse en `GET /v1/admissions/academy/cohort/me` para un chequeo más completo de cohortes.

La probé con el comando "check 4Geeks" y respondió tal cual se esperaba: token válido, HTTP 200, y me trajo mi perfil completo — Diego Reynoso, mi correo, 4Geeks Madrid, la lista de cohortes activos con sus proyectos pendientes, y hasta los cursos donde ya me gradué. Funcionó a la primera.

---

## 2. get-my-projects

Esta la pedí para no tener que entrar a la plataforma cada vez que quería saber en qué estado estaban mis proyectos:

> "crea una skill llamada get-my-projects donde openclaw recupere mi lista de proyectos asignados con su estado actual (pendiente, entregado y calificado)"

Usa `GET /v1/assignment/user/me/task?task_type=PROJECT&limit=100` y agrupa todo en tres cajones: sin entregar, entregado esperando revisión, y ya revisado (aprobado o rechazado).

La primera versión funcionaba, pero algo no cuadraba: me devolvía 28 proyectos, y varios de los que aparecían como "no entregados" en realidad ya los tenía aprobados. Investigando encontramos el motivo — la API de 4Geeks duplica los proyectos entre el macro-cohort (spain-aie-pt-4) y los micro-cohortes reales donde de verdad los entregué, así que el mismo proyecto contaba dos veces. Actualizamos la skill para que, si un proyecto ya aparece aprobado en algún cohorte, ignore la copia "pendiente" del macro-cohort. Después le agregué que ordenara todo por fecha de asignación (`opened_at`), de más viejo a más nuevo.

Con esos dos ajustes, la lista bajó de 28 a 23 proyectos únicos y quedó bien ordenada cronológicamente en cada grupo — la probé pidiendo "lista de proyectos" y los números por fin cerraban.

---

## 3. get-pending-work

Quería algo más específico que "mis proyectos": necesitaba que me dijera, sin rodeos, qué me falta.

> "Crea una skill llamada get-pending-work donde me digas específicamente que me falta completar de mis trabajos pendientes"

Esta trae todas las tareas sin filtrar por tipo (`GET /v1/assignment/user/me/task?limit=100`, así que incluye proyectos, ejercicios y lecciones) y las separa en sin entregar, esperando revisión, y rechazadas.

La fui puliendo en varias vueltas: primero le pedí que ordenara todo por fecha de entrega, de lo más urgente a lo menos urgente; después le pedí que dejara fuera las lecciones y solo mostrara ejercicios y proyectos; y al final le pedí que se quedara solo con proyectos, porque los ejercicios me generaban demasiado ruido. En su versión más ambiciosa, hasta llegó a cruzar el catálogo completo del sílabo publicado en GitHub (81 proyectos en total) contra lo que ya tengo aprobado, para poder decirme no solo lo que ya me asignaron sino también lo que todavía viene más adelante en el programa.

El resultado final, pidiendo solo proyectos, me mostró 4 sin entregar y 4 esperando revisión, con fechas de asignación claras para priorizar. Y en la versión extendida me confirmó que voy 16 de 81 proyectos del sílabo completo, un 20% del camino total.

---

## 4. get-progress-summary

Esta la pedí para tener una foto general de cómo voy en el curso, no proyecto por proyecto sino el panorama completo:

> "Crea una skill llamada get-progress-summary [...] Donde obtengas el resumen de todo mi progreso y me des una visión general de cuanto he avanzado en el curso"

Combina mi perfil (`GET /v1/admissions/user/me`) con el conteo de tareas (`GET /v1/assignment/user/me/task`).

Acá también hubo idas y vueltas. La primera versión me dio un 43 en "aprobados" que no cuadraba para nada con los 16 proyectos aprobados que ya sabía por get-my-projects — resulta que estaba mezclando proyectos, ejercicios y lecciones como si fueran lo mismo. La arreglamos para que desglosara por tipo de tarea, y ahí sí los proyectos coincidieron. Pero después noté que el porcentaje que me daba (36%) tampoco coincidía con el 23% que me muestra la plataforma de 4Geeks directamente. Ahí encontramos que la plataforma usa un campo ya calculado, `completion.overall.percent`, que viene en el endpoint de cohortes — así que actualizamos la skill para usar ese dato directo en vez de inventar su propia fórmula.

Con ese ajuste, el número final rondó el 23-25%, que es justo lo que veo en la plataforma cuando entro. Coincide.

---

## 5. get-certificates

Esta ya no la pidió la academia, la propuse yo después de revisar qué más ofrecía la documentación. Quería saber qué certificados ya tengo:

> "Crea una skill llamada get-certificates donde openclaw revise qué certificados ya he obtenido en 4Geeks y de qué cohortes."

Debía llamar a `GET /v1/certificate/`, pero chocó de entrada con un 403: mi token no tiene el permiso `read_certificate` para mi academia. Esto no es un problema del token en sí, sino un permiso de rol que decide la academia a nivel de backend — no había forma de arreglarlo desde la skill.

Lo que sí se puede hacer es un rodeo: el endpoint `user/me` trae, para cada cohorte, si ya la completé al 100% o no. No es lo mismo que "certificado emitido" — la emisión del certificado suele ser un paso aparte que hace un admin de la academia — así que le pedí que la etiqueta de salida dejara eso bien claro, en vez de sonar a "ya tienes tu certificado".

La probé pidiendo "mis certificados" y respondió exactamente así: avisa del 403, explica que está usando el dato de cohortes completadas como aproximación, y aclara que eso no confirma que el certificado ya esté emitido. Me mostró 4 cohortes al 100% de tareas completadas de las 29 en las que estoy inscrito. Es la respuesta correcta dentro de lo que el permiso permite.

---

## 6. deliver-task

Esta fue la más ambiciosa de las 6, porque es la única que no solo lee datos sino que escribe sobre mi expediente real:

> "Crea una skill llamada deliver-task donde openclaw entregue una tarea mía en 4Geeks. Debe funcionar así: le doy el nombre del proyecto y las URLs a entregar, la skill busca el task_id correspondiente, me muestra un resumen y me pide confirmación explícita antes de tocar nada, y solo si confirmo actualiza las URLs y marca la tarea como entregada."

El flujo quedó en tres pasos: primero busca la tarea por nombre contra `GET /v1/assignment/user/me/task`; si encuentra una sola, me muestra un resumen con el `task_id`, el estado actual y las URLs que va a mandar, y se detiene ahí hasta que yo confirme; y solo si confirmo, ejecuta `PUT /v1/assignment/task/{task_id}` para actualizar las URLs y después `POST /v1/assignment/task/{task_id}/deliver` para marcarla como entregada.

La probé de verdad con un proyecto real: "My 4Geeks Assistant — Teaching OpenClaw to Track Your Progress" (task_id 983174), con el repo `https://github.com/DiegoReynoso04/OpenClaw-DiegoReynoso04`. Y el paso de confirmación demostró su valor de inmediato: la primera vez le pasé mal la URL (me faltó un carácter) y en vez de mandarla así nomás, la skill se detuvo, me mostró el resumen con la URL incorrecta y esperó mi confirmación. Al verla, me di cuenta del error, se lo hice notar, y solo después de corregirlo confirmé la entrega. Justo para eso sirve ese paso.

El `PUT` funcionó perfecto: la URL del repo quedó bien vinculada a la tarea. Pero el `POST /deliver` me devolvió un 405 (Method Not Allowed) — probamos también con PUT y PATCH sobre esa misma ruta, sin suerte. Al final entendimos que es otra limitación de permisos, parecida a la de los certificados: la acción de "entregar" definitiva no está habilitada por API para el rol de estudiante, solo se puede hacer desde la web de 4Geeks. Lo confirmé entrando directamente a la plataforma: la URL del repo ya estaba cargada y lista, pero la tarea seguía sin marcarse como entregada hasta que le di el clic manual ahí mismo.

Entonces la prueba fue un éxito parcial, pero un éxito real: la skill hace bien lo que la API le permite hacer (buscar la tarea, mostrarme un resumen honesto, pedirme confirmación, actualizar la URL), y donde se topa con una pared de permisos, avisa claramente en vez de fingir que la entrega quedó completa. Quedó documentado en el SKILL.md que el paso final de "Entregar" hay que darlo manualmente en la plataforma.

---

## Conclusiones

Mirando las 6 skills en conjunto, casi todos los problemas que aparecieron fueron variaciones del mismo tipo de sorpresa: la API de 4Geeks no siempre da los datos limpios a la primera.

- **Duplicados entre cohortes:** el mismo proyecto puede aparecer dos veces (macro-cohort y micro-cohort), y si no se filtra, los números no cuadran. Nos pasó en get-my-projects.
- **Mezclar tipos de tarea sin querer:** proyectos, ejercicios y lecciones no son comparables, y sumarlos todos juntos da porcentajes que parecen razonables pero están mal. Nos pasó en get-progress-summary.
- **El porcentaje "real" no siempre es el que uno calcularía a mano:** la plataforma tiene su propio campo ya calculado (`completion.overall.percent`), y conviene usar ese en vez de reinventar la fórmula.
- **Permisos de rol que no dependen del token:** tanto los certificados como la entrega final de tareas están bloqueados a nivel de backend para el rol de estudiante. En ambos casos, la solución no fue "arreglar" la skill sino aceptar la limitación, avisar con claridad, y ofrecer la mejor alternativa posible (un fallback en un caso, un aviso de paso manual en el otro).
- **La confirmación explícita en deliver-task valió la pena de inmediato:** apenas empezamos a usarla de verdad, atrapó un error humano (una URL mal escrita) antes de que se guardara. Es la prueba de que ese paso no era solo un capricho de seguridad, sino algo que efectivamente evita errores.