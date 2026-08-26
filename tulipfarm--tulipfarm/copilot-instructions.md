## tulipfarm

> Root entry point for AI coding agents. Read this, then the nearest `AGENTS.md` to your task —

# AGENTS.md

Root entry point for AI coding agents. Read this, then the nearest `AGENTS.md` to your task —
not the whole repo.

> **Terminology is binding.** Use the canonical names in
> [`metadata/terminologies.md`](metadata/terminologies.md) at every layer (code, DB, REST, URL,
> UI, docs). That file also lists banned synonyms and the rename backlog.

## What this is

TulipFarm is a **self-hosted control panel where autonomous agents run a business's
operations**. The user describes what they want in chat — "track our customers", "create a
support agent", "review this pull request" — and agents build and run it. Users never edit
files or write code to configure it. It runs on the operator's own infrastructure and model
provider keys; business data leaves the instance only when an agent was authorized to send it.

**That product promise is a constraint on you**: if a capability cannot be reached from chat or
the UI, it does not exist for users. Never close a gap by hand-editing the runtime `soul/` repo.

An instance has two halves:

- **The soul** — a git-backed config store (`soul/`, a separate repo, not a workspace). Resource
  schemas, agents, skills, routines and integrations live there as files. Agents write to it when
  asked to build something, so its git history is the audit trail.
- **The runtime** — the API and workers that load the soul, store records, index knowledge, and
  execute agent turns against configured LLM providers.

### The nouns you will meet in code

Full glossary with banned synonyms: [`metadata/terminologies.md`](metadata/terminologies.md).

- **Chat** is the external word (routes, URLs, UI); **Conversation** is the internal entity
  (table, repo, domain). Never let them bleed. A Conversation holds **Turns**, which hold
  **Messages**.
- **Agent** — a configured persona with its own instructions, tools and bounded authority.
- **Resource type** — a user-defined schema (Ticket, Customer); one instance is a **Record**.
  Never call an instance a "resource".
- **Routine** — a scheduled or triggered automation, built from **States**. One execution is a
  **Run**, which emits ordered **Run events**, each with an **Audience** (participant vs operator).
- **Integration** — a connected third party, fully defined by a declarative **manifest**:
  **egress** (what agents may do to the provider) and **ingress** (what the provider may send).
- **Skill** — an installable capability package. **Knowledge** — cited, ACL-preserving
  retrieval. **Memory** — scoped, versioned assertions. **Tool** — a callable an agent
  invokes, brokered with approvals. **Surface** — the channel-neutral protocol for
  rendering agent output.

### Stack

pnpm + Turborepo monorepo, TypeScript throughout.

- **Node** `26.5.0` (`.node-version`) · **pnpm** `11.5.3` — never npm/yarn
- **Workspaces**: `apps/*`, `packages/*`
- PostgreSQL (pgvector + pg-boss), Fastify API, Remix web UI

## Navigating this repo

Every app and package owns an `AGENTS.md` that states what it is, when to read it, and its
directory map. **Use them instead of grepping the repo.** `CLAUDE.md` files are pointers to the
sibling `AGENTS.md`.

