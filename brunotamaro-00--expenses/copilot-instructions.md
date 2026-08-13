## expenses

> **Expenses**: tracker **personal** (AR) de **gastos/ingresos**. Lo usan **2 personas con cuentas separadas y aisladas** (cada una con su login web y su número de WhatsApp vinculado; **sin** vista compartida ni comparación entre cuentas — no proponer features "de pareja"). Entrada por **WhatsApp** (Meta Cloud) + dashboard web. Stack: **FastAPI (Py 3.11)** + **Postgres** + **OpenAI** (solo parse de gastos) + **React 19 + Vite**.

# CLAUDE.md (contexto mínimo para agentes)

**Expenses**: tracker **personal** (AR) de **gastos/ingresos**. Lo usan **2 personas con cuentas separadas y aisladas** (cada una con su login web y su número de WhatsApp vinculado; **sin** vista compartida ni comparación entre cuentas — no proponer features "de pareja"). Entrada por **WhatsApp** (Meta Cloud) + dashboard web. Stack: **FastAPI (Py 3.11)** + **Postgres** + **OpenAI** (solo parse de gastos) + **React 19 + Vite**.

El bot es de **captura pura**: registra/edita/borra movimientos y crea recurrentes. No hay assistant de lenguaje natural, ni help/menús, ni consultas por chat (los números se ven en el dashboard). Auto-registra sin confirmación salvo: categoría genuinamente ambigua (botones con candidatas), tarjeta de crédito (flujo de imputación) o borrado (irreversible).

Estructura:
- `backend/` API + auth + DB + LLM parse + `app/bot/` (dispatcher del bot)
- `wpp-bot/` webhook WhatsApp, proxy fino; llama al backend
- `frontend/` dashboard
- `docker-compose.yml` postgres/redis/api/wpp-bot/frontend/backup

## Arranque (Docker end-to-end)

```bash
cp .env.example .env
docker compose up -d --build
```

Puertos (host): Postgres `5432`, API `8000` (`GET /health`), bot `8001` (webhook `/webhooks/whatsapp`), frontend `3000`.

## Dev local (iteración rápida)

```bash
cp .env.example .env
docker compose up -d postgres

cd backend
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
alembic upgrade head
uvicorn app.main:app --reload --port 8000

cd ../frontend
npm install
npm run dev # http://localhost:5173 (Vite proxya /api -> :8000)
```

Bot (solo con credenciales Meta):

```bash
cd wpp-bot
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
python -m wpp_bot.main
```

## ENV (ver `.env.example`)

Mínimas:
- DB/LLM: `DATABASE_URL`, `OPENAI_API_KEY` (`OPENAI_TIMEOUT_SECONDS`, `EMBEDDING_MODEL` opcionales)
- Backend auth: `SECRET_KEY`, `BOT_API_KEY`, `JWT_EXPIRE_DAYS`, `CORS_ORIGINS`, `AUTH_USERS` (seed del usuario web)
- Bot: `API_URL` (URL del backend); Redis opcional para dedupe de webhooks (`REDIS_URL`)
- Web: `APP_URL` (CTA "Ver en Dashboard") y `VITE_API_URL` (vacío en dev)

WhatsApp Cloud (solo bot real): `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_VERIFY_TOKEN` (+ `WHATSAPP_APP_SECRET` recomendado), `WHATSAPP_GRAPH_VERSION`.

## Arquitectura + auth

```text
WhatsApp -> webhook -> wpp-bot (verify HMAC, dedupe wamid Redis, lock por chat)
wpp-bot -> POST /api/v1/assistant/whatsapp/dispatch (Bearer BOT_API_KEY + X-WhatsApp-WA-ID)
backend app/bot/dispatcher.py (ruteo determinista, sin FSM):
  interactive -> tarjeta esperando fecha -> sí/no -> editar/borrar -> recurrente -> consulta (CTA dashboard) -> captura
captura: 1 llamada LLM (parser) + 1 embedding (cache LRU) -> gating por gap/top1 -> auto-registro

Browser -> frontend -> backend (Authorization: Bearer <JWT>)
```

- Web: `POST /api/v1/auth/login` (form) → JWT; se guarda en `localStorage.auth_token`. Usuario se crea con `AUTH_USERS`.
- Bot: no usa JWT; backend resuelve usuario por `whatsapp_wa_id`. Vinculación: `AUTH_USERS=user:pass:wa_id`. wa_ids desconocidos se rechazan salvo `WHATSAPP_AUTO_REGISTER` (default: solo dev auto-registra `wa_<hex>`).
- Errores: nunca genéricos — el bot responde `⚠️ {Tipo}: {mensaje}` (un solo try/except en el borde del dispatcher).
- Botones: ids planos con token opaco del pending (`confirm:`, `cancel:`, `catpick:{t}|{cat}`, `cc_keep:`, `cc_future:`, `cc_cancel:`, `rec_today:`, `rec_schedule:`). Sin firma HMAC.

## Endpoints clave (backend)

- Salud: `GET /health`
- Movimientos: `POST /api/v1/expenses/parse` (no persiste), `POST /api/v1/expenses`, `GET /api/v1/expenses`, `PATCH/DELETE /api/v1/expenses/{id}`
- Dashboard: `GET /api/v1/expenses/{summary|timeseries|composition}`
- Catálogo: `GET /api/v1/categories`
- Bot: `POST /api/v1/assistant/whatsapp/dispatch` + `/api/v1/bot/pending-confirmations*`, `/api/v1/bot/pending-actions*`

## Gotchas

- Dev frontend: `frontend/vite.config.ts` proxya `/api` a `http://localhost:8000`.
- Docker/prod: `frontend/nginx.conf` **NO** proxya `/api` (solo SPA fallback): usar `VITE_API_URL` + CORS, o agregar proxy en nginx.
- JSON: montos como **string** por `Decimal` (`frontend/src/types/index.ts`).
- Gating de categoría: única fuente de verdad `category_gate()` en `backend/app/llm/parser.py` (umbrales `category_clarification_*`). Calibrar con `backend/tools/eval_categories.py` antes de tocarlos.
- Interpretabilidad: cada asignación emite un log JSON `category_assignment_decision` (outcome, top1, gap, componentes por candidato).

## Dónde tocar (atajos)

- Categorías: `backend/app/categories/catalog.py` + seed startup (`backend/app/main.py` → `backend/app/categories/seed.py`)
- LLM parse: `backend/app/llm/{prompts.py,parser.py,postprocess.py}` + config `backend/app/config.py`
- Bot (ruteo/copy/flows): `backend/app/bot/{dispatcher,capture,interactive,manage,recurring,render,state}.py`
- Proxy WhatsApp: `wpp-bot/wpp_bot/{main.py,meta_client.py}`
- Dashboard: `frontend/src/{api,components,pages}/`

## Tests

```bash
cd backend && pytest -v
cd ../wpp-bot && pytest -v
cd ../frontend && npm run build && npm run lint
```

---
> Source: [brunotamaro-00/expenses](https://github.com/brunotamaro-00/expenses) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
