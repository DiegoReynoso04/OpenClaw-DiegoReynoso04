# MEMORY.example.md

## OpenClaw

- Agente: Pixel
- Configuración inicial: 2026-07-27
- Gateway local: puerto 18789

## VPS

- Host: <VPS_HOST>

## AI Provider

- Provider: LiteLLM
- Endpoint: llm.4geeks.ai
- Modelo primario: deepseek/deepseek-v4-flash

## Important context

- Pixel debe consultar esta memoria para recuperar información persistente sobre la infraestructura.
- Los datos de infraestructura pueden cambiar y deben verificarse antes de realizar operaciones importantes.

## Herramientas externas

MCP Zapier conectado (OAuth). Acciones autorizadas:

### Google Calendar

| Acción | Alcance | Confirmación |
|---|---|---|
| Find Events / Retrieve Event by ID | calendario "Personal" | no |
| Find Busy Periods | calendario "Personal" | no |
| Find Calendars / Get Calendar Information | — | no |
| Quick Add Event | calendario "Personal" | no |
| Create Detailed Event | calendario "Personal" | no |
| Update Event | eventos creados por Pixel | no |
| Update Event | eventos creados por Diego | **sí** |
| Add Attendee(s) to Event | — | **sí, siempre** |
| Remove Attendee(s) From Event | — | **sí, siempre** |
| Move Event to Another Calendar | — | **sí** |
| Delete Event | — | **sí, siempre** |
| Create Calendar | — | **sí** |

### Google Docs

| Acción | Alcance | Confirmación |
|---|---|---|
| Find a Document | carpeta "Pixel" | no |
| Get Document Content / Tabs Content | doc cuya URL o ID pasa Diego | no |
| Get Document Content / Tabs Content | cualquier otro doc del Drive | **sí** |
| Find Text in Document | docs en carpeta "Pixel" | no |
| Create Document From Text | carpeta "Pixel" | no |
| Create Document From Template | carpeta "Pixel" | no |
| Append Text / Insert Text | docs creados por Pixel | no |
| Append Text / Insert Text | cualquier otro doc | **sí** |
| Format Text | docs creados por Pixel | no |
| Find and Replace Text | docs creados por Pixel | **sí** |
| Insert Image / Replace Image | docs creados por Pixel | **sí** |
| Update Document Properties | docs creados por Pixel | **sí** |
| Upload Document | — | **sí** |

Todo lo que no esté en esta tabla, no está autorizado: pregunta antes.
Ninguna acción de escritura se ejecuta en cadena sin confirmar cada paso.

### GitHub

| Acción | Alcance | Confirmación |
|---|---|---|
| Find Repository / Find Branch | repos de Diego | no |
| Get File Contents | repos de Diego | no |
| Find Issue / Find Pull Request | repos de Diego | no |
| Find User / Find Organization | — | no |
| Check Organization Membership | — | no |
| Create Branch | repos de Diego | no |
| Create or Update File | rama que no sea main/master | **sí** |
| Create or Update File | main / master | **PROHIBIDO** |
| Create Issue | repos propios de Diego | **sí** |
| Create Issue | repos de terceros u organizaciones | **sí, siempre** |
| Update Issue / Add Labels to Issue | issues propias | **sí** |
| Create Comment | — | **sí, siempre** |
| Create Pull Request | sin merge automático | **sí, siempre** |
| Create Pull Request | con merge automático | **PROHIBIDO** |
| Update Pull Request | PRs abiertas por Pixel | **sí** |
| Submit Review | — | **sí, siempre** |
| Delete Branch | — | **sí, siempre** |
| Create Gist | — | **PROHIBIDO** |
| Set Profile Status | — | **PROHIBIDO** |

Regla general de GitHub: todo lo que escriba queda con el nombre de Diego y es
público o visible para colaboradores. Ninguna escritura sin confirmación explícita.
Nunca incluir credenciales, tokens ni rutas de secretos en commits, issues,
comentarios ni gists.

### Telegram (vía Zapier)

| Acción | Alcance | Confirmación |
|---|---|---|
| Send Message | chat con Diego (<TELEGRAM_USER_ID>) | no |
| Send Message | cualquier otro chat, grupo o canal | **sí, siempre** |
| Send Photo | chat con Diego | no |
| Send Photo | cualquier otro chat, grupo o canal | **sí, siempre** |
| React to Message | mensajes de Diego | no |
| React to Message | mensajes de terceros | **sí** |
| Send Poll | — | **sí, siempre** |

Nota: el canal nativo de OpenClaw con Diego no pasa por esta tabla — es la vía
normal de conversación. Estas acciones son las de Zapier, y su riesgo está en
escribir a alguien que no es Diego.

Nunca enviar respuestas a medias, borradores ni salidas de diagnóstico a un chat
que no sea el de Diego.