| Path | Read it when your task touches |
| --- | --- |
| [`apps/api`](apps/api/AGENTS.md) | HTTP routes, auth/sessions, migrations, OpenAPI, soul git store |
| [`apps/web`](apps/web/AGENTS.md) | Remix UI, routes, loaders/actions, product screens |
| [`apps/worker`](apps/worker/AGENTS.md) | Run dispatch, Agent/Tool States, timers, reconciliation, projections |
| [`apps/integration-worker`](apps/integration-worker/AGENTS.md) | Integration ingress, sync, delivery, retries |
| [`apps/docs`](apps/docs/AGENTS.md) | Public Fumadocs site content and conventions |
| [`apps/eval`](apps/eval/AGENTS.md) | Offline eval Corpus, Expectations, Sweeps, Scorecards, red team |
| [`packages/agent-runtime`](packages/agent-runtime/AGENTS.md) | Context assembly, bounded Tool loop, model profiles, delegation |
| [`packages/run-kernel`](packages/run-kernel/AGENTS.md) | Run/State machines, waits, retries, child Runs |
| [`packages/files`](packages/files/AGENTS.md) | Upload limits, the type allowlist, magic-byte sniffing, the `files` table |
| [`packages/curator`](packages/curator/AGENTS.md) | Curator prompt, output schema, citation and injection validation, proposal templating |
| [`packages/curator-host`](packages/curator-host/AGENTS.md) | Minting a Curator job and its Run, context pinning, output revalidation, crash recovery |
| [`packages/model-adapter`](packages/model-adapter/AGENTS.md) | Translating `ModelPort` requests, tool calls and usage to and from the AI SDK |
| [`packages/turn-executor`](packages/turn-executor/AGENTS.md) | Chat Turn execution, Agent States, Turn guardrails, Run events |
| [`packages/tool-broker`](packages/tool-broker/AGENTS.md) | Tool catalog, intent/effect orchestration, approvals |
| [`packages/tool-host`](packages/tool-host/AGENTS.md) | Tool contract, authorization gate, dispatcher, co-location rule |
| [`packages/kv`](packages/kv/AGENTS.md) | Agent key-value store and its `kv_*` Tool family |
| [`packages/platform-tools`](packages/platform-tools/AGENTS.md) | Platform Tools that need no Soul, renderer or credential, so both hosts can run them |
| [`packages/schema`](packages/schema/AGENTS.md) | Any config shape, TypeBox schema, validator, Run event type |
| [`packages/deploy-render`](packages/deploy-render/AGENTS.md) | Rendering deployment guidance from `deploy/` — targets, generated pages, prompt, guided flow |
| [`packages/soul`](packages/soul/AGENTS.md) | Soul artifact loading, git sync |
| [`packages/storage`](packages/storage/AGENTS.md) | PostgreSQL repositories, outbox/inbox, blob/vector/cache ports |
| [`packages/resources`](packages/resources/AGENTS.md) | Record write policy, validation, hooks, idempotency, and side-effect orchestration |
| [`packages/authz`](packages/authz/AGENTS.md) | Principals, roles, grants, authority intersection |
| [`packages/audit`](packages/audit/AGENTS.md) | Audit events, hash chaining, sealing/export, lineage |
| [`packages/knowledge`](packages/knowledge/AGENTS.md) | Source ingestion, retrieval, provenance |
| [`packages/memory`](packages/memory/AGENTS.md) | Scoped, versioned memory assertions |
| [`packages/llm`](packages/llm/AGENTS.md) | Provider abstraction, tiered fallback chains |
| [`packages/secrets`](packages/secrets/AGENTS.md) | Encrypted secret storage, key rotation |
| [`packages/integrations`](packages/integrations/AGENTS.md) | Adapter contracts, event normalization, identity mapping |
| [`packages/surface`](packages/surface/AGENTS.md) | Tulip Surface Protocol contracts, catalog, Artifacts |
| [`packages/surface-web`](packages/surface-web/AGENTS.md) · [`-slack`](packages/surface-slack/AGENTS.md) · [`-github`](packages/surface-github/AGENTS.md) | Channel-native TSP renderers |
| [`packages/sandbox`](packages/sandbox/AGENTS.md) | Isolated execution contract, backend ports |
| [`packages/observability`](packages/observability/AGENTS.md) | OTel conventions, metrics, health/readiness, redaction |
| [`packages/editor`](packages/editor/AGENTS.md) | Shared rich-text editor |
| [`packages/testkit`](packages/testkit/AGENTS.md) | Shared test fixtures and helpers |
| [`packages/constants`](packages/constants/AGENTS.md) · [`tsconfig`](packages/tsconfig/AGENTS.md) | Env-aware constants, tsconfig bases |

Not workspaces: `soul/` (separate git repo created by `scripts/setup-dev.sh` — Resources,
Routines, Agents, Skills, Integrations), `docs/architecture/` (design decisions), `metadata/`
(terminology).

### Keep `AGENTS.md` current

- After changing a directory's structure, contracts, or local conventions, update its `AGENTS.md`
  in the same change. A stale map costs every future agent more than it saved you.
