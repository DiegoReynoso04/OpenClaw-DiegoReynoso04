# Notas de integración MCP — OpenClaw (Telegram, Google Docs, Calendar)

- **Túnel SSH**: `8989:127.0.0.1:8989` fallaba porque VS Code mantenía ocupado el puerto 8989; resuelto cerrando el proceso `Code.exe` antes de levantar el túnel.
- **Callback OAuth**: `ERR_CONNECTION_RESET` en el redirect — OpenClaw no expone un servidor HTTP local, por lo que el callback automático no puede completarse.
- **Decisión de flujo**: se adoptó OAuth manual, extrayendo el parámetro `code` desde la URL de redirección en lugar de depender del callback.
- **Expiración de código**: el error *Invalid or expired authorization code* se produce por reuso o demora; el `code` es de un solo uso y vida corta.
- **Mitigación**: generar un código nuevo y ejecutar de inmediato `openclaw mcp login zapier --code <nuevo_codigo>`.
- **Estado final**: credenciales persistidas correctamente y MCP operativo.