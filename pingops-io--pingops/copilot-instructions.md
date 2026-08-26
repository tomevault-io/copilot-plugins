## pingops

> This repository is maintained in an agent-first model. Codex, Claude, and Cursor must follow the same contract.

# PingOps Agent Playbook

This repository is maintained in an agent-first model. Codex, Claude, and Cursor must follow the same contract.

## Fast Start

```bash
mise exec -- pnpm install
mise exec -- pnpm dev
```

Core checks:

```bash
mise exec -- pnpm lint
mise exec -- pnpm test
mise exec -- pnpm agent:check
```

## Runtime Tooling

- Always run repository commands through `mise exec -- ...` so the configured Node.js and toolchain versions are active.
- This applies to all `pnpm`, `node`, and script commands used by agents.

## Repo Map

- `apps/web`: Next.js dashboard UI.
- `apps/api`: Hono API (REST + MCP endpoint).
- `apps/collector`: ingestion API + queue workers.
- `packages/*`: shared runtime libraries (`db`, `clickhouse`, `env`, `logger`, `queue`, `types`, `common`).
- `docs/*`: canonical architecture, plans, reliability, and security references.
- `tools/agent-lints/*`: repository-specific structural lint checks.
- `scripts/agent/*`: harness automation scripts.

## Canonical Documents

- Architecture: `docs/ARCHITECTURE.md`
- Plan workflow: `docs/PLANS.md`
- Quality index: `docs/QUALITY_SCORE.md`
- Reliability rules: `docs/RELIABILITY.md`
- Security policy: `docs/SECURITY.md`
- Frontend conventions: `docs/FRONTEND.md`
- UX/product design policy: `docs/DESIGN.md`
- Autonomy/merge policy: `docs/AUTONOMY_POLICY.md`

## Agent Workflow

1. Read this file and the nearest scoped `AGENTS.md` before coding.
2. If work is non-trivial, create or update an execution plan in `docs/exec-plans/active/` using `docs/exec-plans/TEMPLATE.md`.
3. Make small, reversible changes and keep docs synchronized.
4. Run local checks (`mise exec -- pnpm agent:check`) before finalizing.
5. Move completed plans to `docs/exec-plans/completed/` and update `docs/QUALITY_SCORE.md`.

## Browser Validation (CDP)

Use Chrome DevTools Protocol smoke checks during development for UI feature validation.

Default run:

```bash
mise exec -- pnpm ui:smoke:cdp
```

Useful overrides:

```bash
# Reuse an already running dev server
UI_BASE_URL=http://127.0.0.1:3000 WEB_SERVER_COMMAND=':' mise exec -- pnpm ui:smoke:cdp

# Use a specific Chrome binary
CHROME_BIN='/Applications/Google Chrome.app/Contents/MacOS/Google Chrome' mise exec -- pnpm ui:smoke:cdp
```

Artifacts are written to `out/agent-artifacts/` and should be referenced in execution plans when relevant.

## Guardrails

- Never commit secrets, tokens, or raw credentials.
- Prefer explicit types and schema validation at process boundaries.
- Database migrations, auth changes, billing logic, and workflow permission changes require human approval.
- Safe categories may auto-merge only when all required checks pass.

## Definition Of Done

A task is complete only when code, tests, docs, and policy checks are aligned. If a check is intentionally deferred, record the reason in the active exec plan.

---
> Source: [pingops-io/pingops](https://github.com/pingops-io/pingops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
