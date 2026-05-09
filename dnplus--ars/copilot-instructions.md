## ars

> This file is read by Claude Code child agents and defines the project-level contract for automated work on this repository.

# ARS — Agent Working Agreement

This file is read by Claude Code child agents and defines the project-level contract for automated work on this repository.

---

## Project Context

**ARS (Agentic Remotion Studio)** is a Claude Code plugin + starter repo for building Remotion-based video episodes. It is structured as a registry-first card system where every content type is registered via `CardSpec` and rendered through a single dispatch path.

Key paths:
- `src/engine/` — Remotion rendering engine (cards, layouts, scenes, shared primitives)
- `src/engine/cards/registry.ts` — Runtime card registry (Vite/browser only, `import.meta.glob`)
- `src/engine/shared/card-catalog.ts` — CLI-only static card metadata (`CARD_CATALOG`, not a runtime registry)
- `src/engine/shared/types.ts` — Source of truth for `Episode`, `Step`, `ShellConfig`, `SeriesConfig`
- `src/types/studio-intent.ts` — Source of truth for Studio intent schema shared by Studio and CLI
- `src/studio/studio-intents.ts` — Node-side helpers for creating, listing, and resolving Studio intents
- `src/episodes/template/` — Template series used for smoke testing
- `cli/` — Node.js CLI (`npx ars <command>`)
- `plugin/skills/` — Claude Code slash commands (shared via the ARS plugin)

---

## Operating Principles

- **Read before editing.** Never modify a file you haven't read in the current session.
- **Prefer deletion over addition.** The right fix is usually removing the wrong thing, not adding a workaround.
- **Reuse existing patterns first.** Before introducing a new abstraction, verify no existing utility covers it.
- **Write a cleanup plan before refactoring.** For changes touching 3+ files, draft a plan first.
- **Keep diffs small and reversible.** One logical change per commit.
- **No new dependencies without explicit user approval.**
- **Verify after every change.** Run `npm run lint` after edits. TypeScript errors are blocking.

---

## Agent PR Contract

Most ARS changes are authored by coding agents. Treat every PR as an artifact that another agent and a human maintainer can audit later.

Before opening or preparing a PR:
- Confirm the exact repo is `/Users/dylan_lu/cowork-workspace/ars` or the intended ARS source checkout. Do not include dirty changes from consumer/dogfood repos such as generated episode output unless the task explicitly says so.
- Keep one logical change per PR. If the work mixes docs, runtime behavior, generated repo support, and dogfood episode output, split it or explain why it cannot be split.
- Read `CONTRIBUTING.md` and include its PR expectations in the PR body.
- For any release-sensitive surface, include a migration decision: `No migration required` or concrete `ars update` work.
- For Studio/UI changes, include screenshots, browser notes, or a concise manual verification note.
- For package-surface changes, run and summarize `npm pack --dry-run`.
- Never publish to npm, create public tags, or push release tags without explicit maintainer confirmation.

Recommended agent-authored PR body:

```markdown
## Summary
- ...

## Compatibility / Migration
- Release-sensitive surfaces touched:
- Migration decision:

## Verification
- [ ] npm run lint
- [ ] npm run build:studio
- [ ] npm test
- [ ] npx ars episode validate template/ep-demo
- [ ] npm pack --dry-run (when package/release surface changed)

## Notes
- Screenshots / Studio notes:
- Known risks:
```

---

## Card System Contract

Every card type MUST follow this contract. Violating it breaks the registry-first renderer path.

### Required structure for a core engine card
```
src/engine/cards/<type>/
  component.tsx   # React component implementing CardRenderProps<TData>
  spec.ts         # exports `cardSpec satisfies CardSpec<TData>`
  schema.ts       # (optional) Zod schema for data validation
```

### Required structure for a series-scoped custom card
```
src/episodes/<series>/cards/<type>/
  component.tsx
  spec.ts         # exports `cardSpec satisfies CardSpec<TData>`
```

