## reticle

> > Master rules for this codebase. Read this first — the non-negotiables, layout, and conventions.

# CLAUDE.md — Reticle

> Master rules for this codebase. Read this first — the non-negotiables, layout, and conventions.

## What Reticle is

Reticle is the **proof layer for AI agents** — it verifies a running web app from the inside, without screenshots. The app embeds a dev-only SDK that instruments the DOM, network, routing, console, animations, and framework state; a local bridge + MCP server exposes that as structured tools the agent uses to **look, act, observe, and assert**. See `plan/` for the full design (gitignored).

## Monorepo layout

```
packages/core          @reticlehq/core         — bottom-of-graph foundation: wire contract, constants, zod schemas (deps: zod)
packages/browser       @reticlehq/browser      — instrumentation SDK embedded in the app (DOM-side)
packages/server        @reticlehq/server       — bridge + MCP server, the `reticle` CLI (Node-side)
packages/react         @reticlehq/react        — React adapter: DOM ref -> component -> source file
packages/vite-plugin   @reticlehq/vite-plugin  — Vite integration: stamps source + auto-injects connect()
packages/babel-plugin  @reticlehq/babel-plugin — stamps data-reticle-source (source mapping, React 19)
packages/next          @reticlehq/next         — Next.js source mapping (keeps SWC) via withReticle (CJS)
packages/test          @reticlehq/test         — spec runner + matchers for CI (peer vitest)
packages/eslint-plugin @reticlehq/eslint-plugin — dev-only lint rule: state changed ⇒ signal fired
apps/bench-app         @reticlehq/bench-app    — Vite/React fixture used to dogfood Reticle
apps/api               @reticlehq/api          — Express backend exercising real-world behaviors (CJS-ish .mjs)
apps/next-smoke        @reticlehq/next-smoke   — Next.js 15 app verifying Reticle on Next
docs/                  — user-facing docs (getting-started, usage, token-efficiency, local-registry)
SKILL.md               — PUBLIC skill for users integrating Reticle into their own project (the canonical paste-URL)
plan/                  — research/design docs only, no code (ALWAYS gitignored)
```

This is **one git repo** at the root (pnpm + turbo monorepo). The TS library packages are strict TypeScript; `@reticlehq/babel-plugin`/`@reticlehq/next` are plain CJS tooling and `apps/api`/ `apps/next-smoke` are local fixtures — all excluded from the build/lint/test gates.

## Service boundaries (who owns what)

- **`@reticlehq/core` is the contract.** Any message that crosses browser ↔ bridge ↔ agent is defined there as a constant + zod schema. It sits at the bottom of the graph (deps: `zod` only); everything depends on it, it depends on nothing. Never inline a wire string in `browser` or `server` — add it to `core`.
- **`@reticlehq/browser` only touches the DOM/page.** It never imports Node APIs.
- **`@reticlehq/server` only runs in Node.** It never imports DOM APIs.
- **`@reticlehq/react` is optional enrichment.** Core must work without it.

## Non-negotiable rules

1. **Equality:** `===`/`!==` always. `eqeqeq` is an error.
2. **No `any`.** Use `unknown` + zod narrowing at boundaries. `no-explicit-any` is an error.
3. **No free strings.** Every domain/wire/UI string is a named constant.
4. **No non-null `!`.** Use optional chaining + explicit null checks.
5. **Tests first.** RED → GREEN → REFACTOR.
6. **600-line file cap.** Over it = a cohesion failure; split before adding. (A few cohesive units — a stateful class, a package's public-API barrel + bootstrap — sit naturally in the 500–600 band and don't decompose without artificial seams.)
7. **Inject the clock.** Never call `Date.now()`/`Math.random()` inside pure logic — pass them in.
8. **Scope every data access to the authenticated principal.**
9. **Design tokens are the only place design values live.**
10. **Telemetry is part of the feature, not a follow-up.** Adding a tool? Put it in `TOOLS` and, if it produces a verdict, in `VERIFICATION_TOOLS`. Adding a finding kind? Add it to core's enum and never re-list it locally. Adding a dispatch path that bypasses `runTool`? It is invisible until you give it a reporter. Telemetry fails SILENTLY — nothing throws, no test reddens, the data is just permanently gone — so the rules are enforced by `telemetry-contract.test.ts` and written down in [`docs/telemetry-contract.md`](docs/telemetry-contract.md). Read that before touching anything that emits.
11. **No internal tracking tags.** Comments, file names, directory names, and test descriptions must never contain design-doc reference codes (letter + digit patterns like `N5`, `G4`, `M8`, `P2`, `F1`, `R1`) or internal version strings (like `0.3.7`).

