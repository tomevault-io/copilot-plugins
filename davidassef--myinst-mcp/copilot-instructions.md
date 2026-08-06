## myinst-mcp

> > Este arquivo é destinado a agentes de código. Leia-o antes de modificar qualquer parte do projeto.

# AGENTS.md — MyInst

> Este arquivo é destinado a agentes de código. Leia-o antes de modificar qualquer parte do projeto.
> Idioma do projeto: **português (pt-BR)** para documentação, comentários e nomes de negócio; inglês para APIs/frameworks.

---

## Visão geral do projeto

O **MyInst** é um vault open source para armazenar, versionar e sincronizar contexto agentic entre projetos, workspaces, dispositivos e clientes MCP.

Ele centraliza `skills`, `instructions`, `agents`, `hooks`, `memory`, `snippets` e configurações de clientes em um backend próprio, com interface web, API, CLI e um MCP server local.

Componentes principais:

| Componente | Pacote / Pasta | Papel |
|------------|----------------|-------|
| Frontend | `frontend/` | Painel web (React + Vite) para gerenciar workspaces, projetos, conteúdo e API keys |
| Backend | `backend/` | API Fastify com auth, busca, sync, versionamento e persistência |
| CLI | `packages/cli/` | CLI publicável no npm (`myinst`) para login/list/pull/push fora do fluxo MCP |
| MCP Server | `packages/mcp-server/` | Servidor MCP local publicável (`myinst-mcp`) que conecta clientes ao vault |
| Shared | `packages/shared/` | Schemas Zod, tipos TypeScript e constantes compartilhados |

Repositório: `git@github.com:davidassef/MyInst-mcp.git`
Licença: **AGPL-3.0**

---

## Stack e pré-requisitos

- **Linguagem:** TypeScript 5.8+
- **Runtime:** Node.js **>= 22.0.0**
- **Package manager:** pnpm **10.28.0** (configurado via `packageManager` no `package.json`; `corepack` ativado nos Dockerfiles)
- **Monorepo:** pnpm workspaces + Turborepo 2.5+
- **Backend:** Fastify 5.3, Drizzle ORM 0.44, PostgreSQL 16+, Zod, bcrypt, nanoid, diff
- **Frontend:** React 19, React Router DOM 7, Vite 6, Tailwind CSS 4, lucide-react
- **MCP:** `@modelcontextprotocol/sdk`
- **Testes:** Vitest 3+
- **Containeres:** Docker + Docker Compose

---

## Arquivos de configuração principais

### Raiz

- `package.json` — scripts do monorepo, dependências comuns, `packageManager`, `engines`
- `pnpm-workspace.yaml` — define `frontend`, `backend` e `packages/*`
- `turbo.json` — pipeline de `build`, `dev`, `test`, `lint`
- `tsconfig.base.json` — configuração base TypeScript para todo o monorepo
- `.env.example` — variáveis de ambiente obrigatórias/opcionais
- `docker-compose.yml` — stack local de desenvolvimento (API + Postgres)
- `Dockerfile` — build de produção da API (compila `shared` e `backend`)
- `Dockerfile.web` — build do frontend servido por nginx

### Backend

- `backend/package.json` — scripts e dependências
- `backend/tsconfig.json` — extende `tsconfig.base.json`
- `backend/drizzle.config.ts` — configuração do Drizzle Kit
- `backend/vitest.config.ts` — configuração dos testes

### Frontend

- `frontend/package.json`
- `frontend/tsconfig.json`
- `frontend/vite.config.ts` — proxy `/api` para `localhost:3000` quando `VITE_MYINST_API_BASE` está vazio
- `frontend/vercel.json` — configuração de deploy na Vercel (Root Directory = `frontend`)

### Pacotes publicáveis

- `packages/shared/package.json` — `@myinst/shared`
- `packages/mcp-server/package.json` — `@myinst/mcp-server`, binário `myinst-mcp`
- `packages/cli/package.json` — `@myinst/cli`, binário `myinst`

---

## Comandos de build, dev e teste

Todos os comandos partem da raiz do monorepo.

