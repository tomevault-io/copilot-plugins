## assistencianet

> - SmartConserto is a SaaS for assistencias tecnicas with multi-branch operation.

# Agent Instructions

## Product
- SmartConserto is a SaaS for assistencias tecnicas with multi-branch operation.
- Current pilot customer is Orquidia Assistencia Tecnica; treat its workflow as the default domain baseline until product decisions say otherwise.
- Prioritize operational simplicity, auditability, branch visibility, and PT-BR UX.

## Source Of Truth
- Raw interview: `doc/resposta.txt`
- Pilot summary: `doc/customer-profile-orquidia.md`
- Product/domain rules: `doc/product-context.md`
- Web-specific conventions: `apps/web/AGENTS.md`

## Package Manager
- Use **npm workspaces**: `npm install`, `npm run dev`, `npm run build`, `npm run lint`
- Web app workspace: `npm run dev --workspace @smartconserto/web`

## File-Scoped Commands
| Task | Command |
|------|---------|
| Lint one file | `npx eslint apps/web/src/path/to/file.tsx` |
| Typecheck web app | `npx tsc --noEmit -p apps/web/tsconfig.json` |
| Run web app | `npm run dev --workspace @smartconserto/web` |

## Key Conventions
- UI copy is PT-BR only unless a task explicitly requires another language.
- Product direction is light-mode-first and desktop-first with responsive fallback.
- Reuse shared dashboard and table patterns before creating new management layouts.
- Keep Supabase migrations in `apps/web/supabase/migrations/` with timestamped filenames aligned to remote history.
- Toda migration Supabase deve ser aplicada no projeto remoto antes de concluir a tarefa, preferencialmente via MCP/Management API do Supabase.
- Não deixar fallback temporário ou compatibilidade paliativa por migration não aplicada, salvo quando o usuário pedir isso explicitamente.
- Durante a fase de desenvolvimento, o Supabase remoto é o baseline operacional para reconciliar drift; após reconciliar, Git + migrations aplicadas voltam a ser a fonte de verdade.
- Não editar migrations já aplicadas; corrigir drift com nova migration ou, se autorizado em dev, remover/recriar migrations/dados para alinhar remoto e local.
- Preserve soft delete and audit log behavior when changing branches, employees, or other administrative entities.
- Treat WhatsApp, client/equipment history, branch separation, and role-based permissions as core product concerns.

## Commit Attribution
- AI commits MUST include:
```text
Co-Authored-By: Codex GPT-5 <noreply@openai.com>
```

---
> Source: [luciovitorio/assistencianet](https://github.com/luciovitorio/assistencianet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
