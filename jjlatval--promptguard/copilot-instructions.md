## promptguard

> You are working in the PromptGuard monorepo (pnpm + TypeScript). Follow these rules for all future changes.

You are working in the PromptGuard monorepo (pnpm + TypeScript). Follow these rules for all future changes.

## 0) Non‑negotiable security principles
- Open world for text; closed world for execution.
- Untrusted text must never directly drive tool execution.
- The ONLY executable authority is a validated Action AST (JSON) that conforms to:
  - allowlisted Action DSL schema (Ajv validation against `packages/core/schemas/action.json`)
  - deterministic normalization (e.g., COMMIT risk forced to HIGH)
  - policy evaluation (fail-closed)
- Fail‑closed: if anything is invalid/unknown/missing → DENY (or NEEDS_APPROVAL where appropriate), never ALLOW.
- High-risk actions require explicit approval by default; never silently auto-commit.
- Never add new “escape hatches” like dynamic code execution, eval, shell, arbitrary URL fetchers, etc.

## 1) Repository layout and boundaries
- This is a pnpm workspace. Keep package boundaries clean:
  - `packages/core`: schemas + minimal validator + core types
  - `packages/policy`: policy loading/evaluation (no network calls)
  - `packages/gateway`: HTTP API orchestration, adapters, auth, audit logging
  - `packages/n8n-nodes-promptguard`: thin n8n client only
  - `packages/redteam`: regression runner (no production deps)
  - `packages/examples`: example payloads/policies/workflows (no runtime code)

## 2) TypeScript conventions (strict)
- Keep TypeScript strict and compile-clean (`pnpm -r build`).
- Prefer explicit types at API boundaries (Fastify handlers, policy evaluator inputs/outputs).
- Respect `exactOptionalPropertyTypes`; avoid passing `undefined` as an optional field:
  - use conditional spreads: `...(x ? { field: x } : {})`
- Avoid `any`. If unavoidable, confine to parsing/untyped IO and immediately validate.
- Do not commit compiled artifacts accidentally into source trees.

## 3) Action DSL + schema rules (core)
- Any change to Action types MUST remain consistent with `packages/core/schemas/action.json`.
- Validation must use Ajv draft 2020-12, and be deterministic.
- Any new fields must be added to the JSON schema and validated.
- Treat schema changes as breaking unless proven otherwise; add tests for schema acceptance/rejection.

## 4) Policy rules (policy)
- Policy evaluation must be deterministic and side-effect-free.
- Default posture is conservative:
  - unknown role/action/connector data ⇒ DENY
  - COMMIT defaults ⇒ NEEDS_APPROVAL unless explicitly allowed
- Always return structured reasons (stable `code`, human `message`).
- When adding new policy predicates, add unit tests for both allow and deny paths.

## 5) Gateway API rules
- Gateway should NEVER “execute” anything. It plans, validates, distills, linearizes.
- `/v1/validate` pipeline must be:
  1) schema validate (Ajv)
  2) normalize action (risk, derived fields if any)
  3) policy evaluate
  4) respond with decision + reasons + normalizedAction
- Keep responses stable and backwards-compatible where possible.
- If changing an endpoint contract, update:
  - OpenAPI/spec files under `packages/gateway` or `packages/examples/openapi`
  - example payloads in `packages/examples/payloads`
  - any tests / redteam cases that depend on it

## 6) n8n community node constraints (CRITICAL)
- `packages/n8n-nodes-promptguard` must remain “thin”:
  - NO runtime dependencies (`dependencies` must be absent or `{}`)
  - Use ONLY n8n APIs at runtime (e.g., `this.helpers.request(...)` for HTTP)
  - Do not read local filesystem or environment variables at runtime
  - Secrets must live in n8n Credentials
- Ensure `package.json`:
  - name starts with `n8n-nodes-`
  - keywords include `n8n-community-node-package`
  - `n8n` field lists nodes + credentials in `dist/`
  - `files` includes `dist`, `README.md`, `LICENSE`
- Always keep `pnpm release:check` green when touching node package.

## 7) Testing and quality gates
- Before finishing a change, ensure these are green:
  - `pnpm -r build`
  - `pnpm -r test`
  - `pnpm release:check` (especially after node changes)
- If changing planner/validator/policy behavior, update or add:
  - unit tests (core/policy)
  - gateway integration tests
  - redteam cases (when relevant)
- Prefer small, focused tests with clear failure messages.

## 8) Release process (Changesets)
- Only publish `n8n-nodes-promptguard`.
- Use Changesets:
  - Add a changeset for user-facing node changes in PRs
  - Release workflow runs `pnpm release:check` before publishing
- Keep internal packages private; do not make them publishable.

## 9) Documentation expectations
- When changing developer workflows, update the root `README.md`.
- When changing release mechanics, update `RELEASING.md` and `docs/RELEASE_CHECK.md`.
- Keep example curl commands consistent with the gateway’s default port (currently 8080).

## 10) Practical coding style
- Keep implementations minimal and production-minded; avoid overengineering.
- Prefer deterministic behavior in demo/default modes; no hidden LLM requirements.
- Add TODO markers for future integrations (GF/LLM) but don’t block current functionality.

---
> Source: [jjlatval/promptguard](https://github.com/jjlatval/promptguard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
