## gaido

> Local-first framework for visual creative agent workflows on a node graph. A coder agent generates visual code (Pixi/canvas/HTML), a renderer captures it as video, a critic agent gives feedback, and the whole exploration is shown as a forkable xyflow graph. Eventual two use cases: open-ended creative-exploration and videoeffects.com production templates with senior-artist review.

# Gaido — project notes for Claude

Local-first framework for visual creative agent workflows on a node graph. A coder agent generates visual code (Pixi/canvas/HTML), a renderer captures it as video, a critic agent gives feedback, and the whole exploration is shown as a forkable xyflow graph. Eventual two use cases: open-ended creative-exploration and videoeffects.com production templates with senior-artist review.

This file is the orientation. Detail lives in the imports below; code is the source of truth for *how*.

@docs/architecture.md
@docs/graph-model.md
@docs/conventions.md
@docs/lessons.md

## Where else to look

- **PRODUCT.md** and **DESIGN.md** — brand, register, visual system. Defend design choices against the five principles in `docs/conventions.md`.
- **TODO.md** — known leftover issues and still-ahead work.
- **`docs/publishing.md`** — design for static export of a canvas to Cloudflare (R2 + Worker). Not yet built.
- **`playwright-testing` skill** — load when driving the UI via Playwright MCP. Covers `window.__gaido` and the `data-testid` map.

---
> Source: [vlobanov/gaido](https://github.com/vlobanov/gaido) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
