## landa-agent-service

> Microservicio de LANDA Tech: agente de WhatsApp para DPG Seguros. Q&A inbound de pólizas (saldo, estado, coberturas) + flujo de validación de pago con escalación humana vía Chatwoot. Repo aparte de `lambda-proyect` (que tiene el agente de voz).

# CLAUDE.md — landa-agent-service

Microservicio de LANDA Tech: agente de WhatsApp para DPG Seguros. Q&A inbound de pólizas (saldo, estado, coberturas) + flujo de validación de pago con escalación humana vía Chatwoot. Repo aparte de `lambda-proyect` (que tiene el agente de voz).

Para contexto profundo lee `.planning/PROJECT.md` y `.planning/ROADMAP.md`. Este archivo es la briefing rápida.

---

## Quick start

```bash
# Setup
cp .env.example .env  # Llenar credenciales antes de correr
uv sync --frozen

# Dev local
uv run uvicorn app.main:app --reload

# Tests
uv run pytest

# Lint
uv run ruff check . && uv run black --check .
```

Variables de entorno críticas (ver `.env.example` para la lista completa):
- `OPENROUTER_API_KEY` — gateway de LLMs
- `LLM_MODEL_CONVERSATION`, `LLM_MODEL_JUDGE` — modelos por rol (cambiables sin redeploy)
- `WA_TOKEN`, `WA_PHONE_ID` — Meta Cloud API (WhatsApp Business)
- `WA_WEBHOOK_SECRET` — validación HMAC `X-Hub-Signature-256`
- `SOFTSEGUROS_BASE_URL` — `https://app.softseguros.com/`
- `CHATWOOT_URL`, `CHATWOOT_API_KEY` — inbox self-hosted
- `POSTGRES_URL`, `REDIS_URL` — stores principales
- `LANGSMITH_API_KEY`, `LANGSMITH_PROJECT` — tracing
- `LAMBDA_PROYECT_BASE_URL`, `LAMBDA_PROYECT_INTERNAL_TOKEN` — integración con voice agent

---

## Arquitectura: Vertical Slice (feature-based)

**No es Hexagonal**. Vertical slice fue la elección consciente — hex es over-engineering para v1 con un cliente.

```
app/
├── features/                # Cada feature de cara al usuario
│   ├── qa/                  # Q&A inbound: graph, nodes, tools, prompts
│   ├── payment/             # Validación de pago + forward a cartera
│   ├── escalation/          # Escalación a Chatwoot, manejo de respuestas humanas
│   └── handoff/             # Recibe handoff de lambda-proyect (voice → WhatsApp)
├── integrations/            # Clientes externos — clases planas, SIN ABCs
│   ├── softseguros.py       # httpx + tenacity retry + pybreaker + caché Redis
│   ├── chatwoot.py          # API client (create conversation, post message)
│   ├── meta_cloud.py        # Meta Graph API v18.0 (send/receive)
│   ├── openrouter.py        # Factory get_llm(role) → ChatOpenAI con base_url OpenRouter
│   └── lambda_proyect.py    # REST client a lambda (update_debtor, escalate)
├── security/                # Cross-cutting — Chain of Responsibility
│   ├── prompt_firewall.py
│   ├── input_sanitizer.py
│   ├── judge.py             # LLM-as-judge sobre cada salida
│   ├── output_firewall.py
│   ├── hmac_validator.py
│   └── audit_log.py         # Append-only Postgres + hash chain + S3 sink
├── memory/                  # L3 cases + L4 debtor flags
│   ├── case_store.py        # db.cases (cross-canal)
│   └── debtor_flags.py      # flags resumidos del deudor
├── models/                  # Pydantic compartidos (Conversation, Case, Policy, Debtor)
├── webhooks/                # FastAPI handlers: meta.py, chatwoot.py
├── config/                  # settings.py, llm.py, tenants.py
└── main.py
```

