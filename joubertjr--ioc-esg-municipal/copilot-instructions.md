## ioc-esg-municipal

> > **Leia inteiro antes de agir. Cada regra é operacional, não sugestão.**

# IOC ESG Municipal — CLAUDE.md

> **Leia inteiro antes de agir. Cada regra é operacional, não sugestão.**

---

## ⛔ PREMISSA ABSOLUTA E INEGOCIÁVEL — LEIA PRIMEIRO

```
O ÚNICO OBJETIVO ATUAL É:
Entregar o software funcionando para os usuários de Santa Catarina (SC)
e obter a aprovação do cliente final.

NÃO FAÇA NADA QUE NÃO SEJA NECESSÁRIO PARA O USUÁRIO FINAL DE SC USAR A PLATAFORMA.
```

**O que isso significa na prática:**

- **NÃO** implemente funcionalidades novas que não sejam demanda direta de uso do usuário de SC.
- **NÃO** evolua arquitetura, adicione camadas ou refatore se não for um bloqueador para o usuário de SC.
- **NÃO** expanda o escopo para outros estados, municípios fora de SC ou funcionalidades "nice to have".
- **NÃO** trabalhe em itens do backlog marcados como "Planejado" ou "Backlog" sem instrução explícita.
- **SIM** corrija bugs que impeçam o usuário de SC de usar a plataforma.
- **SIM** melhore estabilidade, dados e UX se for bloqueador para o uso real em SC.

**Só depois de aprovado o produto em SC pensaremos em expansão nacional, novas features ou evolução arquitetural.**

Documento de estado atual: `docs/ESTADO_ATUAL_SC.md` — leia sempre ao iniciar uma sessão.

---

## CONTEXTO DO PROJETO

**IOC ESG Municipal** = Plataforma SaaS B2G que ajuda prefeitos de Santa Catarina a investir FPM com impacto nos 17 ODS da ONU, com dados públicos reais, simulação de políticas e recomendação por IA.

**Mercado atual:** 295 municípios de Santa Catarina (FOCO EXCLUSIVO)

**Modelo:** Assinatura R$12k–200k/ano por município, 80%+ de margem

**Diferencial:** Dados públicos reais + simulação multi-agente + recomendação por IA

**Stack aprovada:**

- Backend: Node.js 20 + TypeScript strict + Express + Prisma ORM + PostgreSQL + Redis
- Frontend: React 18 + TypeScript + Vite + Tailwind CSS + Shadcn/ui + Recharts
- Testes: Vitest (unit/integration) + Playwright (e2e)
- Infra: Docker Compose (dev e prod)

---

## ESTADO OPERACIONAL — ONDE ESTÁ

Este arquivo é **instrução persistente**. Estado vivo (o que está pronto, o que falta, o que quebrou) **não** fica aqui.

| Fonte                                             | Conteúdo                                                            |
| ------------------------------------------------- | ------------------------------------------------------------------- |
| `docs/ESTADO_ATUAL_SC.md`                         | Projection humana oficial do estado atual (ler no início da sessão) |
| `logs/claude-sessions.jsonl`                      | Audit trail de sessões e eventos de runtime                         |
| `docs/architecture/CLAUDE_CODE_ADOCAO_IOC_ESG.md` | Baseline e matriz de aderência do projeto à arquitetura Claude Code |
| `.claude/GOTCHAS.md`                              | Armadilhas estáveis do domínio                                      |
| `docs/RUNBOOK_PRODUCAO.md`                        | Procedimentos operacionais de deploy, backup, SSL, alertas          |
| `docs/decisions/`                                 | ADRs — decisões arquiteturais                                       |

> **Nunca** escreva tabelas de status com ✅/⚠️/❌ neste arquivo. Elas mentem em ≤24h.

---

## INÍCIO DE SESSÃO — execute sempre

```bash
git log --oneline -5
git status --short | head -5
cat docs/ESTADO_ATUAL_SC.md
```

Reporte: **feito / em progresso / próximo passo exato**

---

## AMBIENTES — NÃO CONFUNDIR

| Ambiente          | Comando                                                                          | Composição                                                                           |
| ----------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **Dev (local)**   | `docker compose up --build`                                                      | `docker-compose.yml` — 4 containers: postgres, redis, backend (HMR), frontend (Vite) |
| **Prod (Docker)** | `docker compose -f docker-compose.prod.yml up -d`                                | Imagem multi-stage + nginx HTTP-only + postgres + redis                              |
| **Prod SSL**      | `docker compose -f docker-compose.prod.yml -f docker-compose.prod.ssl.yml up -d` | Override com certificados Let's Encrypt                                              |

**Regra operacional:** toda alteração em `backend/`, `shared/types/`, `prisma/` ou rotas exige rebuild da imagem antes de ser considerada concluída:

```bash
docker build -t ioc-esg-municipal:$(git rev-parse --short HEAD) . && echo "prod build OK"
```

