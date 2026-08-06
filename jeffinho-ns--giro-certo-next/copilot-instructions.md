## giro-certo-next

> > Este repositório faz parte da iniciativa **Loja Virtual (Giro Certo)**, que abrange três repos:

# AGENTS.md — Instruções para qualquer IA / agente

> Este repositório faz parte da iniciativa **Loja Virtual (Giro Certo)**, que abrange três repos:
> `giro-certo-api` (backend), `giro-certo-next` (web) e `giro-certo-flutter` (app).
>
> **ANTES de qualquer tarefa, leia o documento mestre [`PLANO_LOJA_VIRTUAL.md`](./PLANO_LOJA_VIRTUAL.md) na raiz.** Ele contém o objetivo, a arquitetura, o modelo de dados, o roadmap e os checklists de segurança. Siga-o como fonte de verdade.

## Como você (agente) deve agir

- **Sempre alerte o usuário** (em português) quando uma ação tocar em qualquer um dos pontos críticos abaixo, antes de executá-la.
- Não desvie do plano sem avisar. Se algo no plano conflitar com a realidade do código, **pare e alerte**.
- Trabalhe pela **Fase 1** primeiro (ver `PLANO_LOJA_VIRTUAL.md`, Seção 10).

## ALERTAS OBRIGATÓRIOS (sempre avisar antes de prosseguir)

1. **Segurança mora na API, não no front.** O gate de auth aqui é client-side via `localStorage` (UX). Toda autorização real tem de ser imposta na `giro-certo-api`. Alerte se alguém tentar confiar no front como barreira.
2. **Nunca commitar segredos nem lixo de build.** Especialmente `.env`, chaves de API, tokens. Alerte e bloqueie antes de qualquer `git add`/commit amplo.
3. **Nunca confiar em preço/valor vindo do cliente.** O total é recalculado no servidor (API).
4. **Pedido só vira entrega após pagamento confirmado pelo webhook do Asaas.**
5. **Endpoints/telas públicas usam apenas dados reduzidos.** Nunca exibir CNPJ, conta bancária, e-mail do lojista, comissões ou dados de motoboy na área pública.
6. **Rastreamento com privacidade.** Acompanhamento por token; localização do motoboy só durante a entrega ativa.
7. **Separação total de áreas.** Lojista nunca acessa `/dashboard` (admin/moderador); admin é outra audiência.

## Específico deste repo (giro-certo-next)

- **Next.js 16 (App Router)** + React 19, **shadcn/Radix + Tailwind v4**, **TanStack Query**, `apiClient` em `lib/api.ts`. Reuse o design system `components/ui`.
- O `/dashboard` atual (admin/moderador) deve permanecer **intocado**.
- Novas áreas: `app/(public)/` (vitrine, sem login) e `app/(lojista)/minha-loja/` (portal do lojista). Estender `ProtectedRoute` com `requireLojista` (tem `partnerId`).
- Vitrine pública idealmente em Server Components (SEO).

## Git

- Só commitar/push quando o usuário pedir explicitamente.
- Nunca incluir segredos. Em commits, adicione arquivos específicos, não `git add .` cego.

---
> Source: [jeffinho-ns/giro-certo-next](https://github.com/jeffinho-ns/giro-certo-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