```bash
# Instalação
pnpm install

# Ambiente local
cp .env.example .env
pnpm db:push      # cria/atualiza as tabelas no Postgres local
pnpm dev          # sobe backend (localhost:3000) e frontend (localhost:5173)

# Comandos individuais
pnpm dev:backend   # turbo dev com filtro no backend
pnpm dev:frontend  # turbo dev com filtro no frontend

# Build
pnpm build         # turbo build (depende de ^build)

# Validação
pnpm lint          # turbo lint = geralmente tsc --noEmit em cada pacote
pnpm test          # turbo test
pnpm validate      # lint + build + test

# Docker / deploy local
pnpm compose:check      # valida todos os docker-compose.yml do projeto
pnpm prod:preflight     # simula produção localmente (build, sobe Postgres, schema, API, smoke)
pnpm smoke              # executa smoke test contra a API (MYINST_SMOKE_BASE_URL)

# Banco de dados
pnpm db:push            # drizzle-kit push no backend
pnpm db:migrate         # drizzle-kit migrate no backend
pnpm db:studio          # drizzle-kit studio
pnpm db:deploy:schema   # aplica schema via container Docker (usado em deploy)
pnpm db:backup          # backup do banco
pnpm db:restore <arquivo.sql>  # restore (exige MYINST_CONFIRM_RESTORE=CONFIRMO_RESTORE)
```

### Comandos por pacote

```bash
pnpm --filter @myinst/backend lint
pnpm --filter @myinst/backend test
pnpm --filter @myinst/frontend build
pnpm --filter @myinst/mcp-server test
pnpm --filter @myinst/cli test
```

---

## Organização do código

### Backend (`backend/src/`)

```
backend/src/
├── index.ts            # entrypoint: carrega env, cria app, seed e listen
├── app.ts              # factory do Fastify, plugins (cors, helmet, jwt, rate-limit) e rotas
├── env.ts              # validação de variáveis de ambiente
├── routes/             # handlers da API
│   ├── auth.ts
│   ├── oauth.ts
│   ├── workspaces.ts
│   ├── projects.ts
│   ├── content.ts
│   ├── sync.ts
│   ├── tags.ts
│   ├── search.ts
│   ├── profiles.ts
│   ├── client-profiles.ts
│   ├── project-state.ts
│   ├── usage.ts
│   └── mcp-connect.ts
├── middleware/
│   ├── auth.ts         # autenticação JWT e API key
│   ├── validation.ts   # middleware Zod
│   └── usage.ts        # limites de plano (desativados por padrão)
├── db/
│   ├── index.ts        # conexão Drizzle/Postgres
│   ├── schema.ts       # definição das tabelas
│   └── seed.ts         # seed dos planos (free/pro/unlimited)
└── lib/
    ├── workspaces.ts
    ├── client-profiles.ts
    └── client-profile-replication.ts
```

A API é prefixada em `/api/v1`. Rotas legadas usam workspace/projeto default; rotas novas usam `/workspaces/:workspaceSlug/projects/:projectSlug/...`.

### Frontend (`frontend/src/`)

```
frontend/src/
├── main.tsx
├── App.tsx
├── pages/        # Login, Dashboard, Workspace, Projeto, ApiKeys, etc.
├── components/   # Layout, BrandProvider, ContextMenu, etc.
└── lib/
    ├── api.ts    # cliente HTTP para o backend
    └── *.ts      # helpers (slug, brand, replicação)
```

### Pacotes

- `packages/shared/src/`
  - `constants.ts` — content types, client IDs, scopes, etc.
  - `schemas/index.ts` — schemas Zod de input
  - `types/index.ts` — tipos TypeScript inferidos
- `packages/mcp-server/src/`
  - `index.ts` — entrypoint MCP com registro das tools
  - `client/index.ts` — `MyInstClient`, cliente HTTP da API
  - `sync-targets/index.ts` — adapters de sync para Claude, Codex, Cursor, Gemini, OpenCode, Qwen, Aider, Antigravity
  - `applier/index.ts` — aplica conteúdo no disco (formato canônico)
  - `reader/`, `importer/`, `detector/` — leitura/importação/detecção de estruturas locais
  - `auth.ts`, `credentials.ts` — autenticação do MCP
  - `project-state.ts` — memórias, decisões e sessões
  - `client-profile-replication.ts` — replicação entre Client Profiles
- `packages/cli/src/`
  - `index.ts` — entrypoint Commander
  - `commands/{login,pull,push,list}.ts`

---

## Arquitetura de runtime

```
Máquina do usuário
  ┌──────────────┐   stdio   ┌─────────────────────┐
  │ Cliente MCP  │◄─────────►│ @myinst/mcp-server  │
  │ (Claude/etc) │           │ (Node.js, local)    │
  └──────────────┘           └──────────┬──────────┘
                                        │ HTTPS
                                        ▼
                          ┌─────────────────────────┐
                          │   MyInst Server         │
                          │  API Fastify :3000      │
                          │  PostgreSQL             │
                          └─────────────────────────┘
```

