## arena

> This file is the entry point for any agent (Claude Code, Cursor, automated reviewer) working in this repo. It is loaded into context on every session (`CLAUDE.md` `@import`s it), so keep it short and current.

# AGENTS — ARENA

This file is the entry point for any agent (Claude Code, Cursor, automated reviewer) working in this repo. It is loaded into context on every session (`CLAUDE.md` `@import`s it), so keep it short and current.

> **Keep these docs current.** When you make a big change to the architecture or
> code — a new top-level component, a renamed/retired pipeline stage, a changed
> tech stack, a moved directory, a new convention — update `AGENTS.md` and the
> relevant file under `agent_docs/` **in the same change**. Stale agent docs cost
> every future agent. Treat a doc that contradicts the code as a bug to fix, not
> a thing to route around. If you notice drift while working on something else,
> fix it or flag it; don't leave it.

## Mental model

ARENA is an AI Pentest Bench: it generates fully custom CTF security challenges on
the fly with AI, wraps each in a Docker container, and serves them through a web
platform. There is no single `platform/` root — code lives in several top-level
areas:

| Top-level dir | What it is |
| ------------- | ---------- |
| `platform/`   | The web platform — a pnpm/Nx monorepo. **pnpm/Nx workspace root is `platform/`, not the repo root.** Contains `api/` (Fastify + Drizzle + Postgres), `web/` (Vite + React 19 + Tailwind v4 + React Router 7), `keycloak/` (theme + import for local auth), and `docker-compose.yml`. |
| `spawner/`    | The lab generator — a Python (uv) Claude Agent SDK pipeline that turns a spec into a runnable benchmark lab + image. Run with `python -m arena spawn --local <spec.yaml>` (bare/`spawn` defaults to a remote platform spawn). |
| `lambda/`     | AWS Lambda handlers for the cloud generation pipeline (Step Functions): `dispatch`, `pipeline_start/upload/wait`, `terraform`, `validate`, `score`. |
| `terraform/` + `modules/` + `bootstrap/` + `keycloak/` | Infrastructure as code (Terraform/OpenTofu). `terraform/` is the root deployment; `modules/` holds `platform`, `labs`, `labs_tier`, `cost_monitoring`; `bootstrap/` sets up the state backend; `keycloak/` is the identity deployment. |

`agent_docs/` (repo root) holds all agent guidance so it's easy to find before
opening any code.

## Conventions

Read these — they are enforced, not suggestions:

| Doc | What it covers |
| --- | -------------- |
| [`agent_docs/code_review_rules.md`](./agent_docs/code_review_rules.md) | Hard rules the reviewer agent blocks PRs on |
| [`agent_docs/typescript_conventions.md`](./agent_docs/typescript_conventions.md) | Zod-first, no `as` / `any` / `!`, error policy |
| [`agent_docs/frontend_conventions.md`](./agent_docs/frontend_conventions.md) | React/Tailwind rules, no speculative memo |
| [`agent_docs/ui_components.md`](./agent_docs/ui_components.md) | Typography + Card primitives |
| [`agent_docs/python_conventions.md`](./agent_docs/python_conventions.md) | Pydantic-first, uv, error handling |
| [`agent_docs/terraform_conventions.md`](./agent_docs/terraform_conventions.md) | Terraform/OpenTofu module + naming rules |
| [`agent_docs/dockerfile_conventions.md`](./agent_docs/dockerfile_conventions.md) | Multi-stage cache ordering, build-fat/ship-thin, layer hygiene |
| [`agent_docs/prompt_engineering.md`](./agent_docs/prompt_engineering.md) | LLM prompt structure, tool contracts, few-shot for the spawner pipeline |

Spawner & ops references (read when working on the generator):

| Doc | What it covers |
| --- | -------------- |
| [`agent_docs/spawner_architecture.md`](./agent_docs/spawner_architecture.md) | The spawn pipeline: planner → skeleton → feature loop (with deferred per-feature validation) → sanity → image |
| [`agent_docs/spawner_skill_authoring.md`](./agent_docs/spawner_skill_authoring.md) | Adding/editing the `arena-spawner` plugin skills |
| [`agent_docs/benchmark_spec_format.md`](./agent_docs/benchmark_spec_format.md) | The spec/`SpecDocument` format the spawner consumes |
| [`agent_docs/spec_validity.md`](./agent_docs/spec_validity.md) | Cross-axis "what goes with what" rules: capability resource facts + central policy |
| [`agent_docs/environments.md`](./agent_docs/environments.md) | Local vs prod: AWS account, Logfire, images |

Operational skills (`.claude/skills/`): **`run-arena-lab`** to spawn a lab and bring up
its container, **`logfire`** to query and triage spawn traces.

Highlights:

- **Capability tree + lab sizing are spawner-owned, loaded at runtime.** `spawner/capabilities/**` (the exploit/defense/identity/scale menu) and `spawner/config/lab.yaml` (size tiers → feature/endpoint bands) are the single source of truth. The API reads them at boot and serves them at `GET /api/v1/catalog`; the web builds the lab catalog from that. No generated copy — there is no `*.gen.ts`.
- **Comments**: almost none. Only when the *why* is non-obvious. Don't narrate what code does.
- **Errors**: catch specific named classes only. Never bare `catch (e) {}`. Log + rethrow.
- **Tests**: Playwright E2E for UI; real unit/integration tests for backend persistence.
- **Avoid**: premature abstractions, speculative `useMemo`/`useCallback`, `as`/`any`/`!`, defensive null checks for impossible cases.
- **Names matter.** The reviewer has authority to bounce a PR on naming alone.

## Local dev

See [`README.md`](./README.md) (platform) and [`spawner/README.md`](./spawner/README.md) (generator). Short version:

```bash
cd platform && cp .env.example .env && pnpm install && docker compose up -d
```

---
> Source: [arena-labs-ai/arena](https://github.com/arena-labs-ai/arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
