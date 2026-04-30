## explore-claude-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

An interactive educational website that teaches Claude Code features by simulating a project you explore. Static HTML/CSS/JS — zero build steps, no framework, no bundler.

## Serving Locally

```bash
# Any static server, pointed at the site/ directory
npx serve site
python -m http.server -d site 8080
```

Opening `site/index.html` directly also works (fetches manifest via relative path).

## Architecture

All educational content is stored as JSON strings inside `site/data/manifest.json`. This single file drives the entire UI — tree structure, file content, labels, badges, and feature groupings. To add or change content, edit the manifest.

**Component classes (all vanilla JS, no modules, loaded via `<script>` tags):**

- `App` (app.js) — Controller. Loads manifest, wires components, handles keyboard nav (arrow keys), hash routing, traffic light buttons, and the void easter egg (minimize button → canvas particle animation).
- `FileExplorer` (file-explorer.js) — Sidebar tree. Draws connector lines (├── └──) on `<canvas>` elements inside `.tree-children-guided` containers. `.claude` is auto-expanded on load.
- `ContentLoader` (content-loader.js) — Renders file content. Has a hand-rolled markdown parser supporting: YAML frontmatter (rendered as tables), fenced code blocks, tables, lists, inline formatting, and links. Markdown files get a Rendered/Raw toggle. Syntax highlighting via Prism.js.
- `Terminal` (terminal.js) — Right-side panel. Interactive slash command emulator (`/help`, `/init`, `/doctor`, `/diff`, `/compact`, `/model`, `/cost`, `/status`, `/config`, `/memory`). Animated output sequences.
- `ProgressTracker` (progress.js) — Tracks visited features in localStorage under key `tcc-progress`.

**CSS is split by concern:** `variables.css` (design tokens), `layout.css` (shell/sidebar/content grid), `components.css` (tree items, badges, content panels, frontmatter), `syntax.css` (Prism overrides), `terminal.css`, `void.css` (easter egg).

## Critical Invariants

**Canvas DPI scaling:** `_createCanvas()` in file-explorer.js already calls `ctx.scale(dpr, dpr)`. Callers must never scale the context again or tree connector lines will misalign on high-DPI displays (coordinates get multiplied by dpr²).

**Static tree line timing:** The `.claude` directory is auto-expanded on load. `_drawStaticLines` uses double `requestAnimationFrame` to ensure the browser has completed layout before measuring `offsetTop`/`getBoundingClientRect`. If the zero-dimension guard triggers, it retries on the next frame.

**Frontmatter handling:** The markdown renderer detects `---` fenced blocks at the start of content and renders them as styled tables. Without this, `---` becomes `<hr>` and YAML `#` comments render as headings.

**Manifest node schema:** Each tree node has `name`, `path`, `type` ("file"|"directory"|"separator"). Files can have: `content` (markdown/code string), `feature` (groups related files), `badge`, `label`, `description`, `command`. Directories have `children` array. Separator nodes have only `type: "separator"` and render as a dashed divider line.

**Content title priority:** The content loader displays `node.label` first, then falls back to the feature title, then the file name. This matters for the built-in section where multiple files share a feature but need distinct titles (e.g., each bundled skill shows its `/command` name, not "Bundled Skills").

**Related files for built-in section:** Files under `built-in/` only link back to overview files (e.g., `BUNDLED-SKILLS.md`), not to every sibling sharing the same feature. This is filtered in `content-loader.js`.

**Code block first-line indent bug:** The global `code` styles (padding, background, border) were inherited by `<code>` inside `.md-code-block`, causing a visible indent on the first line of rendered code blocks. Fixed by resetting `<code>` inside `.md-code-block` to `padding: 0; background: none; border: none`.

**Content file line endings:** Always use Unix (LF) line endings for content files in `site/content/`. Windows CRLF can cause rendering issues in code blocks even though the markdown renderer normalises line endings.

## Content Design Principles

- Content should feel like exploring a real repo — self-describing boilerplate that explains itself
- Concise overview for scanning, with depth available for those who want it
- Each `.claude/` subfolder has a grounding entry-point file (e.g., `SKILLS.md`) outside the scaffolding, then the scaffolding demonstrates the actual structure
- The `built-in/` section covers features that ship with Claude Code and require no setup. A visual separator (dashed line) divides it from the `.claude/` project config above. Each built-in category gets an overview file and individual entries in subdirectories
- Avoid em-dashes in content. Use commas, periods, or colons instead

---
> Source: [LukeRenton/explore-claude-code](https://github.com/LukeRenton/explore-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
