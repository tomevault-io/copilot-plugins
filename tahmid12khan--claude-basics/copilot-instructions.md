## claude-basics

> Teaching repo: HTML "storybook" decks that explain Claude Code concepts to

# claude-basic-learnings

Teaching repo: HTML "storybook" decks that explain Claude Code concepts to
engineers. Each deck is one `.html` file that links the shared design system in
`assets/` via relative paths — no build, no framework, no tests, no backend.
Works opened directly via `file://` and on GitHub Pages (only `<link>`/`<script src>`
relative references, never `fetch`). A deck needs the `assets/` folder next to it.

## Files

- `ep-1-agentic-development.html` — Agentic development: what Claude Code is (vs the model/chatbot), the loop run on a concrete task, why it beats re-prompting, the extension-stack mental model (19 slides)
- `ep-2-the-fundamentals.html` — Day-to-day Claude Code: sessions, the context window (+ an interactive grow/`/compact` playground), permission modes, models & commands, an "extend the loop" tour of the five primitives, and two click-to-reveal quizzes (28 slides)
- `ep-3-skills-subagents-hooks.html` — Skills vs Subagents vs Hooks (23 slides)
- `ep-4-claude-setup-and-claudemd.html` — claude-code-setup & claude-md-management (19 slides)
- `ep-5-mcp.html` — MCP: connecting Claude to databases, browsers & tools (15 slides)
- `ep-6-agentic-tdd.html` — Agentic TDD: a crew of subagents (spec → test → implement → verify coverage, tests-first) + step-by-step TDD & review flowcharts + a detailed criticality-aware review pipeline (5 tiers, 17 dimensions, verify + gate) + model choice (28 slides)
- `ep-7-monitoring-with-claude.html` — Monitoring with Claude: read-only eyes on prod via API/MCP, safely (19 slides)
- `ep-8-task-to-tested.html` — AI-driven QA: refine ticket → derive tests → drive Chrome → write E2E, via a Workflow of 4 QA subagents (20 slides)
- `ep-9-agentic-ownership.html` — Agentic development ownership: don't blame the AI; AI as a scalable/parallel you (capped by what you can verify); ship only what you understand & can debug; the 3 AM incident playbook (foundation prereqs → clear/describe/diagnose → adversarial multi-subagent review → deploy & observe loop) (20 slides)

Naming: `ep-<N>-<kebab-topic>.html` in the repo root. The landing index sorts by number and tolerates gaps.

## Design system (`assets/` — shared by every deck)

- **`assets/storybook.css`** — one file, three sections in cascade order:
  1. *COMIC BASE THEME* — layout, slides, frags, diagram pieces, design tokens
     (`:root` colors/sizes).
  2. *STORYBOOK THEME OVERRIDE* — recolors the comic base to the softer earthy
     palette (redefines the `:root` tokens).
  3. *EPISODE CHROME* — fixed `.deck-header`/`.deck-footer` styling.
  **Editing this file changes every deck.** Episode-specific styling never goes here.
- **`assets/deck.js`** — loaded as the last element of `<body>`. Three jobs:
  injects the shared cast sprites (`#ch-claude` engineer, `#ch-lead` tech lead);
  builds the deck header/footer from `<body data-ep-label="Ep N · Title">`;
  runs the `Deck` controller — keyboard (→/space/PageDown next, ←/PageUp back,
  Home/End), wheel, touch-swipe, progress bar (`#bar`), counter (`#counter`),
  `#/N` hash. Do not fork it per-episode.

## Deck anatomy (each episode file)

- **Head**: one Google Fonts `<link>` (Bangers, Space Grotesk, Caveat, Fraunces,
  Work Sans) + `<link rel="stylesheet" href="assets/storybook.css">`.
- **`<body data-ep-label="Ep N · Title">`** — the label deck.js puts in the header.
- **Slides**: each is one `<section class="slide">` inside `<main>`. The active
  slide gets `.active`.
