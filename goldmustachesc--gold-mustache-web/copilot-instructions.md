## gold-mustache-web

> Diretrizes globais para agentes e IA neste repositório. Este arquivo é a fonte canônica e único contrato compartilhado entre **Claude Code**, **Codex** e **OpenCode** — todos os 3 leem `AGENTS.md` automaticamente. Workflows longos ficam em `.claude/skills/` (espelhados via symlink em `.opencode/command/` quando aplicável).

# Gold Mustache - AI Development Guidelines

## Overview

Diretrizes globais para agentes e IA neste repositório. Este arquivo é a fonte canônica e único contrato compartilhado entre **Claude Code**, **Codex** e **OpenCode** — todos os 3 leem `AGENTS.md` automaticamente. Workflows longos ficam em `.claude/skills/` (espelhados via symlink em `.opencode/command/` quando aplicável).

## Ferramentas suportadas

| Provider | Lê | Tooling |
|---|---|---|
| Claude Code | `AGENTS.md`, `CLAUDE.md` (→ `@AGENTS.md`), `.claude/{skills,agents,commands,hooks}/`, `.claude/settings.json`, `.mcp.json` | hooks runtime, slash commands, MCP project |
| Codex | `AGENTS.md`, `~/.codex/config.toml` (user-level) | MCP user-level (`scripts/setup-codex-mcp.sh` adiciona vercel + prisma) |
| OpenCode | `AGENTS.md`, `opencode.json`, `.opencode/{command,plugins}/` | MCP project, plugins (hooks), slash commands |

Nenhum provider lê configuração específica do outro: contrato precisa estar em `AGENTS.md` ou ser duplicado em arquivos paralelos. Quando precisar adicionar diretriz nova, **edite `AGENTS.md` primeiro**.

## Repository structure

- `src/app` — Next.js App Router, layouts, route handlers
- `src/components/ui` — primitivos; `src/components/custom` — widgets específicos
- `src/hooks`, `src/utils`, `src/lib`, `src/services` — lógica compartilhada e integrações
- `src/config`, `src/constants`, `src/types` — config e contratos; `public/` — assets
- Imports com alias `@/` (ver `tsconfig.json`)

## Commands

| Comando | Uso |
|---------|-----|
| `pnpm install` | Dependências; manter lockfile commitado |
| `pnpm dev` | Dev local (Turbopack), `http://localhost:3001` |
| `pnpm build` | Build de produção |
| `pnpm start` | Servir build compilado |
| `pnpm lint` / `pnpm format` | Biome |
| `pnpm test` / `pnpm test:watch` | Vitest |
| `pnpm test:gate` | Lint + test + coverage — antes de PR |
| `pnpm snyk:test` / `pnpm snyk:code` | Snyk SCA e SAST locais |

## Core

- TypeScript em todo código novo (`.ts`/`.tsx`). Imports com alias `@/`.
- **Proibido `any`**. Preferir tipos explícitos, `unknown` com narrowing, generics e utility types.
- **TDD obrigatório** (RED → GREEN → REFACTOR) para: lógica de negócio (`src/services/**`), hooks (`src/hooks/**`), route handlers (`src/app/api/**`), utils (`src/utils/**`), integrações (`src/lib/**`). Rodar `pnpm test` após cada etapa.
- **TDD opcional** para: componentes UI sem lógica em `src/components/ui/**` (Tailwind puro, primitivos). Snapshot ou visual regression cobre melhor.
- Cobertura medida por **caminho crítico** (auth, booking, billing, fidelidade), não % global.
- Clean Code, SOLID, KISS, YAGNI; evitar overengineering.
- Biome: seguir `biome.json` (indentação, import sorting). Não duplicar o que o linter já garante.
- Decisões arquiteturais relevantes: documentar em PR ou na resposta ao revisor, não em comentários no código.
- Preferir código autoexplicativo; evitar comentários desnecessários.

### Anti-padrões

| Anti-padrão | Sintoma | Mitigação |
|---|---|---|
| Skip do RED | Teste passa de primeira sem falhar antes | Confirmar que teste falha antes de implementar |
| `any` no TS | "tipagem depois" | Bloqueado por Biome/tsconfig |
| Vibe coding em superfície sensível | Auth/billing sem spec nem teste | Risco ALTO força tier Full (`.kiro/TIERS.md`) |
| Multi-agente sem orchestrator | Trabalho duplicado, conflitos | Definir lead session |
| Spec inflada | brainstorm.md de 5k palavras pra feature pequena | Cap: brainstorm ≤ 800, requirements ≤ 1200 palavras |
| Gate skipado | "depois eu rodo" | CI bloqueia merge sem `test:gate` verde |

## SDD — Subagent-Driven Development

