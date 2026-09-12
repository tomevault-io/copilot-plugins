## thousand-words

> Stop and read this before touching anything. This repo is a Claude Code

# thousand-words — Agent Guidelines

## If You Are an AI Agent

Stop and read this before touching anything. This repo is a Claude Code
plugin, and it has product rules (local-first) that override your default
instincts about how to present visual output.

The two rules agents break most often:

1. **Never publish rendered output through the Artifact tool** (and don't
   load the `artifact-design` skill for it). See "The local-first promise".
2. **Never hand-edit the server bundles.** See "Repo layout".

## What this project is

thousand-words turns a sentence into an interactive visual. The model emits a
tiny JSON **spec**; a vendored MCP server renders it into a **standalone
HTML/SVG file** with a bundled vanilla-JS runtime. Once rendered, all
interaction (sliders, steppers, diagram editing) runs locally in the browser
at zero model calls.

Two MCP servers, four skills:

- `servers/gui-maker/` → tool **`render_gui`** — interactive GUIs in two
  modes: COMPUTE (privacy-preserving calculators, forms, what-if sliders,
  tables, live plots, CSV/HTML export) and WALKTHROUGH (stepwise Prev/Next
  algorithm explanations with per-step notes and highlighted cell grids).
- `servers/diagram-maker/` → tool **`render_diagram`** — box-and-arrow
  diagrams (dagre/graphviz layout, browser editor), sequence diagrams, data
  plots, pies, and fsm state machines; outputs an editable HTML page or
  static SVG.
- `skills/making-visuals` routes a request to the right tool and mode;
  `building-compute-guis`, `building-walkthroughs`, and `making-diagrams`
  carry the page-design craft for each.

**The one rule of the product: one page, one tool.** A page is either a GUI
or a diagram, never both. If a request needs both, render two pages and say
so.

## The local-first promise

The selling point is that nothing the user enters into a rendered page ever
reaches a server or a model. Every choice below follows from that:

- Rendered output is a **local file**. Deliver it as a file path or open it
  in the browser. Do **not** publish it via the Artifact tool — Artifacts
  upload the page to claude.ai hosting, which breaks the promise outright.
  Only exception: the user explicitly asks to share something as an artifact.
- Pages must work offline, forever. No CDN scripts, no webfonts (system font
  stacks only), no fetch/analytics/telemetry in templates or runtimes. If a
  change would make a rendered page touch the network, it is wrong.
- For sensitive-data requests, this privacy property is the pitch — the
  skills say to lead with it. Don't undercut it in your own delivery.

## Repo layout

- `servers/*/<name>-mcp.mjs` are **prebuilt, self-contained bundles**, each
  shipped beside its `template.html`. Never hand-edit a bundle — a change
  there is unreviewable and will be lost on the next release. If a server
  misbehaves, file an issue instead.
- `mcp.json` wires both servers via `${CLAUDE_PLUGIN_ROOT}` paths only, so
  the plugin works from wherever the plugin cache puts it.
- `scripts/render-spec.js` renders a spec through a vendored server with no
  MCP client. `docs/cookbook.md` holds copy-paste spec recipes.

## Skills are tuned content, not prose

The four `skills/*/SKILL.md` files are behavior-shaping content that was
benchmarked against alternative phrasings before release (variants that read
better but performed worse were rejected). Do not reword, restructure, or
"improve the style" of a skill without eval evidence that the change helps.
Frontmatter format is enforced by tests.

## Verifying changes

- `node --test tests/*.test.js` — manifests are sound, every skill has
  frontmatter, both servers answer over stdio, and a spec renders end to end.
- `claude plugin validate .` — marketplace + plugin manifest check.
- `node scripts/render-spec.js <gui|diagram> spec.json out.html` — render a
  spec and open the result in a browser to eyeball it.
- Install story end to end: `/plugin marketplace add jycd25/thousand-words`
  then `/plugin install thousand-words@thousand-words`.

"It should work" is not verification. Render a page and look at it.

---
> Source: [jycd25/thousand-words](https://github.com/jycd25/thousand-words) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