- O backend expõe a API em `PORT` (padrão `3000`).
- O frontend em desenvolvimento roda na porta `5173` e faz proxy de `/api` para `localhost:3000`.
- O MCP server roda localmente via stdio; autentica com API key ou faz login OAuth automático no navegador.
- A CLI funciona fora do MCP com comandos `login`, `pull`, `push`, `list`.

---

## Modelo de dados (Drizzle)

Entidades principais (`backend/src/db/schema.ts`):

- `plans` — planos free/pro/unlimited
- `users` — contas (email, passwordHash, OAuth opcional)
- `oauth_accounts` — vínculos OAuth Google/GitHub
- `apiKeys` — chaves de API (`myinst_...`) com hash SHA-256
- `workspaces` — isolamento de contexto
- `projects` — projeto dentro de workspace
- `folders` — pastas dentro de projeto
- `contentItems` — itens de conteúdo versionados
- `contentVersions` — histórico de versões
- `contentTags` / `tags` — tags de modelo/provider/custom
- `clientProfiles` — perfis globais por cliente (claude, codex, cursor, ...)
- `clientProfileItems` / `clientProfileItemVersions` — conteúdo global por cliente
- `projectMemories`, `projectDecisions`, `projectSessions` — Project State
- `modelProfiles` — match automático de modelo para tags
- `auditLog` — log de ações

Tipos de conteúdo (`CONTENT_TYPES`):
`skill`, `instruction`, `mcp_config`, `agent`, `command`, `hook`, `memory`, `output_style`, `setting`, `snippet`.

Clientes suportados (`CLIENT_PROFILE_IDS`):
`codex`, `claude`, `cursor`, `gemini`, `opencode`, `qwen`, `aider`, `antigravity`.

Escopos de sync (`SYNC_SCOPES`): `project`, `global`, `all`.

---

## Autenticação

- **JWT**: obtido em `POST /auth/login` ou `POST /auth/register`; usado pelo frontend.
- **API Key**: gerada em `POST /auth/api-keys`; prefixo `myinst_`; formato `myinst_[32 chars base64url]`; armazenada como hash SHA-256.
- **OAuth**: Google/GitHub opcionais; callback redireciona para `WEB_OAUTH_SUCCESS_URL` com token.

Todas as rotas protegidas exigem header `Authorization: Bearer <token|apikey>`.

---

## Convenções de código

- **Idioma:** pt-BR para nomes de negócio, comentários e documentação; inglês para APIs/frameworks.
- **Nomenclatura:** nomes devem revelar intenção (ex: `criarConteudo`, `autenticarApiKey`).
- **Early return:** prefira retornos antecipados.
- **Tipagem:** evite `any`; use Zod para validação de inputs.
- **Comentários:** comente o **porquê**, não o quê.
- **Commits:** Conventional Commits em pt-BR. Exemplos:
  - `feat: adiciona adapter do cursor para rules e mcp`
  - `fix: corrige colisão de slug ao renomear projeto`
  - `docs: reescreve guia de self-hosting`
- **Lint:** não há ESLint configurado; `pnpm lint` executa `tsc --noEmit` (ou `tsc -b` no frontend).

---

## Estratégia de testes

- **Backend (`backend/tests/api.test.ts`)**: testes de integração com Vitest. O script `backend/scripts/test.ts` sobe um container Postgres temporário, aplica o schema com `drizzle-kit push` e executa os testes.
  - Pode reutilizar um banco existente: `MYINST_USE_EXISTING_TEST_DB=1`
- **Frontend**: `vitest run` em `frontend/`.
- **MCP Server**: testes unitários em `packages/mcp-server/tests/`.
- **CLI**: `packages/cli/tests/`.
- **Smoke**: `pnpm smoke` valida endpoints reais (health, auth, sync, busca, isolamento).
- **Preflight**: `pnpm prod:preflight` simula o deploy local de ponta a ponta.

Antes de abrir PRs relevantes, rode:

```bash
pnpm validate
pnpm compose:check
```

---

## Segurança

### Gerais