Reporte explícito na mensagem de conclusão: `docker build: OK | tsc: OK | tests: OK`.

---

## COMANDOS DO PROJETO

### Dev (Docker)

```bash
docker compose up --build        # sobe os 4 containers com HMR
docker compose down
docker compose logs -f backend
pnpm db:seed                     # seed 295 municípios SC
pnpm db:migrate                  # prisma migrate dev
pnpm test                        # unit + integration
pnpm test:e2e                    # playwright
pnpm lint && pnpm format
```

### Produção

```bash
# Build da imagem multi-stage
docker build -t ioc-esg-municipal:latest .

# Subir stack de produção HTTP-only
docker compose -f docker-compose.prod.yml up -d

# Verificar saúde
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs -f api

# SSL opt-in (requer domínio configurado)
DOMAIN=app.ioc.com.br EMAIL=admin@ioc.com.br ./scripts/setup-ssl.sh
docker compose -f docker-compose.prod.yml -f docker-compose.prod.ssl.yml up -d
```

### Atualização de dados estáticos

```bash
pnpm data:update:snis        # SNIS — saneamento
pnpm data:update:inep        # IDEB — educação
pnpm data:update:sisvan      # SISVAN — nutrição
pnpm data:update:anatel      # Anatel — conectividade
pnpm data:update:aneel       # ANEEL — energia
pnpm data:update:convenios   # Convênios — transferências
pnpm data:update:ieps        # IEPS — saúde
pnpm data:update:all         # todos os 7 scripts em sequência
```

---

## ARQUITETURA DO PROJETO

```
ioc-esg-municipal/
├── backend/
│   ├── agents/          # 14+ coletores de dados (APIs reais + JSONs estáticos)
│   │   ├── ibge/        # Dados demográficos — servicodados.ibge.gov.br/api/v1
│   │   ├── siconfi/     # FPM e finanças — api.siconfi.tesouro.gov.br/v1
│   │   ├── datasus/     # Saúde — datasus.saude.gov.br
│   │   ├── inpe/        # Florestal — terrabrasilis.dpi.inpe.br/api/v1
│   │   ├── pncp/        # Licitações — pncp.gov.br/api/pncp
│   │   ├── ana/         # Água — API ANA
│   │   ├── snis-rs/     # Saneamento — SNIS RS
│   │   ├── ieps/        # Saúde — ieps_latest.json (update-ieps-data.ts)
│   │   ├── inep/        # IDEB — ideb_latest.json (update-inep-data.ts)
│   │   ├── snis/        # Saneamento — snis_latest.json (update-snis-data.ts)
│   │   ├── sisvan/      # Nutrição — sisvan_latest.json (update-sisvan-data.ts)
│   │   ├── anatel/      # Conectividade — anatel_latest.json (update-anatel-data.ts)
│   │   ├── aneel/       # Energia — aneel_latest.json (update-aneel-data.ts)
│   │   ├── convenios/   # Transferências — convenios_latest.json (update-convenios-data.ts)
│   │   └── tse/         # Eleições — tse_latest.json
│   ├── services/
│   │   ├── ods/         # Calculators de score 0-100 por ODS
│   │   ├── simulator/   # Motor de simulação FPM
│   │   ├── reports/     # Gerador de relatórios ESG
│   │   ├── benchmarks/  # Comparativo entre municípios SC
│   │   └── recommendations/ # Recomendações inteligentes por gap analysis
│   ├── routes/          # Express routers (auth, ods, simulator, reports, benchmarks)
│   └── middleware/      # Auth JWT, rate limit, logging Winston
├── frontend/
│   └── src/pages/
│       ├── LoginPage.tsx
│       ├── OnboardingPage.tsx   # Restrito a 295 municípios SC
│       ├── DashboardPage.tsx    # ODS overview + recomendações
│       ├── SimulatorPage.tsx    # Simulador de cenários FPM
│       ├── ReportsPage.tsx      # Relatórios ESG
│       ├── MonitoringPage.tsx   # Acompanhamento de metas
│       └── BenchmarkPage.tsx    # Ranking e comparativo SC
├── shared/
│   ├── types/           # Interfaces TypeScript (Município, ODS, Simulação)
│   ├── constants/       # ODS 1-17, APIs, 295 municípios SC
│   └── data/            # JSONs estáticos (*_latest.json com __meta)
├── scripts/             # update-*-data.ts (7 scripts de atualização)
├── nginx/               # nginx-http.conf (HTTP-only) + nginx.conf (SSL)
├── prisma/              # Schema + seed 295 municípios SC
└── docs/
    ├── ESTADO_ATUAL_SC.md   # ← LEIA SEMPRE AO INICIAR SESSÃO
    └── decisions/           # ADRs
```

---

## DOMÍNIO DO NEGÓCIO

### Personas