- Aplicado com cerimônia proporcional ao tier (ver `.kiro/TIERS.md`).
- Classificar antes de executar: **Trivial** → execução direta, **Light** → só requirements, **Full** → brainstorm + spec completa.
- Mentalidade SDD sempre ativa (decomposição, delegação, paralelização, checkpoints); cerimônia apenas quando agrega valor.
- `SDD` complementa `TDD`, testes, validação final. Nunca usar `SDD` como justificativa pra pular testes, revisão de impacto, confirmação de causa raiz ou validações necessárias.

### Quando NÃO delegar pra subagent

- Task sequencial e curta (< 10 min de implementação)
- Resposta cabe em 1 ferramenta (1 grep, 1 read, 1 edit)
- Decisão crítica que precisa de contexto cumulativo da sessão
- Edição de 1 linha em 1 arquivo conhecido

## App Router e APIs

Aplica-se a `src/app/**/*.{ts,tsx}`.

- Respeitar limites server/client: `"use client"` só quando necessário; dados sensíveis e secrets só no servidor.
- Route handlers: validar entrada com Zod; respostas HTTP adequadas; não expor stack traces nem detalhes internos ao cliente.
- Autenticação/autorização: verificar sessão Supabase e regras de negócio antes de operações sensíveis.
- Preferir helpers de `@/lib/api/response` (`apiSuccess`, `apiError`, `apiCollection`, etc.) nas route handlers.
- Server actions: mesma disciplina de validação e permissões que em rotas HTTP.

## Prisma e Supabase

Aplica-se a `prisma/**/*`, `src/lib/**/*`, `src/services/**/*`, `src/app/api/**/*`.

- Queries: `select`/`include` mínimos necessários; evitar N+1 em mudanças novas.
- Transações para operações que devem ser atômicas.
- Migrações: revisar impacto em dados; não commitar secrets; alinhar com `prisma/schema.prisma`.
- Supabase: não expor service role ao cliente; validar auth e autorização no servidor.
- Ao assumir RLS ou políticas, documentar no PR se o comportamento depender delas.
- Validar entrada com Zod em route handlers; nunca expor stack traces ao cliente.
- Verificar sessão Supabase antes de operações sensíveis.
- Nunca commitar credenciais em código ou em arquivos MCP versionados.

## Frontend e componentes

Aplica-se a `src/components/**/*.tsx`.

- Antes de mudanças visuais, ler `docs/Brand_Book_Gold_Mustache.md`.
- Design tokens: `src/app/globals.css` (`--primary`, `--background`, etc.). Marca: `src/config/barbershop.ts`.
- Suportar Light e Dark mode conforme Brand Book.
- Estilos: Tailwind; `clsx`/`tailwind-merge` para classes condicionais.
- Componentes em PascalCase; primitivos em `src/components/ui`, widgets em `src/components/custom`.
- Acessibilidade: semântica HTML, `aria-*` quando necessário, navegação por teclado em interativos.
- Tom de voz da marca: direto, caloroso, profissional.

## Testes (Vitest)

Aplica-se a `**/*.test.ts`, `**/*.test.tsx`, `**/*.property.test.ts`.

- Localização: `__tests__/` no módulo ou `*.test.ts` / `*.test.tsx` junto ao arquivo. Property-based: `*.property.test.ts`.
- Stack: Vitest, Testing Library, jest-dom, user-event, happy-dom, fast-check para regras de negócio quando fizer sentido.
- Mockar apenas externas (Prisma, Supabase, APIs). `vi.mock()` com o mínimo necessário.
- Hooks em mocks: usar `vi.hoisted()` quando variáveis forem referenciadas dentro de `vi.mock()`.
- `vitest.setup.ts` pode falhar em `console.error`/`console.warn` inesperados; usar spy e restore se precisar.
- Cobrir happy path, erros e edge cases. TDD: testes antes da implementação para código novo.

### Mocks frequentes (referência)

```typescript
vi.mock("@/lib/prisma", () => ({ prisma: { model: { method: vi.fn() } } }));

vi.mock("@/lib/supabase/server", () => ({
  createClient: vi.fn().mockResolvedValue({
    auth: { getUser: () => mockGetUser() },
  }),
}));

vi.mock("@/lib/auth/requireAdmin", () => ({
  requireAdmin: () => mockRequireAdmin(),
}));

const mockFn = vi.hoisted(() => vi.fn());
vi.mock("@/hooks/useAuth", () => ({ useSignIn: () => ({ mutate: mockFn }) }));
```

Comandos: `pnpm test`, `pnpm test:watch`, `pnpm test:gate`, variantes de coverage conforme `package.json`.

## Review (high signal)