### spec.ts contract
```typescript
export const cardSpec = {
  type: "my-type",          // unique, kebab-case
  title: "Human Label",
  description: "One-line description of what this card renders.",
  component: MyCardComponent,
  schema: MyZodSchema,      // optional but strongly recommended
} satisfies CardSpec<MyCardData>;
```

### Invariants
- `cardSpec.type` must be unique within series cards. A series-scoped card **may** share a `type` with an engine card to fully replace it — the series card silently overrides the engine card in that repo's registry.
- Series-scoped cards still live under `src/episodes/<series>/cards/`, but `cardSpec.type` should use a globally unique bare name unless intentionally overriding an engine card (e.g. `normal-distribution` for a new card, `markdown` to replace the built-in).
- Core engine cards (`src/engine/cards/`) must never import from `src/episodes/`.
- `card-catalog.ts` (`CARD_CATALOG` array) is CLI-only. Never import it in browser/Remotion code.
- `cards/registry.ts` (`CARD_REGISTRY` Map) is Vite/browser-only. Never import it in CLI/Node code.
- `npx ars card list` is the operational source of truth for available built-in and series-scoped cards. Do not invent portable guidance around card names that are not listed there; use `markdown`, `image`, or `mermaid`, or propose a series-scoped custom card.
- For SVG-heavy chart cards, avoid fractional text positioning. If you render SVG `<text>` labels, ticks, legends, or axis titles, especially with `textAnchor="middle"`, snap `x` / `y` to integer pixels and prefer `textRendering="geometricPrecision"` to reduce render shimmer in encoded video.

---

## Studio Intent & Workstate Contract

Studio uses `StudioIntent` as the shared channel for plan, build, review, and prepare actions. Treat pending intents as work requests from the user, not passive telemetry.

Invariants:
- `feedback.kind` is authoritative for non-prepare intents. For prepare intents, route by `target.anchorMeta.hash` first, then `feedback.kind`, then message text.
- `prepare:*` hashes belong to YouTube prepare metadata. Do not patch narration, step content, or card visuals for a prepare intent unless the message explicitly asks for that.
- Use `prepare-generate`, `prepare-select`, and `prepare-edit` for new prepare intents. Legacy `prepare-trigger` is ambiguous and must be interpreted by hash, never by label alone.
- `prepare:youtube:generate` means run `/ars:prepare-youtube` to generate candidates from the prepare context.
- `prepare:<candidateId>:select` means apply that candidate to the episode source `metadata.youtube` and mirror the prepare artifact to match.
- After successful handling, resolve intents with `npx ars studio intent resolve <id> --summary ...`; include changed files and before/after evidence when meaningful. Use clearing only for explicit skips or legacy cleanup.
- Cross-episode work must begin with `npx ars workstate switch <epId> --stage <stage>` or an equivalent skill command that performs the switch. Do not infer the active episode from an IDE-opened file, stale statusline, or unrelated pending intent.
- `npx ars workstate set --stage "<phase>:<epId>"` infers `seriesId` and `episodeId`; keep monitor stages target-bound when switching phases.

Onboarding uses the same Studio intent channel but has different routing:

- `ep-demo` is the template preview surface during `/ars:onboard`, not a production episode.
- During `onboard-walkthrough`, Studio comments should be collected or deferred; do not patch files unless the user explicitly asks.
- During `onboard-customize`, comments default to series-level template work. Route recurring visual/style requests to `series-config.ts`, `SERIES_GUIDE.md`, shared assets, or series-scoped cards under `src/episodes/<series>/cards/`.
- If the built-in card cannot express the requested recurring behavior, create a series-scoped card override with the same `cardSpec.type` instead of patching only `ep-demo.ts`.
- Treat comments as demo-local only when the user clearly says the change is temporary or only for the current page.
- Studio's in-app status strip can report frontend polling and onboard Studio connectivity, but it is not proof that an external `npx ars studio intent watch` process or Claude Code Monitor is alive.

---

## Lint Exemptions (Do Not "Fix" These)

These patterns are intentional. Do not modify them:

