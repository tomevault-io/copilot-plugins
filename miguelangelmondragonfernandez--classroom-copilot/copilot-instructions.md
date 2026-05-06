## 04-ai-integrations

> Reglas para integración con IA (Gemini) y APIs de Google (Classroom y Drive).


# Integraciones IA y Google APIs

## Gemini / IA — Reglas de consumo

### Control de tokens
1. **Antes** de llamar a Gemini, verificar que `perfiles.token_balance > 0`.
2. **Después** de una respuesta exitosa, descontar el costo estimado:
   - Actualizar `token_balance = token_balance - costo`
   - Actualizar `total_consumed = total_consumed + costo`
3. Si el saldo es insuficiente, responder con `HTTP 402` y mensaje claro al usuario.

```js
// Ejemplo de flujo en un service
const perfil = await getPerfil(userId);
if (perfil.token_balance <= 0) throw new Error('Saldo de tokens insuficiente');

const result = await gemini.generateContent(prompt);

await deductTokens(userId, estimatedCost);
return result;
```

### Prompts
- Definir prompts del sistema en constantes o archivos `.js` separados dentro del módulo `ai`.
- Nunca construir prompts concatenando directamente datos del usuario sin sanitizar.
- Proporcionar ejemplos de formato esperado en el prompt cuando la respuesta sea JSON estructurado.

## Google Classroom API
- Toda interacción con Classroom (`courses`, `topics`, `courseWork`, `announcements`) ocurre **exclusivamente** en el backend a través del cliente en `/backend/src/clients/`.
- Al crear un **topic** en Classroom, guardar el `classroom_topic_id` en la tabla correspondiente (`ciclos`).
- Al crear un **courseWork** (actividad), guardar el `course_work_id` en `actividades_evaluables`.
- Al eliminar un nodo en la BD, propagar la eliminación a Classroom con manejo explícito de errores (anotado en logs, no silenciado).

## Google Drive API
- Los archivos generados (Docs, Slides) se crean y comparten a través de Drive API en el backend.
- Guardar siempre `drive_file_id` y `drive_url` en `materiales_generados` tras la creación.
- Para selección de archivos desde el cliente, usar únicamente `googlePicker.js` — nunca llamar a Drive API desde el frontend directamente.

## Gestión de errores en APIs externas
- Envolver todas las llamadas a Google APIs y Gemini en `try/catch` con logging a Pino.
- Si una llamada externa falla, **no** dejar un registro a medias en la BD — usar transacciones o rollback.
- Propagar al frontend un error descriptivo (nunca exponer el detalle interno de Google o Gemini).

---
> Source: [MiguelAngelMondragonFernandez/Classroom-copilot](https://github.com/MiguelAngelMondragonFernandez/Classroom-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