- **Step reveals**: any element with class `frag` is hidden until stepped into; add
  `.pop` or `.slam` for entrance style. DOM order = reveal order.
- **Sprites**: episode-specific cast as SVG `<symbol id="ch-...">` in the hidden
  defs block, referenced via `<use href="#ch-...">`. The shared cast comes from
  deck.js — don't redefine `#ch-claude`/`#ch-lead`.
- **Per-episode CSS** in the trailing `NEW COMPONENTS` `<style>` in `<body>` —
  later in the document than the shared stylesheet, so it wins cascade ties.
- **Per-episode JS** (rare) in its own inline `<script>` after the
  `assets/deck.js` tag (see ep-3's gate-shake observer).
- ep-3 and ep-4 additionally carry an `EPISODE BASE PATCH` `<style>` at the top of
  `<body>`: their historical deviations from the shared base, kept episode-local.

## Working on decks

- **New deck**: run `/new-storybook` — scaffolds `ep-N-*.html` from
  `.claude/skills/new-storybook/template.html` (a slim skeleton that links the
  shared assets and includes sample slides + header/footer via `data-ep-label`).
  Fill placeholders; add only slides, episode sprites, and `NEW COMPONENTS` CSS.
- **Review a deck**: dispatch the `slide-deck-reviewer` subagent (rendering, a11y,
  responsive, theme drift, content hygiene).
- **Validate**: `npx --yes html-validate <file>` (or, in a sandboxed/no-network shell,
  the installed `./node_modules/.bin/html-validate <file>`) — config in `.htmlvalidate.json`
  ignores the decks' intentional patterns (inline styles, multiple `<style>` blocks in
  `<body>`) and keeps structural checks (unclosed tags, dup ids, bad nesting).

## Local automations (`.claude/`)

- `skills/new-storybook/` — user-invoked scaffold skill (+ `template.html`).
- `agents/slide-deck-reviewer.md` — read-only deck reviewer subagent.
- `hooks/validate-html.sh` — PostToolUse hook: runs html-validate on any edited/created
  `.html`, exits 2 with the errors fed back to Claude on failure (silent on success).
  Wired in `settings.json` for `Write|Edit|MultiEdit`.
- `settings.json` — enables the `turja-skills` plugin + the validate-html hook.

## Non-obvious

- The decks put `<style>` blocks inside `<body>` on purpose (per-episode CSS must
  come after the shared stylesheet to win the cascade). This is technically invalid
  HTML but works everywhere; `.htmlvalidate.json` disables the rules that would flag
  it. Don't "fix" it by hoisting styles into `<head>`.
- `npx html-validate` prints `Unknown command: "html-validate"` when it can't fetch —
  cold-cache race OR a sandboxed/no-network shell. Run `./node_modules/.bin/html-validate
  <file>` directly (v10, same `.htmlvalidate.json`). The hook uses the `--package` form to
  dodge the cold-cache case.
- The fixed **deck header/footer** (`.deck-header` = `← Episodes` link + byline +
  `Ep N · Title`; `.deck-footer` = brand) is built at runtime by `assets/deck.js`
  from `<body data-ep-label>` — there is no header/footer markup in the deck files.
- Slide *fit* is not checked by html-validate. The base theme only compacts at
  `@media (max-height: 620px/480px)`, so dense slides overflow in the ~620–768px band —
  add a scoped `@media (max-height: 768px)` block in `NEW COMPONENTS`. Verify by measuring
  `.content` `scrollHeight > clientHeight` in a browser at ~660px height with every `.frag`
  revealed (chrome-devtools MCP works).
- Count slides with `grep -oE '<section class="slide[^"]*"'` — plain `class="slide"` misses
  `slide compact` / `slide act-card`.

---
> Source: [Tahmid12Khan/claude-basics](https://github.com/Tahmid12Khan/claude-basics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
