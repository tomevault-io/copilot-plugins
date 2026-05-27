## bagre

> Notas e contexto do projeto para assistentes de IA / agentes de codificação

# AGENTS.md

Notas e contexto do projeto para assistentes de IA / agentes de codificação
trabalhando neste repositório (também útil pra humanos que queiram um overview rápido).

## Projeto

Bagre — IPAM (IP Address Management) opensource sob licença MIT. Stack:
Node 20 + Fastify + Prisma + PostgreSQL 15 + React + Vite + Tailwind, orquestrado
por Docker Compose. Substitui planilhas e ferramentas legadas de controle de IP
por uma fonte única de verdade com auditoria, busca, RBAC e API REST.

## Arquitetura (visão geral)

- **3 containers** em `docker-compose.yml`: `db` (Postgres 15), `api` (Fastify + Prisma), `web` (Vite build servido por nginx).
- **API** em `apps/api/src/`: `index.js` registra um **hook global `onRequest`** que aplica auth JWT a todas as rotas `/api/*`, com uma whitelist `PUBLIC`. Rotas individuais em `apps/api/src/routes/<area>.js`.
- **Web** em `apps/web/src/`: SPA React + Vite. Build estático servido por nginx, com proxy `/api` apontando para a API.
- **Banco**: Prisma + Postgres. Schema em `apps/api/prisma/schema.prisma`. Migrations gitignored em dev — use `prisma db push` em dev e `prisma migrate deploy` em prod.
- **Overlay opcional**: `docker-compose.zabbix-dev.yml` sobe Zabbix local para testar a integração de descoberta; `docker-compose.wiki.yml` sobe DokuWiki integrada.

## Comandos comuns

### Docker (modo principal)
```bash
docker compose up -d                                    # sobe a stack
docker compose build api && docker compose up -d api    # rebuild após mudar apps/api/
docker compose build web && docker compose up -d web    # rebuild após mudar apps/web/
docker compose logs -f api                              # logs em tempo real
docker compose down                                     # para tudo
```

### Health checks
```bash
curl http://localhost:3001/api/health                   # API viva
curl http://localhost:3000/                             # Web vivo
curl http://localhost:3001/api/stats                    # contagens (requer STATS_PUBLIC=true)
```

### Dev local (sem docker, opcional)
```bash
cd apps/api && npm install && npm run dev               # porta 3001
cd apps/web && npm install && npm run dev               # porta Vite default
```

### Prisma
```bash
cd apps/api
npm run prisma:generate    # regerar cliente
npm run prisma:push        # sync schema → DB (dev)
npm run prisma:migrate     # aplicar migrations (prod)
```

### Importação de dados (opcional)
```bash
cd apps/api && npm run import   # importa data/*.xlsx se houver
```

## Pontos de atenção

1. **`apps/` NÃO é volume-mounted no compose.** Mudou código? `docker compose build <api|web>` antes de `up -d`. Mexeu só em `.env` ou `docker-compose.yml`? `up -d` resolve.

2. **Auth é GLOBAL, não por rota.** Adicionou rota nova? Se for pública, adicione na Set `PUBLIC` em `apps/api/src/index.js`. Se for protegida, é automático.

3. **`/api/stats` é env-gated.** Só vira público se `STATS_PUBLIC=true` no `.env`. Padrão é protegido.

4. **Bootstrap admin** em `apps/api/src/index.js` cria um usuário admin no startup baseado em `BOOTSTRAP_ADMIN_EMAIL` / `BOOTSTRAP_ADMIN_PASSWORD` do `.env`. Sempre defina valores reais antes de subir em qualquer ambiente.

5. **Endpoints públicos atuais**: `/api/health`, `/api/config`, `/api/auth/login`, `/api/auth/signup`, `/api/auth/reset-request`, `/api/auth/reset`, `/api/auth/sso/start`, `/api/auth/sso/callback`, `/api/import/seed`, `/api/ingest/discoveries`, `/api/ingest/heartbeat`, `/metrics`, e `/api/stats` (se `STATS_PUBLIC=true`).

6. **Endpoints de ingest** (`/api/ingest/*`) usam **token próprio** (`INGEST_TOKEN` env var), não JWT — destinados a tooling externo (Zabbix, scanners).

7. **NÃO commitar**: `.env`, `apps/api/prisma/migrations/` em dev, `apps/web/dist`, `infra/tls/`, dados sensíveis em `data/`.

## URLs locais (após `docker compose up`)

- Web: http://localhost:3000
- API: http://localhost:3001
- Banco: localhost:5433 (porta exposta no host; container usa 5432 internamente)
- Métricas Prometheus: http://localhost:3001/metrics

## Documentação detalhada

Em `docs/` (8 arquivos numerados):

- `01-arquitetura.md` — Stack e modelo de dados
- `02-instalacao.md` — Subir do zero
- `03-uso-diario.md` — Operação pelo usuário final
- `04-administracao.md` — Usuários, perfis, RBAC
- `05-integracoes.md` — Zabbix, OIDC, Prometheus
- `06-api-rest.md` — Endpoints com exemplos `curl`
- `07-operacao.md` — Backup, restore, troubleshoot
- `08-desenvolvimento.md` — Dev local, contribuir

Versão navegável em HTML único: `docs.html` (gerada por `node scripts/build-docs-html.mjs`).

## Contribuindo

Este é um projeto opensource sob MIT. PRs e issues são bem-vindos —
ver `README.md` para diretrizes resumidas.

---
> Source: [fabgcruz/bagre](https://github.com/fabgcruz/bagre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
