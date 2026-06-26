## feishu-deck-h5

> > Cross-tool agent entry (Codex / Cursor / any platform that reads `AGENTS.md`).

# AGENTS.md — feishu-deck-h5

> Cross-tool agent entry (Codex / Cursor / any platform that reads `AGENTS.md`).
> **Claude Code** users get this automatically via the `feishu-deck-h5` skill + local
> hooks; this file gives **non-Claude** agents the same cardinal rules so they don't
> re-discover the CLI, hand-extract `deck.json`, or render the whole deck for a
> one-line change. Canonical, full version:
> `skills/feishu-deck-h5/references/raw-page-quickstart.md` (and that skill's `SKILL.md`).

## What this repo does
Renders a structured `deck.json` into a 16:9 Feishu/Lark-style HTML deck
(`runs/<deck>/index.html`). **`deck.json` is the single source of truth; `index.html`
is derived** — always edit `deck.json` then re-render. Never hand-edit `index.html`.

## Tools (paths relative to repo root)
- Editor:   `skills/feishu-deck-h5/deck-json/deck-cli.py`
- Renderer: `skills/feishu-deck-h5/deck-json/render-deck.py`
- Page map: `skills/feishu-deck-h5/deck-json/deck-map.py <index.html|deck.json>`
- Validator:`skills/feishu-deck-h5/assets/validate.py <index.html> [--strict]`

Run `deck-cli.py -h` / `render-deck.py -h` for full flags. Page addressing:
**`#N` (URL hash) = page N = `slides[N-1]`** (0-based in JSON paths).

## Cardinal rules (violating these = slow / broken deck)
1. **Read one page's content** — don't `json.load` / `jq` the `deck.json` directly
   (fragile, loses the optimistic lock). Use:
   `deck-cli.py <deck.json> get-page <key|#N> --html|--css`
2. **Change one value** (fit_width / a color / accent …):
   `deck-cli.py <deck.json> set slides.<N-1>.<dotted.path> <value>`
   (path is 0-based; `--str` for a raw string; `--from-file f` for large css/html).
3. **Swap a page's whole `custom_css` / `data.html` while keeping `lifted`/`title`**:
   `deck-cli.py <deck.json> set slides.<N-1>.custom_css --from-file f.css`
   (`set-page` rewrites the full page payload + resets the `lifted` marker via
   `--lifted`; for a single field use `set`, not `set-page`).
4. **iframe-embed demo text too small / side gutters** — don't touch `custom_css`,
   set `data.fit_width` to the prototype's design width:
   `deck-cli.py <deck.json> set slides.<N-1>.data.fit_width <design-width>`
   (renderer derives `zoom = 1800 / fit_width`; smaller fit_width → bigger).
5. **Re-render only the changed page**:
   `render-deck.py <deck.json> <out-dir> --scope <N> --shoot`
   (one-page edit ≠ whole-deck render; use `--final` only at delivery).
6. **An embedded local prototype** (`runs/<deck>/prototypes/<x>/index.html`) is loaded
   by the slide's `<iframe src>` at runtime — edit that file and it's live; **no
   re-render needed** if `deck.json` didn't change. Just re-shoot / refresh.

## Fixed constants (memorize; don't re-derive)
- Canvas **1920×1080**, absolute positioning; JS scales via `--fs-scale`.
- Type ladder is **{16, 24, 28, 48} only** (body floor 24; chrome — eyebrow/tag/pill/
  page-number/axis-label/≤7-char label — floor 16; a real hero numeral that must
  exceed adds `/* allow:typescale */` in its rule).
- `deck.json` slide shape:
  `{key, layout, screen_label, data:{html, title?}, custom_css, accent?, lifted?, allow?}`
  — HTML in `data.html`, bespoke CSS in `custom_css`, the visible title in the HTML's
  `.title-zh` (`data.title` is metadata, often empty on raw pages).
- Dark content background: `#050a17 url("assets/lark-content-bg.jpg") center/cover`.
- Entrance/emphasis motion lives **only in `slide.custom_css`** (deck.json has no JS
  slot — re-render wipes any `<script>` / head `<style>`).

## Quality / delivery gate (this is the portable hard part)
- **Before calling a deck done, run the validator on its `index.html`** and fix
  non-zero findings: `python3 skills/feishu-deck-h5/assets/validate.py <index.html>`
  (or `render-deck.py … --final`, which validates the whole deck). Do **not** hand-roll
  HTML to dodge it. `validate.py` exit code: `0` pass · `1` fail · `2` file-not-found.
- **CI** (`.github/workflows/validate.yml`) gates the **framework + tools + the sample
  deck** (`examples/sample-deck.html`) + the deck-json unit tests on every push/PR. It
  does **not** see `runs/<deck>/` decks — those are gitignored and never committed, so
  per-deck quality relies on the validator above, not CI. CI is what protects you when
  you change the framework css/js or the tools.

## Where the full rules live
- Edit/insert fast-path + constants: `skills/feishu-deck-h5/references/raw-page-quickstart.md`
- Everything else (modes, layouts, motion, publishing, importing):
  `skills/feishu-deck-h5/SKILL.md` and `skills/feishu-deck-h5/references/`.

---
> Source: [FuQiang/feishu-deck-h5](https://github.com/FuQiang/feishu-deck-h5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
