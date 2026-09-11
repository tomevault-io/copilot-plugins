## dsh-memory

> `dsh-memory` is a local-first memory plugin for DeepSeek Harness. It stores authoritative memory as readable Markdown, exposes only the current Workspace to an Agent, and treats historical DSH Sessions as evidence for later consolidation.

# AGENTS.md

`dsh-memory` is a local-first memory plugin for DeepSeek Harness. It stores authoritative memory as readable Markdown, exposes only the current Workspace to an Agent, and treats historical DSH Sessions as evidence for later consolidation.

## Start here

Before changing behavior, read:

1. [README.md](README.md) for the shipped surface and installation;
2. [docs/architecture.md](docs/architecture.md) for the implemented components and request flows;
3. [docs/design.md](docs/design.md) for stable product decisions;
4. [docs/roadmap.md](docs/roadmap.md) for the next milestone;
5. [docs/development.md](docs/development.md) for verification and release commands.

Read the relevant record under `docs/decisions/implemented/` before changing an established architectural choice.

## Product boundaries

- Keep the product Session-native, local-first, and usable without an external memory provider.
- Markdown files under `$DSH_HOME/memory` are authoritative. Indexes, UI views, receipts, and future access statistics are derived data.
- Persistent scopes are `global` and `workspace`. Do not add Agent, Preset, Session, provider, or cross-Workspace scopes without an explicit design change.
- Global memory contains durable user profile, preferences, and genuinely cross-project guidance. Project facts and feedback belong to the current Workspace.
- Workspace recall uses progressive disclosure: inject `MEMORY.md`, then let the Agent open only relevant detailed files with existing file tools.
- Do not add a model-facing `memory_recall` tool. Reconsider this only if ordinary file search proves insufficient and evidence is recorded.
- Online writes and future consolidation must use `MemoryStore`; Consumers must not write `$DSH_HOME/memory` directly.
- The model may propose memory changes. Deterministic Host code owns scope resolution, validation, revision checks, and commits. Session consolidation must minimize and redact model-facing evidence before commit.
- Model-visible memory must be reconstructable from the DSH Session log.
- v0.1 does not expose or search other Workspaces from an Agent turn.

## Repository layout

```text
src/index.ts                 Host Cordis entry and Consumers
src/memory-store.ts          Markdown storage, validation, revisions, atomic commits
src/protocol.ts              Host/Browser wire types and runtime parsers
src/client/                  Browser Cordis entry and Settings UI
tests/                       Unit and assembled plugin tests
docs/architecture.md         Implemented structure and request flows
docs/design.md               Stable product rules and deliberate exclusions
docs/roadmap.md              Ordered implementation milestones
docs/development.md          Commands, verification, and release discipline
docs/decisions/implemented/  Durable non-trivial decisions
cordis.patch.yml             Bundle layer that inserts the Host entry
```

The repository root is also the npm package root. Host and Browser run in separate Cordis trees and communicate only through validated Connection RPC values.

## Commands

```sh
pnpm install
pnpm run typecheck
pnpm run test
pnpm run build
pnpm run check
npm pack --dry-run
```

Use `pnpm run check` before handing off a code change. Run the narrower command while iterating, then the full local check once. Do not publish, push, create a remote, or change npm ownership unless the user explicitly requests it.

## Implementation rules

- TypeScript is strict ESM. Keep exported contracts typed and avoid `any`.
- Every Cordis registration is a reversible effect. Use `ctx.effect()`, `ctx.on()`, or a registration API whose disposer is owned by an effect.
- Declare every service read through `ctx.get()` in Cordis `inject`; optional Web-only capabilities belong in optional child fibers.
- Keep Host, model Tool, Session worker, and Browser UI as Consumers of `MemoryStore`.
- Treat the Host/Browser wire as hostile input: parse request and response values at runtime on both sides.
- Browser RPC stays loopback-only. The Browser must never submit a memory root, absolute path, Workspace key, or cwd.
- Workspace mutation scope comes from the executing Agent Session. Browser Workspace selection must resolve an opaque id through a Host registry.
- Use revision-checked writes. A mismatch returns conflict; never silently overwrite or automatically rerun a model.
- Workspace updates publish a complete valid generation and rebuild `MEMORY.md`. Do not expose partial files or a stale index.
- Keep Workspace frontmatter limited to `name`, `description`, and `metadata.type` until a real consumer requires another field.
- Do not silently truncate memory or Session evidence. Fail explicitly when an external model limit prevents processing.
- Generated `lib/` artifacts are build output; do not edit them by hand.

## Tests and evidence

- Storage changes: cover parsing, invalid frontmatter, revision conflicts, atomic replacement, and deterministic index generation.
- Protocol changes: test valid and invalid request and response values.
- Plugin wiring changes: prove Headless can load without Web services and Web registers the expected Host/Client surfaces.
- Model-visible behavior: add an assembled DSH snapshot or replay test when the harness supports the changed path; unit tests alone do not prove Session reconstructability.
- UI changes: build the client and verify the real installed page. Add a screenshot or GIF for user-visible behavior before release.
- Installation changes: install the built local package into an isolated Web profile, check `--dump-config`, clean boot logs, `__DSH_BOOT__`, the client bundle response, and one representative RPC.

Report only checks actually run.

## Documentation and decisions

- Keep each fact in one home. Link instead of copying large blocks between README, architecture, design, roadmap, and development docs.
- Update README status and `docs/architecture.md` when behavior ships.
- Update `docs/roadmap.md` when a milestone starts, completes, or changes order.
- Change `docs/design.md` only for durable product decisions, not implementation narration.
- Add `docs/decisions/implemented/<kind>/<date>-<slug>.md` for non-trivial architecture, behavior, security, persistence, or protocol choices. Use `Problem`, `Decision`, `Alternatives considered`, and `Consequences`.
- Keep prose current-state and concise. Do not preserve review history or reasoning transcripts in code comments.

## Current priority

Finish M3 release evidence before expanding behavior: add the real assembled worker replay test and the release UI GIF. Manual Session consolidation, model selection, diagnostics, receipts, recovery, and Workspace management are implemented and have passed real Web use. M4 evaluation and any automatic scheduling remain ordered in [docs/roadmap.md](docs/roadmap.md).

---
> Source: [hr98w/dsh-memory](https://github.com/hr98w/dsh-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
