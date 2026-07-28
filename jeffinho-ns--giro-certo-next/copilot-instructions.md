## loja-virtual

> Iniciativa Loja Virtual (Giro Certo) - guia e alertas obrigatórios


# Loja Virtual (Giro Certo) — leia antes de agir

**Leia `PLANO_LOJA_VIRTUAL.md` e `AGENTS.md` na raiz antes de qualquer tarefa.** São a fonte de verdade (arquitetura, modelo de dados, roadmap, segurança).

## Sempre alerte o usuário (em português) antes de prosseguir quando:

- For confiar no gate client-side (`localStorage`) como segurança → **a barreira real é a API**.
- For commitar/adicionar segredos (`.env`, chaves, tokens). Bloqueie e avise.
- For usar preço/valor vindo do cliente → **o total é validado no servidor**.
- For exibir em tela/endpoint público dados sensíveis (CNPJ, conta bancária, e-mail do lojista, comissões, dados de motoboy).
- For deixar lojista acessar `/dashboard` (admin/moderador) ou vice-versa → **áreas separadas**.

## Padrões deste repo (giro-certo-next)

- Next.js 16 (App Router), React 19, shadcn/Radix + Tailwind v4, TanStack Query, `apiClient` em `lib/api.ts`. Reuse `components/ui`.
- `/dashboard` (admin) permanece **intocado**.
- Áreas novas: `app/(public)/` (vitrine, sem login) e `app/(lojista)/minha-loja/`. Estender `ProtectedRoute` com `requireLojista` (`partnerId`).
- Vitrine pública idealmente em Server Components (SEO).
- Só commitar/push quando o usuário pedir. Nunca `git add .` cego.

---
> Source: [jeffinho-ns/giro-certo-next](https://github.com/jeffinho-ns/giro-certo-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
