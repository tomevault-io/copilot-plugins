## noglem

> Instrucoes universais para qualquer agente de IA trabalhando neste projeto.

# AGENTS.md — JulIA Engineering Platform

Instrucoes universais para qualquer agente de IA trabalhando neste projeto.
Valido para Claude, GPT, Gemini, Codex ou qualquer outro LLM.

---

## O que e este projeto

**JulIA** (Julia + IA) e uma plataforma centralizada de ferramentas de engenharia com IA.
Oferece: traducao de documentos, extracao de tabelas PDF, conversao PDF→Word, RAG (base de conhecimento), pareceres tecnicos e extracao de instrumentos de P&ID.

**Tom obrigatorio:** tecnico, direto, preciso. Nunca marketing. Nunca linguagem casual ou infantilizada.

---

## Arquitetura

```
Browser
  └── Next.js 16 App Router (frontend + API Routes como proxy seguro)
        ├── Backend Central (FastAPI, porta 8000)   ← ferramentas simples
        ├── PATEC Microservice (FastAPI + PostgreSQL + Celery + Redis)
        └── Conhecimento/RAG Microservice (FastAPI + PostgreSQL/pgvector + FlashRank)
```

### Regra de seguranca — nunca violar

Toda requisicao do Next.js para os backends deve incluir:
- `X-Internal-API-Key` — chave compartilhada validada no backend
- `X-User-Id` — Clerk User ID do usuario autenticado

O helper `buildBackendAuthHeaders()` em `src/lib/backend.ts` monta essas headers.
**Nunca chame os backends diretamente do browser. Sempre via API Route do Next.js.**

Rotas protegidas: todo `/dashboard/*` requer autenticacao Clerk (middleware.ts).
Rotas publicas: `/`, `/sign-in`, `/sign-up`, webhooks.

---

## Estrutura de Diretorios

```
enghub-v2/
├── src/
│   ├── app/
│   │   ├── (auth)/                  # Paginas de autenticacao Clerk
│   │   ├── dashboard/               # Uma pasta por ferramenta
│   │   │   ├── translate/
│   │   │   ├── pdf-extractor/
│   │   │   ├── pdf-converter/
│   │   │   ├── pid-extractor/
│   │   │   ├── rag/
│   │   │   └── parecer-tecnico/
│   │   ├── api/                     # API Routes (proxies seguros para os backends)
│   │   └── globals.css              # Tokens CSS do design system
│   ├── components/
│   │   ├── ui/                      # Design system: Button, Card, Badge, Input, Logo, Skeleton
│   │   ├── layout/                  # Shell, Header, Sidebar
│   │   └── parecer-tecnico/         # Componentes especificos do PATEC
│   ├── lib/
│   │   ├── backend.ts               # buildBackendAuthHeaders()
│   │   ├── tools-registry.ts        # Registro central de todas as ferramentas
│   │   └── utils.ts                 # cn() e helpers gerais
│   └── design-system/               # Tokens e referencia visual
├── backend/                         # FastAPI central
│   └── app/
│       ├── routers/                 # translate, pdf, pid, email
│       └── services/                # gemini, pdf_extract, pdf_convert, pid_extract
├── services/
│   ├── patec-backend/               # Microservico pareceres tecnicos
│   └── conhecimento-backend/        # Microservico RAG/conhecimento
├── AGENTS.md                        # Este arquivo — instrucoes universais
└── CLAUDE.md                        # Instrucoes especificas para Claude Code
```

---

## Tools Registry

`src/lib/tools-registry.ts` e a **fonte unica de verdade** para todas as ferramentas.
Usado pela sidebar, dashboard grid e status badges. Qualquer alteracao de metadados de ferramenta passa por aqui.

Campos obrigatorios: `id`, `name`, `description`, `icon`, `href`, `category`, `status`
- `status`: `'live'` | `'beta'` | `'coming_soon'`
- `category`: `'documentacao'` | `'conhecimento'` | `'analise'` | `'instrumentacao'`