**Regla**: cuando aparezca una feature nueva, va en `features/<nombre>/`. Cuando aparezca una integración nueva, va en `integrations/<nombre>.py`. No mover cosas a carpetas "técnicas" (services/, controllers/) — eso es n-tier, no vertical slice.

---

## Stack (locked-in, no re-debatir sin razón fuerte)

| Capa | Tecnología |
|---|---|
| Runtime | Python 3.12 + FastAPI |
| Orquestación agente | LangGraph + Postgres checkpointer |
| Gateway LLM | **OpenRouter** (NO Anthropic SDK directo) |
| Default conversation model | `google/gemini-2.5-pro` (cambiable por env var) |
| Default judge model | `google/gemini-2.5-flash` (temp=0) |
| WhatsApp | **Meta Cloud API directo** (NO Twilio) |
| Inbox humanos | Chatwoot self-hosted en Railway, docker-compose |
| DB | Postgres (checkpoints + audit log + cases) |
| Cache + Queue | Redis + ARQ |
| Observability LLM | LangSmith free tier |
| Audit log compliance | Custom append-only Postgres + hash chain + S3 sink |
| Errors | Sentry |
| Deploy | Railway |

---

## Reglas críticas (do / don't)

### Do

- **Llama LLMs solo vía `get_llm(role)`** del factory en `app/config/llm.py`. Nunca instancies `ChatOpenAI` directo en código de feature
- **Usa Pydantic v2 para todo I/O** — tools, webhooks, configs, mensajes entre módulos
- **Lock `poliza_id` en el state del grafo** — el LLM nunca puede cambiar de póliza mid-conversación. El tool recibe `poliza_id` desde el state, no de la generación
- **Cada tool tiene allowlist de operaciones por estado del grafo** — no se puede `confirm_payment` antes de tener aprobación de cartera
- **Sanitiza tool outputs antes de devolverlos al LLM** — limpia patterns tipo `"system:"`, `"instruction:"`, solo campos en allowlist llegan al modelo
- **Audita cada acción crítica** — turn LLM, tool call, decisión del judge, mensaje saliente, escalación → al `audit_log` con hash chain
- **Cachea consultas SoftSeguros en Redis con TTL 60s** — clave `(poliza_id, query_type)`
- **Circuit breaker en SoftSeguros**: tras N fallos consecutivos, el bot escala a humano. **Nunca devolver data stale.**
- **Verifica HMAC `X-Hub-Signature-256` en CADA webhook entrante** de Meta
- **Idempotencia por `message_id`** — Meta puede reentregar webhooks

### Don't

- ❌ **No hardcodear modelos LLM** en código — siempre `get_llm(role)`
- ❌ **No persistir PII de pólizas** (saldos, datos del cliente) en LANDA — todo on-demand desde SoftSeguros. Solo metadata + hashes en audit log
- ❌ **No pasar comprobantes (imágenes/PDFs) por un LLM con visión** — van directo a cartera. Vector de inyección por imagen es real
- ❌ **No usar el SDK de Anthropic directo, ni el de OpenAI directo** — toda llamada a LLM pasa por OpenRouter
- ❌ **No agregar Twilio para WhatsApp** — descartado (Meta Cloud API directo). Twilio existe en lambda-proyect para otros casos de uso, no acá
- ❌ **No crear ABCs/Ports prematuros** — usa clases concretas. Solo extrae ABC cuando exista una segunda implementación real (segundo cliente o segundo provider)
- ❌ **No commitear** `venv/`, `__pycache__/`, `.env`, credenciales — está en `.gitignore`
- ❌ **No generar el mensaje "pago confirmado" desde el LLM libremente** — solo puede aparecer en el path post-aprobación de cartera, con marca de procedencia verificada por `output_firewall`
- ❌ **No exponer al LLM tools de tipo `list_all_*` o `search_*`** — todas las queries están scopeadas a la póliza activa de la conversación
- ❌ **No agregar métodos write (POST/PUT/PATCH/DELETE) en `SoftSegurosClient`**. El bot es READ-ONLY contra SoftSeguros por diseño. Adding write methods requires:
  1. ADR documentado en `.planning/adr/`
  2. Threat model actualizado en PROJECT.md §"Seguridad"
  3. PROJECT.md scope explícitamente actualizado para incluir el write
  4. Operator approval explícito

  CI guard `tests/test_softseguros_readonly.py` falla el build automáticamente si aparecen verbos prohibidos (`post`/`put`/`patch`/`delete`/`create_`/`update_`/`set_`/`modify_`) en method names de `SoftSegurosClient`. Excepción: top-level `_get_token` y `_refresh_token_on_401` POSTean a `/api-token-auth/` (auth bootstrap, no escritura de datos del cliente) — esto está documentado en el módulo docstring READ-ONLY INVARIANT.

