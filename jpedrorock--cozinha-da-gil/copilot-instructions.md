## cozinha-da-gil

> > Lido automaticamente pelo Claude Code no início de toda sessão. Estável — detalhes vivos vão em STATUS.md / BACKLOG.md / PLAYBOOK.md.

# CLAUDE.md — Entrypoint do Pastel da Gil (Cozinha da Gil)

> Lido automaticamente pelo Claude Code no início de toda sessão. Estável — detalhes vivos vão em STATUS.md / BACKLOG.md / PLAYBOOK.md.

---

## 0. RITUAL OBRIGATÓRIO

Toda sessão começa lendo, **nessa ordem**:

1. **`STATUS.md`** — estado atual, bloqueios, próximo passo
2. **`BACKLOG.md`** — fila priorizada de trabalho
3. **`PLAYBOOK.md`** — regras de autonomia (presencial vs background)

Se for primeira sessão: ler tudo dos 3 arquivos + `README.md` + `docs/FASE-6.md` (estado da fase atual).

---

## 1. O que é o Pastel da Gil

App de pedidos pra barraca de pastel da família da Gil. **Roda local** num laptop ou mini-PC durante eventos — tablets/celulares se conectam via wifi local. Não precisa de internet pro app funcionar, só pra WhatsApp.

**4 papéis no app:**
- `app/atendente/` — quem anota o pedido
- `app/cozinha/` — quem prepara, vê pedidos em tempo real (SSE)
- `app/cliente/` — fluxo do cliente (status do pedido, comprovante)
- `app/admin/` — Gil/dono gerencia produtos, fechamento, relatórios

Contexto crítico: **produção é a barraca em evento real.** Bug em produção = fila parada = cliente irritado. Mudanças sensíveis exigem cautela extra.

## 2. Estrutura do repo

```
/
├── CLAUDE.md                  ← você está aqui
├── STATUS.md                  ← estado atual
├── BACKLOG.md                 ← fila priorizada
├── PLAYBOOK.md                ← regras de autonomia
├── ROTINA.md                  ← guia de uso do sistema
├── README.md                  ← guia oficial do projeto
│
├── app/                       ← Next.js 14 App Router
│   ├── atendente/  cozinha/  cliente/  admin/  ← rotas por papel
│   ├── comprovante/           ← geração de comprovante (PDF)
│   ├── api/                   ← endpoints (incl. SSE)
│   ├── design-system/         ← tokens e componentes base
│   ├── guia/                  ← guia visual interno
│   └── layout.tsx page.tsx
├── components/                ← componentes compartilhados
├── lib/                       ← util, db, session, sse helpers
├── prisma/                    ← schema.prisma + seed.ts + backfills + dev.db
├── docs/
│   ├── FASE-6.md              ← fase atual de desenvolvimento
│   ├── IMPRESSORA-TERMICA.md  ← integração com impressora térmica
│   ├── HTTPS-LOCAL.md         ← cert pra rodar via HTTPS local
│   └── BACKGROUND-SYNC.md     ← sincronização offline
├── e2e/ tests/                ← Playwright e2e + Vitest unit
├── public/                    ← PWA assets
├── scripts/                   ← scripts de operação
├── ecosystem.config.js        ← PM2
├── Dockerfile docker-compose.yml docker-entrypoint.sh
└── playwright.config.ts vitest.config.ts tailwind.config.ts
```

## 3. Stack rápida

- **Framework**: Next.js 14 (App Router) com TypeScript
- **DB**: Prisma + SQLite (`dev.db` local, file-based)
- **Auth**: iron-session (cookie criptografado) + bcryptjs
- **Real-time**: Server-Sent Events (SSE) — pedidos aparecem ao vivo na cozinha
- **PWA**: `next-pwa` (instalável em tablet/celular)
- **PDF**: PDFKit (comprovantes, impressora térmica)
- **Testes**: Vitest (unit, 51+) + Playwright (e2e)
- **Prod local**: PM2 (`ecosystem.config.js`) ou Docker (Coolify)
- **Pacote**: npm (NÃO pnpm)

## 4. Comandos mais comuns

```bash
npm install                  # ATENÇÃO: npm, não pnpm
npm run dev                  # http://localhost:3000
npx prisma db push           # sincroniza schema com dev.db (LOCAL)
npm run db:seed              # popula Gil/Maria/José + produtos
npm run db:studio            # Prisma Studio no browser
npm test                     # vitest run (rodar antes de fechar item)
npm run test:e2e             # playwright e2e
npm run lint                 # eslint
npm run build && npm start   # build + servidor de produção (PM2 chama isso)
```

Senha default dos usuários seed (dev): `1234`.

## 5. Identidade da conversa

Esse projeto tem **uma única identidade**:

- `claude-pastel` — qualquer trabalho neste repo

A primeira mensagem da sessão deve incluir: "Sou `claude-pastel`, vou trabalhar em \_\_\_."

## 6. Slash commands disponíveis

(em `.claude/commands/`)

- `/trabalhar` — pega próximo item do BACKLOG, implementa, valida com checker, atualiza STATUS
- `/planejar` — revisa o estado, propõe novos itens
- `/brief` — resumo do que rolou desde a última leitura
- `/noite` — fechamento do dia, prepara snapshot pro briefing matinal

## 7. Regra de ouro

> Esse app vai pra evento real onde família depende dele rodando. Em dúvida sobre mudança que afeta produção: **abra PR**. Não comite direto em main mudança em SSE, sessions, schema, ou auth sem revisão.

## 8. Quando estiver em dúvida

Pare. Escreva em `STATUS.md` → "Bloqueios ativos". Decisões de produto/UX/fluxo da barraca são da Gil/João — execução é sua.

---
> Source: [jpedrorock/cozinha-da-gil](https://github.com/jpedrorock/cozinha-da-gil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
