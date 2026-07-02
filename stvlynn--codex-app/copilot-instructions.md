## codex-app

> > Read this file first. Then follow the documentation map below before writing any code or changing the deobfuscation pipeline.

# Agentic Coding Guidelines

> Read this file first. Then follow the documentation map below before writing any code or changing the deobfuscation pipeline.

---

## ⚠️ Before you start

1. **Read the docs first.** Do not assume conventions; this project is documented so agents do not have to guess.
2. **Ask when boundaries are unclear.** If a requirement, layer boundary, pipeline stage, or technology choice is ambiguous, ask the user before proceeding.
3. **Check logs and docs before inventing workarounds.** When something does not behave as expected, inspect logs, tests, and the relevant `docs/` file. Do not add fallback logic to bypass a problem you have not understood.

---

## Documentation map

### Understand the project

- [`docs/README.md`](docs/README.md) — top-level index and domain map.
- [`docs/project/README.md`](docs/project/README.md) — project overview, goals, and boundaries.
- [`docs/project/architecture.md`](docs/project/architecture.md) — system architecture, module boundaries, and data flow.

### Work on restored UI code

- [`docs/frontend/README.md`](docs/frontend/README.md) — organization of React/TypeScript code in `./src/`.

### Work on the deobfuscation pipeline

- [`docs/backend/README.md`](docs/backend/README.md) — pipeline layers and conventions.
- [`docs/quality/README.md`](docs/quality/README.md) — quality gates and review expectations.

### Operations and evolution

- [`docs/operations/README.md`](docs/operations/README.md) — local development, skill execution, deployment.
- [`docs/decisions/README.md`](docs/decisions/README.md) — architecture decision records (ADRs).

---

## Language and quality rules

### English only for code and comments

- All source code, comments, commit messages, and internal identifiers must be written in English.
- User-facing copy in restored code must also be in English unless the original Codex app used another language.

### Forbidden patterns

- **No hardcoded strings.** All user-facing text must be defined in a single place (constants, i18n keys, configuration) and referenced by identifier.
- **No redundant UI copy.** Do not repeat information that is already conveyed by a title, icon, selected state, or surrounding context.
- **No duplicated implementations.** If a piece of logic already exists, reuse it or extract it to the correct layer/slice. Do not copy-paste with minor variations.
- **No fallback/clever bypass logic.** Do not mask a root cause with a default value, a silent catch, or a conditional shortcut. Face the actual problem and fix it, or ask the user.
- **No assumptions about hidden requirements.** If a feature is not documented and the intent is unclear, ask.

### When something is unclear

1. Search the current `docs/` for the topic.
2. Read the relevant layer README.
3. Inspect recent logs, checkpoints, or existing code.
4. If still unclear, ask the user with a concise, specific question.

---

## Restored UI code

- Code in `./src/` follows Feature-Sliced Design: `app/`, `pages/`, `widgets/`, `features/`, `entities/`, `shared/`.
- Imports go downward: upper layers may import from lower layers; shared code must not import from app/pages/widgets/features/entities.
- Each feature exposes a public API through its barrel (`index.ts`). Do not import internals of another feature.
- React components must have typed props; exported functions must have typed parameters in deep mode.
- Vendored dependencies are represented by typed facades in `shared/boundaries/` or left as bare npm imports.

See [`docs/frontend/README.md`](docs/frontend/README.md) for the full rules.

---

## Deobfuscation pipeline

- The pipeline is organized into layers: **domain** (symbols, import graph, quality rules), **application** (stage orchestration), **infrastructure** (parsing, formatting, file I/O), and **interfaces** (skill entry points and CLI scripts).
- Dependencies point inward: interfaces → application → domain; infrastructure implements domain abstractions.
- A chunk is promoted to `./src/` only after passing `quality-gate.ts`.
- The promotion frontier drains producers before consumers to keep `IMPORT_MAP.json` and relative imports consistent.
- Vendored/npm-leaf chunks are not restored as files; they are either facaded in `boundaries/` or imported as bare npm specifiers.

See [`docs/backend/README.md`](docs/backend/README.md) for the full rules.

---

## Self-evolution rule

After completing a task, update the documentation if the task has changed any of the following:

- Product behavior or user-facing workflows in restored code.
- Architecture, layer responsibilities, or module boundaries.
- Pipeline stages, quality gates, or promotion rules.
- Runtime configuration, skill execution, or deployment steps.
- Testing expectations or quality gates.
- Coding conventions, style rules, or directory boundaries.

**How to update docs:**

1. Edit the relevant existing doc in the same change set as the code.
2. If no doc exists for the new topic, create one under the correct `docs/` subdomain and add it to the nearest `README.md`.
3. Keep docs concise, factual, and example-driven. Avoid future dates, day estimates, and speculative language.
4. Do not let implementation and documentation drift.

---

## Commit conventions

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>
```

- `type`: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`, `revert`.
- `scope`: module or layer name, e.g., `deobfuscate`, `frontend`, `pipeline`, `docs`.
- `subject`: lowercase, concise, no trailing period.

Examples:

- `feat(deobfuscate): add semantic path for browser sidebar chunks`
- `docs(frontend): clarify boundary import rules`
- `fix(pipeline): resolve duplicate export detection in quality gate`

---

## Planning documents

- ❌ Do not use concrete dates ("2026-01-31").
- ❌ Do not estimate durations ("3 days", "Week 1").
- ✅ Describe task order and dependencies only.
- ✅ Use priorities `P0/P1/P2` to indicate importance.

---

## Legal and ethical note

This project is for **personal study and interoperability research** of software you have installed. The extracted code is © OpenAI; respect the Codex app's license and terms of service. Do not redistribute the extracted or restored code.

---
> Source: [stvlynn/codex-app](https://github.com/stvlynn/codex-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