---

## Como Alterar uma Ferramenta Existente

**Regra zero: leia os arquivos envolvidos antes de qualquer mudanca.**

### 1. So UI (layout, texto, estilo)
- Edite `src/app/dashboard/<tool>/page.tsx` e componentes em `src/components/<tool>/`
- Siga o design system — tokens CSS, tipografia, border radius
- Nao toque API Route nem backend

### 2. Logica de frontend (estado, interacao)
- Edite a pagina ou componentes filhos
- Verifique se o endpoint existente ja retorna os dados necessarios antes de criar um novo
- Prefira adicionar parametros opcionais ao endpoint existente

### 3. API Route (proxy Next.js)
- Arquivo: `src/app/api/<tool>/route.ts`
- Nunca remova a validacao de auth (`X-Internal-API-Key`, `X-User-Id`)
- Normalize erros — nunca exponha stack traces do backend ao browser
- Se mudar o contrato (body, params), atualize frontend e backend juntos

### 4. Backend Central
- Router: `backend/app/routers/<tool>.py`
- Service: `backend/app/services/<tool>_service.py`
- Mantenha `require_internal_api_key` em todos os endpoints
- Se mudar schema de resposta, atualize os tipos TypeScript no frontend

### 5. PATEC Microservice
- Endpoints: `services/patec-backend/app/api/v1/endpoints/`
- Mudancas no banco: sempre crie migration via `alembic revision --autogenerate`
- Nunca altere o banco diretamente

### 6. RAG Microservice
- Services: `services/rag-backend/app/services/`
- Mudancas no modelo de embedding afetam documentos ja indexados — documente o impacto

### 7. Metadados de ferramenta
- Edite apenas `src/lib/tools-registry.ts`

### Regras gerais
- Altere apenas o que foi pedido. Nao refatore codigo adjacente.
- Nao adicione tratamento de erro para cenarios que nao podem ocorrer.
- Nao crie helpers ou abstracoes para uso unico.
- Se a mudanca afeta contrato de API, atualize ambos os lados na mesma alteracao.
- **DEPLOY / GITHUB:** Toda alteracao finalizada deve sempre ser comitada (`git commit`) e subida para o GitHub (`git push origin main`), para garantir que reflita no site.

### Versionamento do site

A versao do site e exibida no sidebar em `src/components/layout/sidebar.tsx`.
**REGRA OBRIGATORIA:** A versao DEVE ser atualizada a cada deploy.

Formato: `VX.Y.Z`

| Segmento | Quando incrementar |
|----------|-------------------|
| **X** (ex: v**2**.0.3) | Impacto muito grande: redesign de layout, mudancas brutas de arquitetura |
| **Y** (ex: v2.**0**.3) | Inclusao de nova ferramenta no dashboard |
| **Z** (ex: v2.0.**3**) | Atualizacoes menores: melhorias em ferramentas existentes, correcoes, ajustes |

---

## Como Adicionar uma Nova Ferramenta

1. Registre em `src/lib/tools-registry.ts`
2. Crie a API Route em `src/app/api/<tool>/route.ts` com validacao de auth
3. Crie a pagina em `src/app/dashboard/<tool>/page.tsx`
4. Adicione o endpoint no backend central ou crie um microservico
5. Nunca exponha os backends diretamente — sempre proxy via API Route

---

## Backends

### Backend Central (`backend/`)
- `POST /api/translate/` — Traducao com Google Gemini
- `POST /api/pdf/extract/` — Extracao de tabelas (pdfplumber → Excel)
- `POST /api/pdf/convert/` — PDF → DOCX (pdf2docx)
- `POST /api/pid/extract/` — Extracao de instrumentos P&ID (Tesseract OCR)

Rate limiting por usuario (sliding window): Translate 30/min, PDF/PID 5/min.

### PATEC (`services/patec-backend/`)
PostgreSQL + Redis + Celery. Para analises longas e pareceres tecnicos.
Proxy Next.js: `/api/parecer-tecnico/[...path]/route.ts`

