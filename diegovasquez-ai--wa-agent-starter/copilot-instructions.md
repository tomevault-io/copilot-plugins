## wa-agent-starter

> Este repo es **wa-agent-starter**: un boilerplate para agentes de IA de WhatsApp. Node 20 + TypeScript (ESM). Está construido y testeado — tu rol es ayudar al usuario a **configurarlo, extenderlo y desplegarlo**, no reescribir el núcleo.

# CLAUDE.md — guía para Claude Code

Este repo es **wa-agent-starter**: un boilerplate para agentes de IA de WhatsApp. Node 20 + TypeScript (ESM). Está construido y testeado — tu rol es ayudar al usuario a **configurarlo, extenderlo y desplegarlo**, no reescribir el núcleo.

## Onboarding — leelo antes que nada

**Si el usuario escribe `empezar` (o "empecemos", "quiero empezar", "arranquemos",
"ayudame a configurar esto", o cualquier variante de arrancar de cero), ejecuta
el comando `/empezar`.** No improvises tu propia guía y no le vuelques la lista
de pasos completa: ese comando lo lleva de a un paso por vez, verificando cada
uno, desde el repo recién clonado hasta el agente andando en internet.

Quien escribe eso suele ser un alumno que nunca desplegó nada. La palabra suelta
`empezar` es la puerta de entrada del kit: tiene que funcionar sin que sepa que
existen los slash commands.

Los otros dos caminos:
- `/build-agent` — solo la entrevista de personalidad y credenciales, para quien
  ya tiene el entorno andando. El paso 3 de `/empezar` lo llama.
- `npm run setup` — la misma entrevista por terminal, sin Claude Code.

## Arquitectura (lo mínimo para orientarte)
- El **cerebro** (agente) es siempre el mismo. Lo que cambia por proyecto es el **canal** (por dónde entran los mensajes), las **herramientas** y la **personalidad**.
- `src/channels/` — adaptadores de canal (la abstracción clave, `ChannelAdapter`): `web`, `cloud-api`, `ghl` (completos), `chatwoot`, `ycloud` (stubs). Se elige con `CHANNEL_ADAPTER`.
- `src/agent/` — `runner.ts` (loop de tool-calling), `llm.ts` (OpenRouter), `prompt.ts`, `tools/`, `guards/`, `budget.ts`, `tokens.ts`.
- `src/queue/` — buffer con debounce + worker (canales por webhook). El canal `web` procesa en línea, sin Redis.
- `src/memory/` — store: `postgres` o `memory` (elegido con `STORE`).
- `src/security/` — firmas HMAC de webhooks, rate limit, sesiones firmadas.
- `src/notifications/` — avisos: al equipo por el canal (etiqueta en el CRM) y al desarrollador por mail (Resend).
- `config/agent.yaml` — identidad, conocimiento y `guardrails` (editable por no-devs).

## Seguridad — lee esto antes de tocar el agente

El kit tiene cuatro capas de defensa. Si agregas código, respetalas:

1. **`src/agent/guards/input.ts`** — normaliza el mensaje y marca intentos de manipulación.
2. **`src/agent/guards/untrusted.ts`** — vallado de contenido externo. **La regla más importante del repo.**
3. **`src/agent/tools/policy.ts`** — frena las herramientas de escritura.
4. **`src/agent/guards/output.ts`** — revisa la respuesta antes de enviarla.

Reglas que no se negocian:

- **Nunca metas instrucciones dentro de un payload de herramienta.** Los `run()` devuelven DATOS (`{ error: 'la_herramienta_fallo' }`), no órdenes ("no se lo menciones al usuario"). Si el modelo aprende que los payloads traen órdenes, va a obedecer también las que le meta un atacante en una nota del CRM. Qué hacer ante un error se define una sola vez, en el `system_prompt` de `agent.yaml`.
- **Toda herramienta nueva declara `sideEffect`.** `'write'` ante la duda.
- **`returnsUntrustedContent: true`** si el resultado puede traer texto escrito por un tercero (web, CRM, PDF del cliente, API externa, Sheet compartido). Es el caso más común y el que más se olvida.
- **Las herramientas de escritura validan sus argumentos contra la base de datos**, nunca confían en el número, precio o id que mandó el modelo. Ninguna guarda cubre esto.
- **El handoff no es solo apagar el bot.** `escalate_to_human` apaga el agente, marca la conversación en la plataforma del equipo (la tag de GHL, el label de Chatwoot) y avisa por mail. Si agregas un canal, implementa `handoff` con el mecanismo de etiquetas de esa plataforma — si solo apagás el bot, el cliente queda esperando a alguien que no sabe que lo espera.
- **Ningún aviso automático sin freno.** Todo lo que mande mails va por `notifications/`, que agrupa por clave y no repite dentro de 15 minutos. Un `sendAlert` por cada error convierte una caída de diez minutos en cientos de mails.
- **Ningún webhook sin autenticar.** Firma HMAC (cloud-api) o secreto compartido (el resto), incluso en los stubs: encolan trabajo y hacen correr el LLM.

## Tareas comunes
- **Cambiar la personalidad** → edita `config/agent.yaml` (nada de código).
- **Ajustar límites de gasto o guardas** → sección `guardrails` de `config/agent.yaml`.
- **Agregar una herramienta** → copia un archivo de `src/agent/tools/examples/`, define `sideEffect`, `returnsUntrustedContent`, `definition` y `run`, y súmalo a `defaultTools` en `src/agent/tools/index.ts`.
- **Agregar/terminar un canal** → implementa la interfaz de `src/channels/types.ts` y regístralo en `src/channels/registry.ts`. Los stubs `chatwoot/` y `ycloud/` son plantillas. **Empieza por la verificación del webhook.**
- **Probar** → `npm run demo` (web) o `npm run chat` (terminal). Ambos corren sin Postgres ni WhatsApp.
- **Verificar** → `npm run typecheck` y `npm test`.
- **Desplegar** → Coolify es donde vive el backend, y **desplegar va ANTES de conectar un canal**: el webhook necesita una URL pública. `npm run deploy` la primera vez, `npm run redeploy` después. No hay webhook de GitHub; el deploy se dispara contra la API de Coolify.

## Reglas
- **Escribe en español neutro**, no rioplatense: el repo lo leen alumnos de toda Latinoamérica. "puedes" y no "podés", "ejecuta" y no "corré", "aquí" y no "acá". Ojo con los regex de `guards/input.ts`: ahí conviven las dos variantes a propósito, porque un atacante escribe en la que se le ocurra.
- No pongas secretos en el código ni los commitees; van en `.env` (gitignoreado).
- Mantén el estilo: identificadores en inglés, comentarios y docs en español.
- Los comentarios explican **por qué**, no qué. Este repo se lee para aprender: si arreglás un bug sutil, deja anotado cuál era.
- Antes de dar por hecho un cambio, ejecuta `npm run typecheck` y `npm test`.

---
> Source: [diegovasquez-ai/wa-agent-starter](https://github.com/diegovasquez-ai/wa-agent-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
