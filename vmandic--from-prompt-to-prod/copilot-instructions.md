## from-prompt-to-prod

> - **Talk materials:** PDF + Keynote at repo root; outline and links in [`README.md`](README.md).

# AGENTS.md — from-prompt-to-prod (rudimentary)

## What this repo is

- **Talk materials:** PDF + Keynote at repo root; outline and links in [`README.md`](README.md).
- **Self-check rubric (agent prompt):** [`src/verify-your-agentic-workflows.md`](src/verify-your-agentic-workflows.md) — A–K dimensions vs the talk; materials-only repos may show N/A for some file-based rows.
- **Static site (GitHub Pages):** source lives in **`docs/`**. With Pages set to publish from **`/docs` on `main`**, the **public site root** is `https://vmandic.github.io/from-prompt-to-prod/` — there is **no** `/docs` segment in the URL. **Internal links use root-relative paths** (`/from-prompt-to-prod/...`) so nav and CSS work whether or not the URL has a trailing slash (GitHub project Pages quirk); local `file://` or a bare `http.server` at repo root will not match that base unless you fake the path.

## Conventions we settled on in repo work

- **Do not** add a top-level folder named `writing-plans` for the site — that was a **planning-skill** label, not a product path. The published site is **`docs/`**.
- **Regenerate** [`docs/verify/index.html`](docs/verify/index.html) from the markdown: `python3 scripts/build_ghpages.py` (needs `pip install -r requirements.txt`, or a project **`.venv`** with `markdown` installed), or `npm run build:site`. Commit the HTML if you change the `.md` source and want the live site to match. **Smoke test:** `python3 -m unittest discover -s tests -v` (same Python as the build).
- **Run the site locally:** from repo root, `python3 scripts/serve_site.py` — **reads `docs/` live** (not a one-time copy); save HTML/CSS/JS in `docs/` and **refresh the browser** to see changes (no server restart; optional `Cache-Control: no-store` on responses). Use `PORT=9000` if the default (`8765`) is taken. For **layout/DOM checks**, use that origin (e.g. `http://127.0.0.1:8765/from-prompt-to-prod/verify/`) when the user provides it.
- **Slides on the site:** PDF is linked via **raw** `main` on GitHub (repo root is not inside `docs/`-only deploy).
- **README:** keep it for **readers** of the talk; avoid long “how to build the site” blocks unless the owner asks.
- Shields / badges: the owner preferred a **single website** badge; keep README minimal unless told otherwise.

## If you are unsure

- Read [`README.md`](README.md) and the first section of `src/verify-your-agentic-workflows.md` before big edits.

---
> Source: [vmandic/from-prompt-to-prod](https://github.com/vmandic/from-prompt-to-prod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
