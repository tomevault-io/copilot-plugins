## plataforma-servicos

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# Plataforma de serviços de limpeza (Curitiba + RMC)

Marketplace de dois lados: **cliente** agenda limpeza, **profissional** (diarista) executa.
A plataforma faz o matching, retém o pagamento e repassa após a conclusão. Escopo completo
do produto em [`PROMPT-INICIAL.md`](PROMPT-INICIAL.md).

## Stack

- **Next.js 16** (App Router, Turbopack) + TypeScript + Tailwind v4
- **Prisma 7** + PostgreSQL (Supabase). Prisma 7 exige driver adapter (`@prisma/adapter-pg`);
  client gerado em `src/generated/prisma` (git-ignored, recriado por `postinstall`).
  Config do CLI em `prisma7.config.ts` (lê `.env` via `dotenv`).
- **Auth.js v5** (`next-auth@beta`), estratégia JWT, provider Credentials + `@auth/prisma-adapter`.
  Papéis `CLIENTE` / `PROFISSIONAL` / `ADMIN` na sessão.
- **Supabase Storage** para foto de perfil e documentos (bucket privado, LGPD).
- **Resend** para e-mail transacional.
- **Pagamento**: interface `PaymentProvider` (`src/lib/payments/`) com implementação `fake` no
  MVP. Trocar por provider real = implementar a interface + `PAYMENT_PROVIDER`.
- Deploy na **Vercel**.

## Estrutura

```
prisma/
  schema.prisma          modelo de dados completo (23 models, multi-cidade)
  seed.ts                configs, regiões, tabela de preços, usuários demo
src/
  app/
    (marketing)/         landing e páginas informativas
    (auth)/              login, cadastro
    (painel)/            área logada; layout faz requireUser, sub-layouts requireRole
      cliente/ profissional/ admin/
    agendar/             CTA público do fluxo de agendamento
    api/auth/[...nextauth]/   handlers do Auth.js
    api/webhooks/pagamento/   webhook do provedor de pagamento
  components/ui/          button, input, label, card, empty-state (estilo shadcn, manual)
  lib/
    env.ts               validação das env vars (zod) — server only
    db.ts                PrismaClient singleton + PrismaPg adapter
    auth/                config, dal (getSession/requireUser/requireRole), actions
    pricing/             preço = f(tipo, duração, região) + extras − taxa
    matching/            fila de elegibilidade + atribuição (stub)
    payments/            interface + provider fake
    storage/ email/      wrappers Supabase Storage / Resend
    regioes.ts           cidades da RMC + checagem grosseira de CEP
    validation/          schemas zod compartilhados
```

## Convenções

- **Dinheiro é sempre `Int` em centavos** (BRL). Formatação em `src/lib/format.ts`.
- Horários na agenda: `Int` = minutos desde 00:00.
- Validação de entrada **sempre no backend** (server action / route handler), mesmo com validação no client.
- Sem escopo extra sem alinhar: telas de feature ainda não construídas mostram estado vazio explicando o que falta.

## Comandos

| Comando | O quê |
|---|---|
| `npm run dev` | dev server |
| `npm run build` | `prisma generate` + `next build` |
| `npm run typecheck` | `tsc --noEmit` |
| `npm run db:migrate` | cria/aplica migration (dev) |
| `npm run db:deploy` | aplica migrations (produção) |
| `npm run db:seed` | popula configs, regiões, preços e 3 usuários demo (senha `senha12345`) |
| `npm run db:studio` | Prisma Studio |

## Estado atual

Esqueleto: build e rotas OK, auth por papel funcionando (redirect para `/login`), camadas de
domínio com assinatura definida e implementação pendente. Próximas features na ordem sugerida
no README.

---
> Source: [agentworksia/plataforma_servicos](https://github.com/agentworksia/plataforma_servicos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
