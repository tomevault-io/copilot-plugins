## uptimer

> Guidelines for code assistants and automation agents working in this repository.

# AGENTS.md: Working Agreement for Uptimer

Guidelines for code assistants and automation agents working in this repository.

---

## 1. Required Reading

Before making any changes (code, config, or destructive commands), read and understand:

- `AGENTS.md` (this file)
- `Develop/Application.md` (product specification & technical constraints)
- `Develop/Structure.md` (directory structure & module boundaries)
- `Develop/Plan.md` (milestones & acceptance criteria)
- `Develop/API-Reference.md` (Cloudflare / D1 / outbound API reference)

If documentation conflicts with the current task, stop and align before proceeding.

---

## 2. Technology Stack (Locked)

Do not introduce alternative technologies without explicit approval.

- **Frontend (Pages)**: React + Vite + TypeScript + Tailwind + React Router + TanStack Query + Recharts
- **Backend (Workers)**: TypeScript + Hono + Zod
- **Database**: Cloudflare D1 + Drizzle ORM; migrations via Wrangler D1 (SQL)
- **Auth**: Admin Bearer Token (stored in Workers Secret)

Any new dependency or service (Queues, DO, R2, etc.) requires a written justification covering: why it's necessary, why alternatives don't work, and the impact scope.

---

## 3. Repository Rules

- **Do not modify** `UptimeFlare/` (reference project, read-only).
- All external APIs must follow `Develop/Application.md` conventions (paths, time fields, error format).
- All input must be validated with Zod at runtime — never trust client or DB JSON fields.
- All DB writes must use parameterized queries (Drizzle or D1 prepared statements). No SQL concatenation.
- HTTP monitoring probes must explicitly disable caching (`no-store` + `cf.cacheTtlByStatus`).

---

## 4. Implementation Priority

Follow `Develop/Plan.md` strictly from Phase 0 through Phase 7:

- Worker + D1 must be functional (including scheduled triggers) before building full UI.
- HTTP/TCP monitoring and state machine correctness come before multi-region or advanced analytics.

---

## 5. Definition of Done

Every change must:

- Be small and focused (no "big bang" changes).
- Pass local checks (if scripts exist):
  - `pnpm lint`
  - `pnpm typecheck`
  - `pnpm test` (if established)
- Include a new migration for any D1 schema change (never modify existing migrations).
- Include minimal tests or reproducible steps for behavioral changes.

---

## 6. Security Baseline

- Monitor targets are controlled SSRF: restrict protocols, deny private/reserved IP ranges by default. Port range 1-65535 is allowed. See `Develop/Application.md` for specifics.
- Admin Token goes only in Workers Secrets or `.dev.vars` (local). Never in Git, D1, or frontend code.
- Webhook signing secrets must reference Worker secrets — never store in the database.

---

## 7. Change Description Format

When reporting changes, include:

- **What** you did (1-3 lines)
- **Why** (key constraints / risks)
- **Where** (affected file paths / modules)
- **How to verify** (commands or steps)

---
> Source: [VrianCao/Uptimer](https://github.com/VrianCao/Uptimer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