### RAG / Conhecimento (`services/conhecimento-backend/`)
PostgreSQL (pgvector) + FlashRank reranker + Google Gemini embeddings.
Gerencia colecoes, upload de PDFs, embeddings vetoriais e chat com documentos.
Proxy Next.js: `/api/rag/[...path]/route.ts`

---

## Infraestrutura Railway

Todos os backends rodam no Railway. O diagrama abaixo mostra os servicos e suas conexoes:

```
Usuario → Frontend (Vercel) → Next.js API proxy
                                    |
                    ┌───────────────────────────────┐
                    │  conhecimento-api (RAG)        │ ← upload PDF, chat
                    │  patec-api (PATEC)             │ ← analise tecnica
                    └───────────┬───────────────────┘
                                |
                           Postgres (dados)
                           Redis (filas) → patec-worker (jobs async)
```

### Servicos

| Servico | Funcao | Conexoes |
|---------|--------|----------|
| **Postgres** | Banco de dados central. Armazena colecoes do RAG, documentos, vetores (pgvector), dados do PATEC. Volume persistente (`postgres-volume`). | Recebe conexoes de `conhecimento-api`, `patec-api` e `patec-worker` |
| **Redis** | Cache e broker de filas (Celery). Volume persistente (`redis-volume`). | Recebe conexoes de `patec-api` e `patec-worker` |
| **conhecimento-api** | Backend do modulo RAG/Conhecimento. Gerencia colecoes, recebe uploads de PDFs, gera embeddings vetoriais e responde perguntas via chat. | Conecta no Postgres |
| **patec-api** | Backend do modulo PATEC. Recebe requisicoes do frontend, processa e salva no Postgres. Enfileira tarefas assincronas no Redis. | Conecta no Postgres e Redis |
| **patec-worker** | Worker Celery do PATEC. Consome tarefas da fila Redis e executa processamento em background (analise de documentos, geracao de relatorios). | Conecta no Postgres e Redis |

### Deploy

- Cada servico tem seu proprio `Dockerfile` e `railway.toml` no diretorio raiz correspondente.
- O Railway faz deploy automatico a cada push em `main` no GitHub.
- Variaveis de ambiente sao configuradas no painel do Railway (nao no repositorio).
- Para verificar logs de producao: painel do Railway ou Vercel.

---

## Design System — v3 "instrumento de precisao"

Tokens CSS em `src/app/globals.css` via `@theme inline` (Tailwind v4).
Componentes em `src/components/ui/`. Referencia viva em `/dashboard/styleguide` (admin).
Identidade: dark-first (grafite frio), data-dense, mono como assinatura visual.

### Fontes

IBM Plex Sans (`font-sans` — interface e headings) + IBM Plex Mono (`font-mono` — dados,
tags, contadores, microlabels), servidas via `next/font` em `src/lib/fonts.ts`.
Nao existem mais `font-brand`/`font-heading`/`font-body`.

### Regras visuais — sem excecao