- Priorizar **alto sinal**: bugs, segurança, regressões, testes ausentes para código novo, violações claras de `AGENTS.md`.
- **Não** gastar tempo com nitpicks que o Biome já cobre ou que não mudam comportamento.
- Severidade sugerida: Crítico, Alto, Médio, Baixo — com evidência no diff ou no arquivo.
- Código novo sem testes quando TDD é obrigatório: tratar como problema sério.
- Antes de "aprovar", quando aplicável, validar que `pnpm test`, `pnpm lint` e `pnpm build` foram considerados ou explicitamente justificados.

## Commits and PRs

- Conventional Commits (Commitlint); `pnpm commit` (Commitizen). Tipos: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- Assunto com até 72 caracteres, imperativo, sem ponto final
- PR: resumo, issue linkada, screenshots se UI, verificação manual ou gaps

## MCP (ferramentas externas)

Servidores MCP versionados:
- **Claude Code**: `.mcp.json` (prisma, vercel, snyk)
- **OpenCode**: `opencode.json` mesmo set
- **Codex**: rodar `bash scripts/setup-codex-mcp.sh` uma vez por máquina pra adicionar vercel + prisma ao `~/.codex/config.toml` (snyk-security já vem por default no setup global)

Segredos e integrações pessoais ficam em arquivos de configuração globais por ferramenta (`~/.codex/`, `~/.claude.json`, etc.), nunca commitados.

## Specs and complex features

Use Kiro proporcional ao tier (ver `.kiro/TIERS.md`):
- **Trivial**: execução direta, sem spec
- **Light**: `.kiro/specs/[feature]/` com `requirements.md` apenas
- **Full**: `.kiro/specs/[feature]/` com `brainstorm.md` + `requirements.md` + `design.md` + `tasks.md`

A fonte canônica dos templates é `.kiro/settings/templates/specs/`. Os arquivos `.kiro/SPECIFICATION_TEMPLATE.md`, `.kiro/DESIGN_TEMPLATE.md` e `.kiro/TASKS_TEMPLATE.md` são referência legada e não devem divergir dos templates ativos.


# AI-DLC and Spec-Driven Development

Kiro-style Spec Driven Development implementation on AI-DLC (AI Development Life Cycle)

## Project Context

### Paths
- Steering: `.kiro/steering/`
- Specs: `.kiro/specs/`

### Steering vs Specification

**Steering** (`.kiro/steering/`) - Guide AI with project-wide rules and context
**Specs** (`.kiro/specs/`) - Formalize development process for individual features

### Active Specifications
- Check `.kiro/specs/` for active specifications
- Use `/kiro/spec-status [feature-name]` to check progress

## Development Guidelines
- Think in English, generate responses in Portuguese. All Markdown content written to project files (e.g., requirements.md, design.md, tasks.md, research.md, validation reports) MUST be written in the target language configured for this specification (see spec.json.language).

## Workflows por tier

### Tier Trivial
- Execução direta, sem spec

### Tier Light
- `/kiro/spec-init "descrição"`
- `/kiro/spec-requirements {feature}`
- Implementação direta (sem design.md, sem tasks.md)

### Tier Full
- Phase 0 — Brainstorm (obrigatório): `/kiro/spec-brainstorm "tema"` → gera `brainstorm.md`
- Phase 0b (opcional): `/kiro/steering`, `/kiro/steering-custom`
- Phase 1 — Specification:
  - `/kiro/spec-init "descrição"`
  - `/kiro/spec-requirements {feature}`
  - `/kiro/validate-gap {feature}` (opcional: gap analysis)
  - `/kiro/spec-design {feature} [-y]`
  - `/kiro/validate-design {feature}` (opcional: design review)
  - `/kiro/spec-tasks {feature} [-y]`
- Phase 2 — Implementation: `/kiro/spec-impl {feature} [tasks]`
  - `/kiro/validate-impl {feature}` (opcional: verificação final)
- Progress check: `/kiro/spec-status {feature}` (use anytime)

## Development Rules
- 3-phase approval workflow: Requirements → Design → Tasks → Implementation
- Human review required each phase; use `-y` only for intentional fast-track
- Keep steering current and verify alignment with `/kiro/spec-status`
- Follow the user's instructions precisely, and within that scope act autonomously: gather the necessary context and complete the requested work end-to-end in this run, asking questions only when essential information is missing or the instructions are critically ambiguous.

## Steering Configuration
- Load entire `.kiro/steering/` as project memory
- Default files: `product.md`, `tech.md`, `structure.md`
- Custom files are supported (managed via `/kiro/steering-custom`)

---
> Source: [goldmustachesc/gold-mustache-web](https://github.com/goldmustachesc/gold-mustache-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