| Location | Issue | Reason |
|----------|-------|--------|
| `src/engine/studio/StudioApp.tsx:419-424` | `no-irregular-whitespace` — CJK ideographic spaces `　` | Used for visual column alignment in a UI table; changing breaks the layout |
| `src/adapters/tts/noop.ts:7` | `_text`, `_options` unused vars | Interface conformance; underscore prefix is the convention |
| `src/Root.tsx:27,34,41,91` | `no-explicit-any` | `require.context` is a Webpack runtime API with no TypeScript types |
| `src/engine/vite-studio-base.ts:45-46,95-96,195-196` | `no-explicit-any` | Vite server plugin types are intentionally `any` at the boundary |
| `src/engine/components/cards/ThumbnailCard.tsx:326` | `warn-native-media-tag` | Used in a Remotion `Still` (not `Sequence`), so native `<img>` is correct |
| `src/engine/layouts/ShortsLayout.tsx:32` | `backgroundPreset` unused | Reserved prop for future theme integration; intentional |

---

## Commit Protocol

Use conventional commit format with structured trailers for non-trivial changes.

```
<type>(<scope>): <subject>

<optional body>

Constraint: <active constraint that shaped this decision>
Rejected: <alternative> | <reason>
Directive: <warning for future modifiers>
Confidence: high | medium | low
Scope-risk: narrow | moderate | broad
```

Include trailers when:
- A non-obvious architectural decision was made (`Constraint:`, `Rejected:`)
- Future agents/developers could easily break the invariant (`Directive:`)
- The change has broad blast radius (`Scope-risk: broad`)

Example:
```
fix(cards): rename CARD_REGISTRY → CARD_CATALOG in card-catalog.ts

card-catalog.ts is CLI-only static metadata. cards/registry.ts is the
runtime Vite registry. Both exported `CARD_REGISTRY` with incompatible
types (Array vs Map), causing confusion for agents reading imports.

Constraint: card-catalog.ts and cards/registry.ts serve different runtimes
Rejected: Merge into one file | card-catalog.ts is Node-only, registry.ts is Vite-only
Directive: Do not import card-catalog.ts in browser/Remotion code; do not import cards/registry.ts in CLI/Node code
Confidence: high
Scope-risk: narrow
```

---

## Review Guidelines

Flag as **P0 (blocking)**:
- Breaking changes to `Episode`, `Step`, or `SeriesConfig` types in `shared/types.ts`
- New imports of `card-catalog.ts` in `src/engine/` or `src/Root.tsx`
- New imports of `cards/registry.ts` in `cli/`
- Hardcoded secrets, API keys, or credentials
- `import.meta.glob` patterns that expand beyond their intended scope

Flag as **P1 (should fix)**:
- New card type without a `spec.ts` (bypasses registry)
- `any` cast in non-boundary code (i.e., not at Vite/Webpack/Node API boundaries)
- `Math.random()` in Remotion components (use `random()` from remotion instead)
- Duplicate `cardSpec.type` values across engine + series cards

Flag as **warning (non-blocking)**:
- Missing Zod schema on a new card spec
- `durationInSeconds` missing from a new step (will fail at render time)

---

## Test & Verification Baseline

There is a lightweight automated test suite (`npm test`). Verification baseline means:

1. `npm run lint` — must exit 0 (ignoring the known exemptions above)
2. `npx ars episode validate template/ep-demo` — must pass without errors
3. Remotion studio renders `ep-demo` without throwing (manual check or `remotion bundle`)
4. `npm test` — should pass before shipping workflow or registry changes

For Studio, intent, or Vite shell changes, also run `npm run build:studio`.

When adding a new card type, also verify:
- `npx ars episode stats template --all` lists the new type
- `ep-demo.ts` includes a step using the new card, and it renders

---

## What This Repo Is Not

- Not an agent framework (use OMC/OpenClaw for that)
- Not a general-purpose video editor
- Not a public API surface — `src/engine/shared/types.ts` is the only stable contract

Changes that expand the public API surface require explicit user approval.

---
> Source: [dnplus/ars](https://github.com/dnplus/ars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