- New app or package → add its `AGENTS.md` and a row in the table above.
- These files are navigation, not documentation. Follow the shape in
  [Writing an `AGENTS.md`](#writing-an-agentsmd).

### Writing an `AGENTS.md`

Budget: **≤ 40 lines** for a package, **≤ 80** for an app. Sections, in order:

1. Title + one or two lines on what it is.
2. **Read on / Skip** — the task types that do and don't need this directory.
3. **Map** — a table of the paths that matter and what each owns.
4. **Rules** — only binding, non-obvious local constraints. Anything true of the whole repo
   belongs here in the root file, not there.
5. Links to deeper docs instead of restating them.

Do not include: rationale essays, API listings that duplicate `src/index.ts`, changelogs,
aspirational plans, or anything the reader can get faster from the code.

## Comments

**Write a comment only when the code cannot say it.** Default to no comment.

- Justified: a non-obvious *why* — a workaround, a spec/protocol constraint, an ordering
  requirement, a deliberate deviation. Include the reason, not the mechanics.
- Not justified: restating the code, section banners, commented-out code, TODOs without an owner
  or issue, or a comment that exists because the name is bad. Rename instead.
- **Prefer TSDoc (`/** ... */`) over `//`** when a comment is warranted on an exported symbol, so
  editors and consumers surface it. Keep it to the contract: purpose, non-obvious params,
  `@throws`, `@example` only when the shape is genuinely surprising.
- When editing code that carries redundant comments, drop them only if you are already touching
  those lines — do not open a side quest.

## Commands

Run from repo root; Turbo fans out.

```bash
pnpm install                     # CI: pnpm install --frozen-lockfile
pnpm dev                         # api :4010, web :4000, worker :4020, integration-worker :4030
pnpm dev:api | dev:web | dev:worker | dev:integration-worker
pnpm dev:docs                    # :5000
pnpm dev:kill                    # free ports 4000/4010/4020/4030 left bound by a dead run
pnpm lint                        # biome check, turbo-cached
pnpm typecheck                   # tsc --noEmit, turbo-cached
pnpm test                        # UNCACHED, ~5min — prefer --filter
pnpm build
pnpm reset:dev                   # wipe local db + soul, re-run setup
```

For manual/browser verification, start all four with `pnpm dev`, not a subset. Chat Turn
execution runs in the Worker, not the API — API-only (`dev:api` + `dev:web`) leaves a Run stuck
mid-stream, which triggers reconnect paths that are otherwise never exercised in normal dev and
can surface unrelated latent bugs (e.g. a CORS header gap on the SSE resume route) that look
branch-specific but are not.

Single workspace: `pnpm --filter @tulipfarm/api <script>` — **the default for `test` and
`typecheck`**.

## Verifying your work

**Wait to be asked.** Do not run tests, `pnpm lint` or `pnpm typecheck` off your own bat — they
cost minutes of the author's time for a signal they did not ask for yet, and an agent that
verifies after every edit spends most of a session waiting. Make the change, say what you changed
and what you believe it needs, then stop. Run a check when the author asks for one, or when they
have said to work to completion unattended.

Two things stay yours to run unprompted, because they are sub-second and stop bad code reaching
the author at all: `pnpm exec biome check --write <changed dir>`, and a single test file you just
wrote or changed, to prove it fails before your change and passes after.

When you are asked to verify, match the check to the blast radius of the diff. Escalate through
the tiers; reach Tier 3 once, at the end. Tests are **Vitest**, colocated `*.test.ts`.

| Tier | When | Command |
| --- | --- | --- |
| 1 | after every edit | `pnpm exec biome check --write <changed dir>` |
| 2 | after a coherent unit of work | `pnpm --filter @tulipfarm/web typecheck` + `pnpm --filter @tulipfarm/web test app/components/chat` |
| 3 | once, before handing back | `pnpm lint && pnpm typecheck`, then `pnpm --filter <pkg> test` per touched workspace |

Tier 2 is where you should live almost all the time — a bare positional arg to `vitest run` is a
path filter, so widen the path before widening the workspace.

Run bare `pnpm test` **only** for genuinely repo-wide changes: a shared `packages/*` export,
`turbo.json`, `biome.json`, a tsconfig base, or a root dependency bump.

### What each check costs

| Command | Scope | Cost |
| --- | --- | --- |
| `pnpm exec biome check <dir>` | changed dir | 0.7s |
| `pnpm lint` / `pnpm typecheck` (cache hit) | 31 workspaces | 0.1s / 1.7s |
| `pnpm eval` (scripted tier, whole Corpus) | 11 Cases | 5.5s |
| `pnpm --filter @tulipfarm/web test <path>` | 13 files | 2.1s |
| `pnpm --filter @tulipfarm/web test` | 90 files | 8.0s |
| `pnpm typecheck` (cold) | 31 workspaces | 13s |
| `pnpm --filter @tulipfarm/api test` | 229 files | 71s |
| `pnpm --filter @tulipfarm/integration-worker test` | 20 files | 75s |
| `pnpm --filter @tulipfarm/worker test` | 44 files | 141s |
| `pnpm test` (root) | 30 workspaces | ~5min idle, worse under load |

`lint`/`typecheck` are turbo-cached and invalidation does not cascade to dependents. `test` is
`cache: false` in `turbo.json` and **that is correct — do not "optimise" it**: packages are
consumed straight from source, so there is no build artifact to hang a cache edge on, and caching
without one would hand you a green run that never executed. `worker`, `integration-worker` and
`api` are ~93% of total test time; if you did not touch them, running them buys nothing.

CI is not a justification for a local full run: `ci.yml` filters changed paths, skips unit tests
for markdown-only diffs, and shards the api suite in parallel.

### Before believing a failure

1. **Re-run that suite alone.** Vitest hook timeouts fire spuriously under parallel load.
2. **Check local-only env leakage.** `apps/worker/.env.local` is a gitignored symlink to the root
   env file, and `apps/worker/test/process/**` spawns a real worker that reads it. Move the
   symlink aside to reproduce CI.
3. **Only then** treat it as yours — confirm by stashing your diff and re-running.

### A `packages/*` change does not imply running every consumer's suite

The reflex after editing a shared package is to run `api` and `worker` "to be safe". That is 4-5
minutes, and for most shared-package edits it proves nothing `typecheck` did not already prove.

| What you changed in `packages/*` | What actually needs running |
| --- | --- |
| Removed or renamed an export | `pnpm typecheck` — a broken consumer is a *compile* error, not a test failure |
| Changed a type or signature | `pnpm typecheck`, plus the owning package's suite |
| Changed runtime behaviour a consumer depends on | The owning package's suite **and** that consumer's |
| Added a new export nothing calls yet | The owning package's suite |

Only the third row earns a consumer suite. Name the behaviour you changed before you run one; if
you cannot, `typecheck` is the check you wanted.

### Changing the harness means changing the eval Corpus

`apps/eval` is the pre-release gate. It exists to tell a real harness regression apart from a model
having a different day, and it can only do that for behaviour some Case actually asserts. A change
to what an Agent turn *does*, landed without a Case, is a change the gate is blind to forever.

| If your diff changes | Add or update a Case that asserts |
| --- | --- |
| Context assembly, the system prompt, what a Soul contributes | `prompt_contains` / `prompt_omits` |
| The Tool loop — ordering, limits, arguments, refusals | `tool_called`, `tool_call_count`, `tool_argument_equals` |
| Guardrails, redaction, or a refusal path | an L2 `guardrail_*` Case, plus a `corpus/red-team/` Case if it is an attack surface |
| Run lifecycle, States, Run events, Turn completion | an L3 Case (`run_status`, `state_status`, `run_event_emitted`) |
| Soul writing or loading | an L3 `journey` Case — the writer and the loader must be on opposite sides of it |
| Anything a Turn never reaches — UI, integrations, unrelated migrations | nothing |

Three rules about the Case itself:

- **It must fail before your change and pass after.** A Case added green proves nothing and costs
  seat quota forever. Check it by reverting your change, or by breaking the Case on purpose.
- **Ground every fact you assert.** `loadCorpus` refuses an `output_*` Expectation whose text the
  model was never given, because such a Case passes by invention rather than by behaviour.
- **Editing `corpus/` moves `corpusHash`, which retires every Baseline.** Say so in the PR; a
  maintainer has to re-promote against the new Corpus before the gate compares anything again.

`pnpm eval` runs the whole Corpus on the scripted tier — free, deterministic, no credential, ~6s —
and CI runs it on every non-docs PR. The real-model Sweep (`pnpm eval:matrix`) is manual, costs a
finite subscription seat, and only a maintainer with the Environment secrets can trigger it, so
never assume it ran. See [`apps/eval/README.md`](apps/eval/README.md).

### Do not

- Re-run a tier after an edit that cannot change its result (comments, markdown, a log line).
- Run `pnpm build` unless you changed build config or need `dist/` — `typecheck` is cheaper.
- Run any of the above for a documentation-only diff; run targeted doc checks instead.
- Run a consumer's suite to prove a deletion was safe. That is what `typecheck` is for.

## Lint / format — Biome

`biome.json` (Biome 2.5.9) is the single source of truth. **No ESLint, no Prettier.**

```bash
pnpm exec biome check --write .   # lint + format + organize imports
```

The `pre-commit` hook (lefthook → lint-staged) runs `biome check --write` on staged files — write
conforming code so it is a no-op.

- **Format**: 2-space indent, line width 100, double quotes, always semicolons, `es5` trailing
  commas, Biome-sorted imports (never hand-order).
- **Rules** (`recommended` only) that bite most: no `any`, no `!` non-null assertions, `const`
  over `let`, template literals over concat, `import type` for type-only imports.
- **Not** enabled but still expected of you: remove unused imports/vars you create; prefer the app
  logger over `console`.
- Must break a rule? Scope the suppression to the line, never globally:
  `// biome-ignore lint/suspicious/noExplicitAny: <reason>`.

## Conventions

- **TypeScript everywhere**, extending `@tulipfarm/tsconfig`.
- **Surgical edits** — touch only what the task needs; never reformat adjacent code.
- **No new deps** without need; check `package.json` first.
- Migrations live in `apps/api/src/pg-migrations/` and run on boot.
- Env: copy `.env.local.example` → `.env.local`; never commit secrets.

### Product testing must use product surfaces

- Create and update Soul artifacts for manual/acceptance/E2E testing **only** through agentic Chat
  or the supported UI.
- Never hand-write YAML/Markdown/config into the runtime `soul/` repo to set up a test — no shell
  redirects, patches, scripts, or copied fixtures.
- If the product path cannot create the state you need, that is a product gap, not a reason to
  bypass it.
- Automated fixtures outside the runtime `soul/` repo remain fine.

### Manual QA uses the UI, not curl

Verify features against the running dev servers through the web UI, the same path a real user
takes. `curl` against the dev API is only for the documented credential/setup flow below.
Automated tests may still call routes directly.

## API route schemas (OpenAPI)

`/api/v1/openapi.json` is generated from route schemas — a route with no `schema` is invisible.

1. Every route needs `schema` with `description`, `tags`, `body`/`params`/`querystring`, and
   `response` for every status the handler returns.
2. Shared response shapes go in `apps/api/src/auth/schemas.ts` (or a domain `schemas.ts`) and are
   imported — never inline-duplicated.
3. Protected routes need `security: [{ sessionCookie: [] }, { bearerToken: [] }]`.
4. Verify: `curl http://localhost:4010/api/v1/openapi.json | jq '.paths'`.

## Local dev credentials

Plain `pnpm dev` seeds no admin. On first boot the web app falls through to the `/setup` wizard
(`apps/web/app/routes/setup.tsx`) — create the admin there. To headless-seed instead, set
`ADMIN_EMAIL` + `ADMIN_PASSWORD` (+ `LLM_API_KEY` for the full seed) before starting the API; see
`apps/api/src/setup/bootstrap.ts`.

```bash
pnpm --filter @tulipfarm/api dev

curl -c /tmp/tulip.txt -X POST http://localhost:4010/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"<admin-email>","password":"<admin-password>"}'

curl -b /tmp/tulip.txt "http://localhost:4010/api/v1/auth/tokens?limit=2"
```

## Git

- Never `git commit` unless explicitly asked. Work on the current branch.
- Commits and PR titles follow Conventional Commits (CI-enforced):
  `type(scope): subject`, type ∈ `feat|fix|chore|docs|refactor|perf|test|build|ci|style|revert`.
  Imperative subject, no trailing period, under ~72 chars. Body explains *why*. One logical change
  per commit.
- PR description: Conventional Commits title, 1–3 summary bullets, a test-plan checklist. Scope it
  to the diff.

---
> Source: [TulipFarm/tulipfarm](https://github.com/TulipFarm/tulipfarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
