## condflow4

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**CondoFlow** — Sistema de Gestão de Condomínios. A monorepo with:
- `frontend/` — Next.js 16 (App Router, React 19, Tailwind CSS 4, Supabase SSR)
- `api/` — FastAPI backend (Python, Supabase service-role client)

Production URL: https://condominios-gamma.vercel.app  (projeto Vercel: `condominios`)

> Nota: existe um projeto Vercel antigo `frontend` (frontend-theta-liard-32.vercel.app) que NÃO é mais usado e não recebe deploys. O oficial é `condominios-gamma.vercel.app`.

---

## Commands

### Frontend (Next.js)

```bash
cd frontend
npm run dev       # Start dev server (proxies /api/* → http://127.0.0.1:8001)
npm run build     # Production build (use to verify before deploy)
npm run lint      # ESLint check
```

**Deploy to Vercel** — the project uses CLI deploy, NOT GitHub auto-deploy:
```bash
cd frontend
npx vercel --prod --yes
```
`git push` alone does NOT trigger Vercel. Always use the CLI command above.

### API (FastAPI)

```bash
cd api
uvicorn index:app --reload --port 8001   # Local dev (requires .env with Supabase creds)
```

---

## Antes de escrever no banco: leia `docs/ESQUEMA-BANCO.md`

Colunas reais de `condominios`, as duas **colunas fantasma** que o código cita mas
não existem (`assistente`, `fluxo`), e três armadilhas que já custaram horas:
escrita do `supabase-js` que falha **calada**, `gerente_id` com dois significados
(profile × `gerentes.id`), e `edicoes_mensais` que precisa ser lida por
condomínio **+ mês**. O `applied.txt` está desatualizado — não use como fonte.

---

## Architecture

### Request Flow

```
Browser → Next.js (frontend/)
         ├── /api/* (dev) → rewritten → FastAPI at localhost:8001
         ├── Supabase (direct from client) → emissoes_pacotes, emissoes_ocorrencias, emissoes_arquivos
         └── Supabase Auth (middleware cookie session)
```

In **production on Vercel**, the Next.js app needs `NEXT_PUBLIC_API_URL` pointing to a running FastAPI instance. Without it, `/api/dashboard` and similar SWR calls will fail with "Erro de Conexão". Pages that query Supabase directly (Central de Emissões, emissão metrics) work independently of FastAPI.

### Two Parallel Status Flows

There are two separate data flows with distinct status values — confusing them is a common bug:

| Flow | Table | Status values |
|------|-------|---------------|
| **Arrecadações** (semestral process) | `processos` | `Em edição`, `Enviado`, `Em aprovação`, `Aprovado`, `Solicitar alteração`, `Emitido` |
| **Emissões** (emission packages) | `emissoes_pacotes` | `rascunho`, `pendente_gerente`, `pendente_sup_gerentes`, `pendente_sup_contabilidade`, `aprovado`, `registrado` |

`StatusBadge` (`src/components/StatusBadge.js`) maps both sets (lowercase key lookup). When adding a new status value, add it there.

### Authentication

- **Backend**: `get_current_user()` dependency in `api_routes.py` validates `Authorization: Bearer <JWT>` by calling `supabase.auth.get_user(token)`, then fetches the user's `profiles` row.
- **Frontend**: `AuthProvider` in `src/lib/auth.js` holds session state via `onAuthStateChange`. Every API call in `src/lib/api.js` auto-attaches the token via `apiFetcher`/`apiPost`.
- **Middleware**: `src/utils/supabase/middleware.js` runs on every request, refreshes the session cookie, and redirects unauthenticated users to `/login`.

### Supabase Client Pattern

Always use the factory — do not store a module-level singleton in components:

```js
import { createClient } from '@/utils/supabase/client';
// Inside the component function:
const supabase = createClient();
```

The server-side client is at `@/utils/supabase/server` (uses cookies, for SSR pages).

### RBAC

Roles: `master`, `gerente`, `assistente`, `supervisora`, `supervisora_contabilidade`, `supervisor_gerentes`, `departamento`, `sindico`, `outros`.

- Route-level guards: `ROUTE_ACCESS` map in `src/lib/roles.js` + `RouteGuard` component.
- Feature-level checks: `can(userRole, capability)` from `src/lib/roles.js`.
- `master` always bypasses all guards.

### SWR + Supabase Realtime

Dashboard and Central de Emissões mix two data strategies:
- **SWR** (`useSWR('/api/...', apiFetcher)`) — fetches from FastAPI
- **Direct Supabase** — `emissoes_pacotes`, `emissoes_ocorrencias`, `emissoes_arquivos`, storage bucket `emissoes`

Realtime subscriptions use `supabase.channel(...).on('postgres_changes', ...)` and are cleaned up in `useEffect` return.

---

## Environment Variables

### Frontend (`frontend/.env.local`)

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
NEXT_PUBLIC_API_URL=http://127.0.0.1:8001   # omit in production (Vercel rewrites handle dev)
```

### API (`api/.env` or root `.env`)

```
SUPABASE_URL=
SUPABASE_KEY=            # anon key
SUPABASE_SERVICE_KEY=    # service role key (bypasses RLS)
SECRET_KEY=              # FastAPI session secret
```

Vercel production env vars (set via dashboard or `vercel env add`): `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `SECRET_KEY`. The `NEXT_PUBLIC_API_URL` must also be set in Vercel if the FastAPI backend is hosted separately.

---

## Key Files

| File | Purpose |
|------|---------|
| `frontend/src/lib/api.js` | All FastAPI calls; injects auth headers automatically |
| `frontend/src/lib/auth.js` | `AuthProvider` + `useAuth()` hook |
| `frontend/src/lib/roles.js` | RBAC maps (`ROUTE_ACCESS`, `CAPABILITIES`, `can()`) |
| `frontend/src/components/StatusBadge.js` | Renders colored badges for both status flows |
| `frontend/src/components/AppShell.js` | App shell with sidebar, title map, route guard |
| `frontend/src/utils/supabase/middleware.js` | Session refresh + auth redirect on every request |
| `api/index.py` | FastAPI app entry point; mounts router at `/api` |
| `api/api_routes.py` | All FastAPI endpoints; `get_current_user()` dependency |
| `supabase/migrations/*.sql` | Versioned Supabase migrations (apply in numeric order via SQL Editor; see `applied.txt`) |
| `api/auth_constants.py` | RBAC source of truth — mirrors `frontend/src/lib/roles.js` |

---
> Source: [Delucka/CONDFLOW4](https://github.com/Delucka/CONDFLOW4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
