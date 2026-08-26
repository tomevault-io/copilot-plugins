## sistemav2

> Antes de iniciar QUALQUER tarefa não-trivial (bug fix, feature, refactoring), **sempre** crie um GitHub issue para rastrear:

# Instrucões do Projeto

## Fluxo Obrigatório

### 1. Sempre criar GitHub Issue ANTES de trabalhar
Antes de iniciar QUALQUER tarefa não-trivial (bug fix, feature, refactoring), **sempre** crie um GitHub issue para rastrear:
```
gh issue create --repo tcstulio/sistemav2 --title "TIPO: Descrição curta" --body-file <arquivo_temporario>
```
- Use labels: `bug`, `enhancement`, `security`, etc.
- Inclua: Problema, Solução, Arquivos a modificar, Critérios de aceite
- Delete o arquivo temporário após criar o issue
- Referencie o issue number nos commits

### 2. Fluxo de trabalho
1. Criar issue → 2. Implementar → 3. Commit (ref issue) → 4. PR → 5. Merge → 6. Fechar issue

### 3. Contexto do Projeto
- Backend: Express + TypeScript, roda na porta 3004, reinicia via nodemon
- Frontend: React + Vite, roda na porta 5173
- `npm run dev:all` sobe ambos via concurrently
- Túnel Cloudflare para acesso externo (porta 3003)
- Dolibarr ERP como backend de dados (API REST)
- Repo: `tcstulio/sistemav2`

### 4. Convenções
- Commits em português, referenciando issue: `fix(#122): descrição`
- PRs com descrição completa e link para issue
- Não commitar secrets (.env, credenciais)
- Rodar lint/testes antes de commitar

---
> Source: [tcstulio/sistemav2](https://github.com/tcstulio/sistemav2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
