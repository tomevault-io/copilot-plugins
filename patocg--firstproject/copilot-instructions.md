## firstproject

> Guia para o Claude Code trabalhar neste projeto.

# CLAUDE.md

Guia para o Claude Code trabalhar neste projeto.

---

## Projeto

**firstproject** — portfólio pessoal + álbum de família privado.
Deploy: https://jnths.com.br
Stack: Next.js 16 + React 19, AWS S3 + DynamoDB, NextAuth v4 + Google OAuth, Vercel.

---

## Setup local

```bash
npm install
vercel env pull .env.local   # requer: vercel link (já configurado)
npm run dev
```

As variáveis de ambiente vivem na Vercel. Nunca crie `.env.local` manualmente — use sempre `vercel env pull`.

---

## Testes

```bash
npm test          # todos os testes
npm test -- --watch   # modo watch
```

- 75 testes, 14 suites — todos devem passar antes de qualquer commit
- `.env.test` fornece valores fake para `NODE_ENV=test` (Next.js não carrega `.env.local` em modo de teste)
- Nunca adicione fallbacks hardcoded em `jest.setup.js` — o `NoHardcoded.test.js` vai rejeitar

---

## Estrutura

```
pages/api/        → API Routes (handlers Next.js)
pages/api/album/  → add-photo, delete, list, list-albums, upload-url
pages/api/auth/   → [...nextauth].js
pages/api/whitelist/ → check, list
lib/              → dynamo.js, photos.js, logger.js
components/       → componentes de UI reutilizáveis
tests/api/        → testes das API routes
tests/lib/        → testes das funções de lib
tests/security/   → NoSecrets.test.js, NoHardcoded.test.js
```

---

## Convenções obrigatórias

### Variáveis de ambiente
- Todas as vars sensíveis vivem na Vercel e são puxadas via `vercel env pull`
- `OWNER_EMAIL` deve ser sempre lido via `process.env.OWNER_EMAIL` — nunca hardcoded
- Nunca use prefixo `NEXT_PUBLIC_` em variáveis sensíveis (AWS keys, emails, secrets)

### Commits
Seguir [Conventional Commits](https://www.conventionalcommits.org):
```
feat(escopo): descrição
fix(escopo): descrição
test(escopo): descrição
docs(escopo): descrição
refactor(escopo): descrição
chore(escopo): descrição
```
Commits genéricos como "update" ou "fix bug" não são aceitos.

### Branches
```
main       → produção (push direto bloqueado)
develop    → homologação
feature/*  → novas funcionalidades
fix/*      → correções
refactor/* → melhorias estruturais
hotfix/*   → correções críticas
```
PRs sempre de `feature/*` ou `develop` → `main`. Nunca push direto na `main`.

---

## O que nunca fazer

- Hardcodar e-mails, AWS keys ou qualquer segredo no código
- Commitar `.env.local`, `.env`, `.vercel/` (cobertos pelo `.gitignore`)
- Usar `NEXT_PUBLIC_` em variáveis sensíveis
- Adicionar fallbacks com valor real em `jest.setup.js` (use `.env.test`)
- Push direto na `main`
- Remover ou enfraquecer checagens de autenticação/autorização existentes

---

## Controle de acesso

Toda API que manipula fotos ou dados privados deve:
1. Verificar sessão via `getServerSession` → retornar 401 se ausente
2. Se não for owner (`email !== OWNER_EMAIL`), consultar o DynamoDB com `GetCommand` na tabela `DYNAMO_TABLE_WHITELIST`
3. Verificar a permissão específica (`canUploadPhotos`, `canDeletePhotos` etc.) → retornar 403 se ausente

---

## Issues abertas relevantes

- **#8** — CI/CD com GitHub Actions (`lint → test → build → security scan`)
- **#10** — Commitlint + Husky

---

## Documentação

- `README.md` — visão geral e instruções de uso
- `ROADMAP.md` — fases de evolução e status atual
- `CHANGELOG.md` — histórico de mudanças por versão
- `CONTRIBUTING.md` — guia para contribuições
- `docs/PERMISSIONS.md` — matriz de permissões detalhada
- `docs/ADMIN_PAINEL.md` — documentação do painel admin

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patocg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
