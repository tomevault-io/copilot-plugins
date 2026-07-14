## keystroke

> Keystroke is the code-first AI automation platform — agents, workflows, and triggers as real TypeScript your coding agent can build, test, and maintain. Built on Hono.

# keystroke — coding agent guide

Keystroke is the code-first AI automation platform — agents, workflows, and triggers as real TypeScript your coding agent can build, test, and maintain. Built on Hono.

This is a **pnpm + Turborepo monorepo** (`@keystrokehq/*`). Public framework
code must not import `@keystroke-cloud/*` packages or paths with an exact
`cloud` segment; oxlint enforces that boundary.

Tour and quickstart: [README.md](./README.md). **Read READMEs and source before
implementing** — docs are hints; code, tests, and call sites are truth.

## Principles

Optimize for the smallest correct change. Every line should earn its place.

- **YAGNI** — No speculative APIs, feature flags, or edge-case handling that doesn't exist today.
- **DRY** — Reuse existing utils, patterns, and schemas. Shared wire shapes → `@keystrokehq/shared`.
- **Type safety** — Strict TS, Zod at boundaries, no `any`. Functional, modular style; ESM; `@keystrokehq/` scope.
- **TDD** — Write or update a failing test first for non-trivial logic; keep unit tests fast and integration tests behind `describeIntegration`.
- **Simplicity** — Boring over clever. Inline until a second real call site exists. No premature optimization.
- **Edge cases** — Think them through at trust boundaries; don't build every hypothetical. Mark intentional shortcuts with `ponytail:` comments (ceiling + upgrade path).
- **Research depth** — Search the repo, read the package README, trace call sites, then implement. Update READMEs when your change makes them wrong.

## Architecture

| Component | Role | Read |
| --- | --- | --- |
| **Primitives** | `action`, `agent`, `workflow`, `trigger`, `sandbox` | Per-package READMEs under `packages/` |
| **Framework** | HTTP, runtime, worker, build, CLI, discovery | `packages/http`, `runtime`, `worker`, `build` |
| **Web** | TanStack Start dashboard; SDK client only | [apps/web/README.md](./apps/web/README.md) |
| **Platform** | Orgs, auth, deploys, credentials | `apps/platform/` |
| **Project template** | Scaffold produced by `keystroke init` | `apps/cli/templates/hello-world/` |
| **Product e2e** | Cross-stack tests | [tests/README.md](./tests/README.md) |

Cross-layer wire shapes live once in `@keystrokehq/shared` (Zod schemas, types,
constants). Server and platform handlers implement those shapes; consumers use
`@keystrokehq/sdk`. Do not import database or runtime packages from web or CLI
for wire data.

Classify every change: **primitive · runtime glue · framework convention · plugin**.

HTTP/framework code belongs in `@keystrokehq/http`, `@keystrokehq/runtime`,
`apps/cli`, or plugins — not primitive packages unless it is runtime-agnostic.

## Authoring invariants

- **Action** — `defineAction`; leaf unit; compose in workflows; may call an agent.
- **Workflow** — `defineWorkflow`; global `slug`; `POST /workflows/{slug}` or use as an agent tool.
- **Trigger** — source + `.attach({ workflow, transform? })`; filters on the source; attachment id `{sourceSlug}:{targetSlug}`.
- **Sandbox** — `defineAgent({ sandbox })`; configure VM mode on `defineSandbox`, not as a top-level agent field.
- **Auth** — `@keystrokehq/auth` plus server/apps only.
- **Types** — cross-layer shapes live in `@keystrokehq/shared`.
- **CLI** — `keystroke <plural-collection> <verb> [<slug>] [--flags]`; primary resources are positional.

Full CLI grammar: [apps/cli/README.md](./apps/cli/README.md).

## Toolchain

| Tool | Role |
| --- | --- |
| **oxfmt** | Formatting (semicolons, single quotes, trailing commas, width 100) |
| **oxlint** | Lint and package-boundary enforcement |
| **tsc** | Strict typechecking |
| **Vitest** | Unit and integration tests |
| **Turborepo** | Workspace task orchestration |

Always run commands from the repository root. Filter by the package `name` from
its `package.json` or by directory.

| Task | Command |
| --- | --- |
| Format | `pnpm format` / `pnpm format:check` |
| Affected gate | `pnpm check` |
| Scoped gate | `pnpm exec turbo run build lint typecheck test:unit --filter=<package>` |
| Unit tests | `pnpm test:unit` |
| Integration tests | `pnpm test:integration` |
| Integrations catalog | `pnpm check:integrations:affected` |
| Build | `pnpm build` |

Before finishing TypeScript changes:

```bash
pnpm format
pnpm --filter <package> lint
pnpm exec turbo run typecheck test:unit --filter=<package>
```

Tests use `*.test.ts` for unit coverage and `*.int.test.ts` for integration
coverage. Integration tests skip when required provider credentials are absent.

Dependencies used by at least two packages belong in the root pnpm catalog;
otherwise pin them in the one consuming package.

## Where to read

| Topic | Doc |
| --- | --- |
| Web dashboard | [apps/web/README.md](./apps/web/README.md) |
| Agent primitives | [packages/agent/README.md](./packages/agent/README.md) |
| Database layering | [packages/database/README.md](./packages/database/README.md) |
| Access control | [packages/access-control/README.md](./packages/access-control/README.md) |
| Shared schemas | [packages/shared/README.md](./packages/shared/README.md) |
| SDK | [packages/sdk/README.md](./packages/sdk/README.md) |
| Docs site | [docs/README.md](./docs/README.md) |
| Product e2e | [tests/README.md](./tests/README.md) |
| Project template | [apps/cli/templates/hello-world/README.md](./apps/cli/templates/hello-world/README.md) |

---
> Source: [keystrokehq/keystroke](https://github.com/keystrokehq/keystroke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
