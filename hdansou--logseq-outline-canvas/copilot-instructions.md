## logseq-outline-canvas

> Operational landmines and workflow expectations for agents in this repo. For architecture, modules, and commands, read `CLAUDE.md` — that's the discoverable side of things. **Don't duplicate** what's in `CLAUDE.md`, `README.md`, or `package.json` here.

# AGENTS.md

Operational landmines and workflow expectations for agents in this repo. For architecture, modules, and commands, read `CLAUDE.md` — that's the discoverable side of things. **Don't duplicate** what's in `CLAUDE.md`, `README.md`, or `package.json` here.

## Workflow expectation

For any non-trivial feature or bug fix:

1. **Update `docs/feature-*.md`** (or write a new one) with the scope, decisions, and tradeoffs. Lock these before coding.
2. **Update `tasks.md`** with a TDD-friendly checklist — each chunk testable, tests written first.
3. **Implement TDD-style**: failing test → minimal code to pass → refactor. Enforce DRY, KISS, YAGNI.

When user feedback mid-implementation introduces a new requirement, **pause and update the spec + tasks first**. Don't let the docs lag behind the code. The user has called this "backward" before; honor it.

## Landmines

- **`scripts/logseq-smoke.sh` is broken.** Logseq removed `window.frontend.handler.plugin.load_plugin_from_web_url_BANG_`. The script's programmatic-install path no longer works. Don't run it expecting verification — fall back to asking the user to reload the plugin manually from Logseq desktop.

- **DB-graph block property surface is non-obvious.** `block.properties` (the sub-object) is **mostly empty** in DB graphs. Properties live as **top-level namespaced keys** on the block object (e.g. `block["user.property/foo-XYZ"]`), often with leading colon (`":user.property/foo-XYZ"`). The `extractRefs` adapter pattern in `src/adapter.ts` iterates both surfaces with dedup; reuse it for any new property-reading code instead of reaching for `block.properties` directly.

- **`:node`-typed property values arrive as `{id: <number>}`** more often than as UUID-shaped objects in real graphs. Resolve via `Editor.getBlock(id)` → `.uuid` and cache the result per build. See `extractRefUuids` in `src/adapter.ts` for the canonical handling of all four shapes.

- **The inline macro renderer (`{{renderer :outline-canvas}}`) intentionally omits relationship edges** — it shows badges (counts) only. This is a UX decision (no interaction surface in a static image; click image to open interactive). Don't "fix" the apparent inconsistency by adding edges to `offscreen.renderToDataURL`.

## After code changes

The user runs the plugin from a Logseq desktop install pointed at `dist/`. After modifying any `src/**` file, **always `npm run build`** before telling the user to reload. The dev server (`npm run dev`) is for iframe-installed dev workflows, not the user's normal flow.

## Debugging connector issues

The interactive canvas logs one diagnostic line per layout rebuild:

```
[OutlineCanvas] view=<id> focus=<uuid|none> refs(intra-tree)=N rects=M
```

If a user reports "I don't see connectors," ask them to open DevTools (Cmd+Opt+I in Electron) and copy that line — `refs=0` means the adapter isn't extracting them; `refs>0` but `rects=0` means the view doesn't expose `nodeRectsByUuid`.

---
> Source: [hdansou/logseq-outline-canvas](https://github.com/hdansou/logseq-outline-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