---

## Seguridad: 13 capas de defensa en profundidad

Estas no son opcionales. Cada una es un requirement testeable. Ver `.planning/PROJECT.md` para detalle. Resumen:

1. Prompt firewall de entrada (sanitización + patterns conocidos)
2. Conversation-locked póliza (en state, no en LLM)
3. Tool boundaries en código (allowlist por estado del grafo)
4. Tool output sanitization
5. LLM-as-judge sobre cada mensaje saliente
6. Output firewall determinístico (patterns hardcoded prohibidos)
7. HMAC `X-Hub-Signature-256` en webhooks
8. Allowlist de números autorizados como cartera
9. Idempotencia por `message_id`
10. Egress controls (solo SoftSeguros + Meta + Chatwoot + OpenRouter + LangSmith)
11. Audit log inmutable (append-only + hash chain + S3 sink)
12. Rate limiting multi-nivel
13. Comprobantes nunca por LLM visión

**Defensa en profundidad**: el LLM no es la única línea de defensa. Las restricciones críticas (scope por póliza, no list-all, no autoconfirmación de pagos) viven en código, no en system prompt.

---

## Memoria multi-capa

| Capa | Qué guarda | Storage |
|---|---|---|
| L1 | State del turno actual del grafo | LangGraph Postgres checkpointer |
| L2 | History de mensajes de la conversación | LangGraph `messages` state + Chatwoot |
| L3 | Eventos del caso cross-canal | `db.cases` (nueva collection, key=`case_id`) |
| L4 | Flags del deudor cross-caso | `db.debtors.historial_whatsapp[]` + flags resumidos (`ultima_llamada_fecha`, `promesa_de_pago`, `escalado_previo`, `intentos`) |
| L5 | Knowledge base estático de empresa (~4 pgs DPG) | Markdown en `knowledge/dpg_cartera.md`, inyectado en system prompt envuelto en delimitadores `== REFERENCIA ==`. Audit pipeline en `security/kb_auditor.py`. Vector RAG real diferido hasta que el KB crezca >20 pgs |

L4 obligatorio en v1: el bot carga flags resumidos en system prompt antes de responder. **NO transcripts crudos** (saturan prompt, vector de injection).

---

## Integración con lambda-proyect (voice agent)

`lambda-proyect/backend/cobranza/` tiene el agente de voz. La integración entre ambos:

- **`case_id` (UUID v4)**: voice lo crea al iniciar la llamada y lo pasa en el handoff. Si el cliente escribe sin llamada previa, WhatsApp agent lo crea él mismo
- **`POST /case/handoff`** (nuestro endpoint, lo construimos en F5): recibe payload completo de lambda
  ```
  {case_id, debtor_id, poliza_number, call_id, user_id, phone, initial_context, message}
  ```
- **REST a lambda** para mutaciones del deudor: `POST /cobranza/case/{case_id}/escalate`, `POST /cobranza/debtor/{debtor_id}/update`
- **`landa-shared` (git submodule)**: paquete compartido entre los dos repos para `SoftSegurosAdapter`, modelos Pydantic (Debtor, Policy, ConversationContext), helpers de tenant isolation y descifrado de credenciales

**Stub muerto identificado en lambda**: `cobranza/sub_agents/whatsapp_notifier.py` encola `send_whatsapp_job` que NO está registrado en `worker.py`. Lo reemplazamos por un `POST /case/handoff` a este servicio (parte de F5).

