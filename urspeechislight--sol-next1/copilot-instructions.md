## sol-next1

> You are operating inside a project that uses an agent guardrail harness. Hooks

# Agent guidance — sol-next1

You are operating inside a project that uses an agent guardrail harness. Hooks
under `.claude/hooks/` will block tool calls that violate the rules below. Read
the relevant docs before editing — the harness is strict by design.

## Required reading (in order)

1. [`docs/foundation.md`](./docs/foundation.md) — project purpose, scope,
   non-goals.
2. [`docs/repo-layout.md`](./docs/repo-layout.md) — where every file belongs.
3. [`docs/design-system.md`](./docs/design-system.md) — the SSOT contract:
   tokens, primitives, variants.
4. [`docs/quality-standards.md`](./docs/quality-standards.md) — every rule the
   harness enforces.
5. [`docs/contributing.md`](./docs/contributing.md) — workflow, commands,
   conventions.

## Rules summary (the harness will block violations)

**Design system (SSOT/DRY)**

- Colors, typography, spacing, radii, shadows, motion — defined ONLY in
  `src/frontend/lib/design-system/tokens.css`.
- No raw hex/rgb/hsl color literals outside `tokens.css` and `internal/`.
- No inline `style="..."` attributes anywhere in `.svelte` files.
- Variant maps (status → color/badge/etc.) live ONLY in
  `src/frontend/lib/design-system/variants.ts`.
- Pages and routes consume primitives from `lib/design-system/`. Raw `<button>`,
  `<input>`, `<a>` are blocked in routes — use the primitives.

**Code quality**

- Files: warn ≥300 LOC, block ≥500 LOC.
- Functions: block ≥80 LOC.
- No bare `except:` or `except X: pass`.
- No magic numbers (literals other than `-1, 0, 1, 2, 100`); use named
  constants.
- All Python functions need type hints + docstrings.
- Constants live in their service's `core/constants.py`, naming
  `UPPER_SNAKE_CASE`.
- No `print` in non-script code; use `structlog`.
- Test names: `test_should_<verb>_<object>_<condition>`.

**Imports / boundaries**

- Routes can only import from `$lib/design-system/`, `$lib/stores/`,
  `$lib/server/`, `$lib`.
- `$lib/design-system/internal/` is private — only design-system files import
  it.
- Backend cannot import from frontend; frontend talks to backend via HTTP.

**Security**

- No secrets in code. Hooks scan for AWS/GCP keys, API tokens, private keys,
  `.env` patterns.
- No `eval`, `exec`, `__import__` of dynamic strings.
- No `os.system`, `subprocess.shell=True` in app code.

## Running checks locally

```bash
pnpm ci             # everything
pnpm lint           # JS/TS only
pnpm check          # TS + svelte-check
pnpm py:check       # ruff + pyright
pnpm harness:test   # harness self-tests
```

## When the harness blocks you

Each block emits a structured message: `rule_id`, `why`, `how to fix`. Don't
bypass — read the linked doc and adjust. If you genuinely think the rule is
wrong, open a doc PR proposing the change; do not edit `.claude/policies/` to
wave yourself through.

---
> Source: [urspeechislight/sol-next1](https://github.com/urspeechislight/sol-next1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