- **Prefeito:** quer gastar FPM com segurança, evitar TCE, ser reeleito
- **Secretário de Finanças:** precisa de conformidade com Lei 14.133/2021
- **Secretário de Planejamento:** quer alinhar com ODS e Agenda 2030

### Score ESG

- Cada ODS tem score 0-100 calculado a partir de indicadores públicos
- Verde ≥70 | Amarelo 40–69 | Vermelho <40
- Score global = média ponderada dos 17 ODS

### Gotchas críticos do domínio

- Código IBGE: 7 dígitos (ex: 4204202). SICONFI usa 6 dígitos sem verificador (420420)
- FPM: pago em 3 decênios/mês (dias 10, 20, 30). Some para obter valor mensal
- IDEB: bienal (anos pares). Interpole para anos intermediários
- DATASUS: cai com frequência. Sempre timeout=30s + retry 3x + backoff exponencial
- SNIS: dados chegam com ~18 meses de atraso. Sempre informe o ano de referência
- Municípios <5k hab: muitos indicadores são amostrais ou suprimidos por privacidade
- SISVAN e SNIS têm dados para 284/295 municípios SC (11 municípios sem cobertura na base federal — comportamento esperado)

---

## PADRÕES DE CÓDIGO

### TypeScript

- `strict: true` sempre, zero `any`, use `unknown` + type guards
- Zod para validar TODA resposta de APIs externas antes de usar
- Decimal.js para valores financeiros (FPM, investimentos)
- ESM nativo: use `import.meta.url` + `fileURLToPath` para `__dirname`

### Backend

- Controllers finos — lógica de negócio nos Services
- Winston para logs estruturados (nunca console.log em produção)
- Cache Redis obrigatório em toda chamada de API externa
- Retry com backoff exponencial: 1s, 2s, 4s (3 tentativas)
- Rate limiting: máx 2 req/segundo para APIs governamentais

### Coletores de dados estáticos

- Padrão: importar `*_latest.json`, extrair `__meta`, remover `__meta`, validar com `safeParse` Zod
- `REFERENCE_YEAR = _meta?.referenceYear ?? <fallback>`
- Scripts de atualização: `scripts/update-*-data.ts` — padrão CSV-first com fallback para JSON existente

### Frontend

- React Query para server state
- Componentes funcionais, tipagem explícita de props
- Lazy loading para páginas
- Skeleton loaders para dados assíncronos

### Banco de dados

- Migrations via Prisma Migrate — nunca editar SQL manualmente
- Soft delete em entidades principais
- Índices em: municipality_id, ods_number, reference_date

---

## SEGURANÇA

- NUNCA commita `.env`, chaves de API, tokens, senhas
- NUNCA loga PII (dados pessoais), apenas dados agregados por município
- Validação Zod em toda rota antes de processar
- Rate limiting em rotas públicas
- IDOR corrigido: prefeito só acessa dados do próprio município (JWT + middleware)

---

## QUALIDADE — checklist antes de "concluído"

- [ ] `tsc --noEmit` sem erros TypeScript
- [ ] `pnpm test` passando
- [ ] `docker build -t ioc-esg-municipal:<tag> .` concluído com sucesso
- [ ] Novos testes escritos para nova funcionalidade
- [ ] Sem credenciais hardcoded
- [ ] Erros tratados explicitamente (nunca silencioso)
- [ ] Cache Redis implementado se chamar API externa

---

## CONVENÇÕES DE COMMIT

```
<tipo>(<escopo>): <descrição imperativa>
- detalhe do que foi feito
- motivo da decisão
```

Tipos: `feat` `fix` `refactor` `test` `docs` `chore` `perf` `ci`
Escopos: `ibge` `siconfi` `datasus` `inep` `snis` `inpe` `pncp` `ods` `simulator` `dashboard` `auth` `db` `infra` `agents`

Nunca: `fix bug` `update` `changes` `wip`

---

## O QUE NUNCA FAZER

- Implementar sem plano aprovado para tasks > 15min
- Usar `any` em TypeScript
- Chamar API externa sem cache + retry
- Commitar sem rodar testes
- `git push --force` sem confirmação explícita
- Hardcodar IDs de município, URLs ou valores de configuração
- Misturar refatoração com nova feature no mesmo commit
- **Implementar qualquer funcionalidade não necessária para o usuário de SC usar a plataforma hoje**
- **Expandir escopo para outros estados ou municípios fora de SC sem instrução explícita**

---

## FIM DE SESSÃO — sempre

1. Commit de tudo funcionando
2. Atualize `docs/ESTADO_ATUAL_SC.md`
3. Reporte: **feito / pendente / próximo passo exato**

---

_IOC ESG Municipal — Foco SC | Última atualização: 2026-04-13_

---
> Source: [Joubertjr/ioc-esg-municipal](https://github.com/Joubertjr/ioc-esg-municipal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
