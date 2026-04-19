## sofienekhiari

> Website architecture and design decisions for sofk.ch


# Website Architecture — sofk.ch

## Single-page with dialogs pattern

The site is a **single `index.html`** file. All content lives in one DOM so AI crawlers and LLMs get everything in a single fetch. Visual complexity is managed through progressive disclosure:

- **Always visible**: Hero/about, experience timeline, summary cards for publications/projects, condensed skill tags.
- **`<dialog>` elements**: Full publication list, full project portfolio, full uses/tools grid. Content is in the DOM (AI-readable) but only shown visually on user interaction.
- **`<details>/<summary>`**: For smaller inline expansions.

**Do not split content into separate pages.** If a new section is needed, add it as a section in `index.html` with an anchor link in the nav. Use a `<dialog>` for overflow content.

## AI discoverability

- **JSON-LD** `Person` schema in `<head>` — keep it updated when adding publications, projects, or experience entries.
- **`llms.txt`** at `docs/llms.txt` — update when adding new sections or significant content.
- **Semantic HTML** — use `<article>`, `<section>`, `<h1>`–`<h3>` hierarchy, `<time>` for dates.

## Tech stack

- Pure HTML + CSS + vanilla JS. No frameworks, no build tools, no bundlers.
- `index.html` — single page, all content.
- `styles.css` — all styling via CSS custom properties with light/dark mode.
- `script.js` — all JavaScript (theme toggle, scroll observers, animations, dialog logic). Keep it vanilla and progressively enhanced — the site should remain functional without JS.
- `assets/bootstrap-icons.css` + `.woff` — icon font (only external dependency).
- Inter font via Google Fonts CDN.

Animations should feel smooth and intentional. Use CSS transitions/animations where possible; use JS only for orchestration (IntersectionObserver triggers, staggered timing, scroll-driven effects). Avoid heavy animation libraries.

## Experience timeline — git-log design

The Experience section uses a **git commit history** visual metaphor:
- Each entry is a compact one-liner resembling `git log --oneline --graph` output.
- Date displayed in monospace like an abbreviated commit hash.
- Branch-style labels distinguish Professional (teal) from Academic (amber).
- The commit message is the role/degree title; the author line is the organization.
- Clicking an entry opens a `<dialog>` with full details (like `git show`).
- The vertical line draws progressively as the user scrolls (scroll-driven, not just on-enter).
- Commit nodes pop in as the line reaches them, entries fade in sequentially.

Keep entries compact on the main page — all detail goes in the per-entry dialogs.

## Accent color

Teal `#008080` (light mode) / `#4dd0d0` (dark mode). All accent values defined as CSS custom properties.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sofienekhiari) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