**3 senders WhatsApp existentes en lambda — NO reutilizar**:
- `whatsapp_agent.py` (Twilio, prospecting B2B) — otro caso de uso
- `wa_handler.py` (Twilio inbound) — otro caso de uso
- `services/notifications.py` (Twilio) — notificaciones internas
- `whatsapp_sender.py` (Meta Graph) — patrón a replicar/mejorar dentro de este repo

---

## Cliente: DPG Seguros (single-tenant en v1)

- Credenciales SoftSeguros: encriptadas en `db.tenant_configs`. Helper de descifrado compartido con lambda-proyect
- Número WhatsApp Business: `+16415416615` (LandaTech), Meta Cloud API directo
- Identificación del cliente: por número de póliza (no cédula). UX: si fricciona en prod, evaluar fallback a `/api/cliente/listar_cliente_por_documento/`
- Endpoints SoftSeguros que consumimos: `/api/poliza/`, `/api/cliente/`, `/api/estadopoliza/`, `/api/pagopoliza/`

Multi-tenant arquitectónico está pensado (config por cliente, factory por tenant) pero **operacionalmente single-tenant v1**. Cliente #2 entra en milestone futuro.

---

## Out of scope (no construir en v1)

- ❌ Vector RAG con embeddings + pgvector + retrieval pipeline — diferido hasta que KB crezca >20 páginas. v1 usa inyección directa al system prompt (cabe sobrado en context window)
- ❌ Validación automática / OCR del comprobante — humano siempre
- ❌ Multi-tenant operativo con otros clientes — DPG solamente
- ❌ Dashboard nuevo LANDA para revisión de comprobantes — chat de cartera ya existente sirve
- ❌ Construcción del bot de voz — ya existe en lambda-proyect
- ❌ Cambios estructurales en lambda-proyect — solo definimos contratos REST, ellos implementan su lado
- ❌ Chatwoot SaaS — descartado, self-hosted
- ❌ Twilio para WhatsApp — descartado, Meta Cloud API directo
- ❌ Anthropic SDK directo, OpenAI SDK directo — todo por OpenRouter

---

## Convenciones de código

- **Async por default**: FastAPI + httpx async + asyncpg + arq. Nada bloqueante
- **structlog para logs**: JSON estructurado, PII redactada por default
- **Pydantic v2 settings**: `BaseSettings` con `env_prefix` por dominio (`LLM_`, `WA_`, `SOFTSEGUROS_`)
- **Type hints estrictos**: `mypy --strict` en CI
- **Tests**: pytest + pytest-asyncio. Cada feature tiene su carpeta `tests/` adentro
- **Sin docstrings de planeación** en código: el contexto vive en `.planning/`. En código, solo comentarios cuando el "por qué" no es obvio
- **No emojis en código ni mensajes generados** (a menos que sean parte de un mensaje al cliente final que requiera tono cálido)
- **Commits estilo conventional**: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`. Subject ≤72 chars

---

## Cómo navegar este repo

| Si quieres... | Ve a... |
|---|---|
| Entender el alcance v1 | `.planning/PROJECT.md` |
| Ver el plan por fases | `.planning/ROADMAP.md` |
| Cambiar un modelo LLM | `app/config/llm.py` + `.env` (`LLM_MODEL_*`) |
| Agregar una feature nueva | `app/features/<nombre>/` |
| Agregar una integración nueva | `app/integrations/<nombre>.py` |
| Agregar una capa de seguridad | `app/security/<nombre>.py` |
| Ver decisiones arquitectónicas | `.planning/PROJECT.md` — Key Decisions |
| Ver convenciones GSD | `.planning/config.json` |

---

*Última actualización: 2026-06-27 — initial CLAUDE.md tras cierre de discuss + plan inicial. Actualizar después de cada fase.*

---
> Source: [mapube16/landa-agent-service](https://github.com/mapube16/landa-agent-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