1. **Cores:** APENAS tokens CSS do `globals.css`. Nunca `bg-blue-500`, `text-green-500` etc.
   - Superficies: `bg-canvas` (fundo do app), `bg-surface-1` (paineis/cards), `bg-surface-2` (inputs/hover), `bg-surface-3` (popover/elevado), `bg-overlay`
   - Bordas: `border-edge`, `border-edge-strong`
   - Texto: `text-fg`, `text-fg-muted`, `text-fg-subtle`, `text-fg-disabled`, `text-fg-inverse`
   - Accent (azure #4BA4EE): `bg-accent`, `text-accent`, `bg-accent-subtle`, `text-accent-fg` (texto sobre accent), `hover:bg-accent-hover`
   - Semanticas: `success` / `warning` / `danger` / `info`, cada uma com `-subtle` (fundo) e `-text` (texto claro). Ex: `bg-danger-subtle text-danger`
2. **Accent:** apenas CTAs, links, estado ativo e indicadores criticos. `info` == accent por design.
3. **Numeros:** SEMPRE `font-mono tabular-nums`.
4. **Microlabels:** classe `microlabel` (mono, uppercase, tracking) para categorias, eyebrows e labels de dados.
5. **Border radius:** escala 2/3/4/6/8px (`rounded-xs` ate `rounded-xl`). NUNCA `rounded-full` em containers.
6. **Animacoes:** apenas `fade-in`, `fade-in-up`, `shimmer`. Nada pulsando sem acao do usuario.
7. **Feedback:** sucesso/erro transiente = `toast` (sonner); aviso persistente = `<Alert>`; acao destrutiva = `useConfirm()`. NUNCA `confirm()`/`alert()` nativos.
8. **Headers de pagina:** SEMPRE `<PageHeader tool="{id}">` — titulo/descricao/badge vem do `tools-registry.ts`.

### Componentes

- `Button` — variantes: default/primary, secondary/outline, ghost, danger/destructive, link | tamanhos: sm, md, lg, icon | props: loading, asChild
- `Input`, `Textarea` — props: label, error, hint
- `Card`, `CardHeader`, `CardContent`, `CardFooter`, `CardTitle`, `CardDescription`, `CardAction` — prop: interactive
- `Badge` — chip tecnico mono/uppercase; variantes: default, success, warning, error, info, secondary, outline | props: dot, asChild
- `PageHeader` — props: tool (id do registry) OU title/description; backHref, actions
- `Dropzone` — upload unico do DS; props: onFiles, accept, multiple, maxSizeMB, loading, error, hint, compact
- `Alert` — variantes: info, success, warning, danger | props: title, icon
- `EmptyState` — props: icon, title, description, action, size
- `Progress` — props: value, max, indeterminate, label
- `Spinner` / `LoadingBlock` — loading unico do DS (nunca Loader2 ou divs animate-spin)
- `Toaster`/`toast` (sonner) — montado no Shell; `toast.success/error/info/promise`
- `ConfirmProvider`/`useConfirm` — dialogo de confirmacao (substitui confirm() nativo)
- `Select`, `Tabs`, `Checkbox` — radix com tema do DS
- `Logo` — variantes: full ("Jul/IA"), compact ("J/"), tagline | tamanhos: sm, md, lg
- `Skeleton` — shimmer loader

---

## Stack

**Frontend:** Next.js 16 (App Router), React 19, TypeScript strict, Tailwind CSS v4, Clerk Auth, Radix UI, Lucide React

**Backends:** FastAPI, Pydantic v2, SQLite / PostgreSQL, Redis, Celery, LangChain, ChromaDB, Google Gemini

**Deploy:** Vercel (frontend) + Railway (backends via Docker)

---

## Variaveis de Ambiente

### Frontend (`.env.local`)
```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_API_URL=http://localhost:8000
INTERNAL_API_KEY=
PATEC_API_URL=http://localhost:8001
RAG_API_URL=http://localhost:8002
```

### Backend Central (`.env`)
```
GOOGLE_API_KEY=
INTERNAL_API_KEY=
UPLOAD_DIR=/tmp/julia-uploads
MICROSOFT_CLIENT_ID=
MICROSOFT_CLIENT_SECRET=
MICROSOFT_TENANT_ID=
```

### PATEC (`.env`)
```
DATABASE_URL=postgresql+asyncpg://...
REDIS_URL=redis://...
INTERNAL_API_KEY=
```

---

## Convencoes de Codigo

- TypeScript strict. Sem `any` implicito.
- Server components por padrao. `'use client'` apenas quando necessario.
- Fetch nas API Routes: sempre valide auth antes de proxiar.
- Erros: retorne o status code original, normalize a mensagem. Nunca exponha stack traces.
- Sem `console.log` em producao. `console.error` apenas para erros reais.
- Nao crie arquivos de documentacao sem instrucao explicita.

---
> Source: [alelemos02/Noglem](https://github.com/alelemos02/Noglem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
