## serverless-agent

> - **Before every `git push` or PR merge, run `/code-review` first.** No exceptions. Fix violations before pushing.

## General Approach

- **Before every `git push` or PR merge, run `/code-review` first.** No exceptions. Fix violations before pushing.
- For every bug or fix you make, consider whether a coding guideline would prevent the same issue in the future. After the task is done, ask the user if they want to `/update-code-review-guideline`.
- **When the user asks to merge a PR or push to main**, retrospectively review what you learned during the task and run `/update-code-review-guideline` to capture any new patterns, pitfalls, or conventions worth enforcing.

## Critical Rules

- **NEVER run `tsc` or `npx tsc` directly.** Not for any package. Not ever.
- **For type checking:** `./packages/backend/scripts/build-types.ts` or `./packages/frontend/scripts/build-types.ts`. Run from repo root.
- **All scripts are executable via shebang** — run them directly, not via `npm run`. No exceptions.
- **After pulling/merging remote changes**, always run `npm install` from root before doing anything else. If a package appears missing, it's because you forgot to run `npm install` — do NOT add it as a new dependency.
- **After ANY change to a skill factory** (params, returns, entities, method rename/add/remove under `packages/backend/src/agent-runtime/skill-runtimes/*.ts`), run `./packages/backend/scripts/generate-declarations.ts`. The generated `.d.ts` files are what the LLM sees as the skill's API — typecheck alone will not catch a stale declaration, but the agent will call the wrong shape at runtime. No exceptions.

## Pre-push CI parity — run the full CI suite locally before every `git push`

Waiting on a self-hosted CI runner to tell you what a local script would have told you in 20 seconds is pure CPU waste. Before **every** `git push` (feature branch, follow-up fix, whatever), run both CI jobs verbatim. Fix any failure, then push.

The two jobs in `.github/workflows/deploy.yml` are:

```bash
# Backend job — typecheck + lint + tests
./packages/backend/scripts/build-types.ts \
  && ./packages/backend/scripts/lint.ts \
  && npm test -w backend

# Frontend job — backend types (for cross-package refs) + frontend typecheck + lint + vitest
./packages/backend/scripts/build-types.ts \
  && ./packages/frontend/scripts/build-types.ts \
  && ./packages/frontend/scripts/lint.ts \
  && npm test -w frontend
```

Things a pre-commit hook does **not** catch but CI does:
- Widening a discriminated union in one package breaks an exhaustive switch in the other package — only the *cross-package* typecheck (frontend typechecks while importing backend types) surfaces it.
- Test failures (hooks skip tests for speed).

Rule: **don't push until both jobs are green locally**. If it passes locally and fails on CI, that's a real bug.

## Storage

- All persistent state lives in **DynamoDB**. The CDK stack (`packages/backend/scripts/lib/backend-stack.ts`) declares every table. Attribute names mirror the TS row shapes in `packages/backend/src/types/database.ts` — snake_case keys, ISO-string timestamps.
- Access goes through the DocumentClient singleton in `packages/backend/src/lib/ddb.ts` + typed repos under `packages/backend/src/<domain>/*-repository.ts`.
- Auth is **Google OAuth only** (`packages/backend/src/auth/`, `packages/backend/src/lib/google-oauth.ts`). User identity in the `users` table is a uuid; provider identity (Google `sub`, email) lives in the separate `accounts` table. TTL-scoped `sessions` table is unchanged. Session cookie is `sa_session`, HTTP-only, set by `/api/auth/google/callback`.
- Every user starts with **10 credits** (`users.credits`). One credit is deducted per user-authored chat message. At 0, sends are rejected with "You ran out of credit".

## Demo constraints

- Google OAuth is the only sign-in path. No username/password, no email magic links, no other providers. The agent has two skills: `memory` and `web-search` (Tavily).
- No radar feature.

---
> Source: [breath103/serverless-agent](https://github.com/breath103/serverless-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