- `JWT_SECRET` obrigatório em produção, com pelo menos 32 caracteres e não pode ser um placeholder.
- `DATABASE_URL`, `APP_URL`, `API_PUBLIC_URL`, `CORS_ORIGIN`, `WEB_OAUTH_SUCCESS_URL`, `OAUTH_CALLBACK_URL` são obrigatórios em produção.
- Use sempre HTTPS em produção (Traefik/reverse proxy).
- Não exponha a porta 5432 do Postgres publicamente.
- Senhas são hasheadas com bcrypt; API keys são hasheadas com SHA-256.
- O logger do Fastify redige `authorization`, `password`, `token`, `key` dos request/response bodies.
- Rate-limit de 100 req/min por usuário/IP.
- CORS valida origem estritamente contra `CORS_ORIGIN`.

### Conteúdo sincronizado

- **Nunca** sincronize segredos reais (tokens, senhas, API keys, OAuth, `.env`).
- Substitua valores sensíveis por placeholders (`{{API_KEY}}`, `{{DATABASE_URL}}`).
- Use `dryRun: true` antes de `myinst_push`/`myinst_import`.
- Project State só pode ser enviado com `metadata.reviewed=true` e passa por detecção de segredos prováveis.
- Não sincronize caches, transcripts completos, `sessions/**`, `history/**`, bancos locais ou telemetry.

---

## Deploy e operações

### Deploy recomendado

- Sempre via `git pull`; **nunca** copie arquivos manualmente para a VPS.
- **API:** Docker Compose na VPS (`deploy/docker-compose.shared-infra.yml` + `deploy/docker-compose.vps-api.yml` ou `vps-api-traefik.yml`).
- **Frontend:** Vercel, com Root Directory = `frontend`.
- **Banco e Redis:** serviços em `deploy/docker-compose.shared-infra.yml`.

### Comandos típicos de deploy

```bash
# VPS (infra compartilhada primeiro)
docker compose --env-file .env -f deploy/docker-compose.shared-infra.yml up -d

# Aplicar schema
MYINST_COMPOSE_FILE=deploy/docker-compose.vps-api.yml MYINST_ENV_FILE=.env pnpm db:deploy:schema

# Subir API
docker compose --env-file .env -f deploy/docker-compose.vps-api.yml up -d --build
```

### Variáveis críticas

| Variável | Obrigatória | Descrição |
|----------|:-----------:|-----------|
| `DATABASE_URL` | Sim | Connection string PostgreSQL |
| `JWT_SECRET` | Sim | Secret JWT (>=32 chars, não placeholder) |
| `APP_URL` | Em produção | URL pública do frontend |
| `API_PUBLIC_URL` | Em produção | URL pública da API |
| `CORS_ORIGIN` | Em produção | Origem permitida no CORS |
| `WEB_OAUTH_SUCCESS_URL` | Em produção | Retorno OAuth no frontend |
| `OAUTH_CALLBACK_URL` | Se OAuth ativo | Base dos callbacks OAuth |
| `VITE_MYINST_API_BASE` | Vercel prod | Base da API para o frontend |
| `MYINST_API_HOST` | Com Traefik | Hostname exposto pelo Traefik |
| `PORT` | Não | Padrão 3000 |
| `NODE_ENV` | Não | `development` ou `production` |

### Limites de uso

Planos são semeados automaticamente (`free`, `pro`, `unlimited`). Os limites só são aplicados quando `MYINST_ENABLE_USAGE_LIMITS=true`.

---

## Fluxo de trabalho recomendado para agentes

1. Sempre que modificar código, rode `pnpm validate` na raiz.
2. Para alterações no MCP ou adapters, rode também os testes do pacote:
   ```bash
   pnpm --filter @myinst/mcp-server test
   ```
3. Para alterações de schema, valide localmente com `pnpm db:push` e `pnpm prod:preflight`.
4. Não proponha alterações destrutivas no banco sem documentar impacto.
5. Se alterar contratos públicos (API, tools MCP, formatos de arquivo), atualize:
   - `README.md`
   - `docs/mcp-server.md`
   - `packages/mcp-server/README.md`
6. Não suba assets privados em `frontend/public/brand.local/` (já está no `.gitignore`).

---

## Documentação adicional

- `README.md` — visão geral e quick start
- `CONTRIBUTING.md` — guia de contribuição
- `docs/arquitetura.md` — arquitetura detalhada
- `docs/api.md` — referência da API
- `docs/mcp-server.md` — guia do MCP server
- `docs/self-hosting.md` — self-hosting
- `docs/go-live-checklist.md` — checklist de deploy
- `docs/publicacao-npm.md` — publicação dos pacotes npm

---
> Source: [davidassef/MyInst-mcp](https://github.com/davidassef/MyInst-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