## Naming conventions

| Thing | Convention | Example |
| --- | --- | --- |
| Package | `@reticlehq/<kebab>` | `@reticlehq/browser` |
| File | kebab-case | `ring-buffer.ts` |
| Type / class | PascalCase | `RingBuffer`, `ReticleEvent` |
| Variable / function | camelCase | `pushEvent` |
| Constant object | PascalCase + `as const` | `EventType`, `ActionType` |
| React component file | PascalCase or `create-` prefix for creation flows | `App.tsx`, `create-session-view.tsx` |
| `useX` function | ONLY if it calls React hooks | else use `apply/build/get/handle` |

## Pre/post-coding checklist

**Before coding:** scan for existing code to reuse → identify the constants you'll need and add them first → write the failing test. **After coding:** refactor with tests green → check file < 500 lines → run `pnpm lint && pnpm typecheck && pnpm test:unit` **before committing, not chained after it** → confirm no `any`, no free strings, no `console.log`.

**Touching the tool surface, the wire contract, or an observer?** Also run `pnpm test:e2e` (boots api + bench-app + next-smoke — **~70s**, measured; it was documented as "~20 min" for months, which is why nobody ran it). The unit gate cannot see cross-package drift: a tool rename once left four e2e specs dead across a whole framework and nothing caught it, because the battery is not part of `test:unit`. `e2e-surface-drift.test.ts` now catches the name-lookup half of that in the fast gate; the rest still needs the battery.

**Touching telemetry, feedback, or anything that emits an event?** Read [`docs/telemetry-contract.md`](docs/telemetry-contract.md) first — short, and the difference between a metric that works and one that is silently absent for six months. `pnpm test:e2e` runs `telemetry-events-test`, which fires all ten event kinds against a real capture endpoint and asserts each one lands. It exists because telemetry fails SILENTLY: `daemon_stopped` was emitted fire-and-forget just before `process.exit(0)`, so the POST died every time and nothing threw, nothing failed, and no unit test could see it. Half the spec is leak checks — a telemetry mistake is silent, shipped, and about somebody else's data.

**Touching desktop — `@reticlehq/electron`, `packages/tauri`, the IPC observer, or desktop capture?** Also run `pnpm test:e2e:desktop`. It starts a real Electron main process and a **packaged** Tauri binary and drives them headless (~3 min, most of it the Rust build). The web battery boots three HTTP servers and no desktop runtime, so it is blind to all of this — which is how v2.3.0 shipped Electron and Tauri support with no automated coverage at all, and why a `no-visual-provider` lie on concurrent captures lived in code that no gate touched. `packages/tauri` is Rust and outside every JS gate; CI's `rust` / `rust-macos` jobs are the only thing that compiles it.

**Timing assertions are a bug.** Two flaky tests this build both asserted a DURATION when the invariant was a BOUND. If the property is "cost is fixed", assert the bound (output size, truncation flag) or use a generous per-test timeout — never `Date.now() - t < N`, which is a statement about the machine and fails only under parallel load, i.e. only in CI.

---
> Source: [reticlehq/reticle](https://github.com/reticlehq/reticle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
