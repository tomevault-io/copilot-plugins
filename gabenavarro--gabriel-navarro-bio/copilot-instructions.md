## gabriel-navarro-bio

> FastHTML + MonsterUI portfolio site. Server-rendered Python, no JS framework. BigQuery-backed blog content. Auto-deploys to Google Cloud Run on push to `main`.

# gabriel-navarro-bio

FastHTML + MonsterUI portfolio site. Server-rendered Python, no JS framework. BigQuery-backed blog content. Auto-deploys to Google Cloud Run on push to `main`.

## Stack

- **Web framework**: FastHTML (`from fasthtml.common import *`) + MonsterUI (`from monsterui.all import *`), built on FrankenUI 2.
- **Server**: Uvicorn via `serve()` in `app.py`. Python 3.13 only.
- **Data**: BigQuery (`noble-office-299208.portfolio.gn-blog`) via `src/services/gcp/bigquery.py` REST client. No SQLAlchemy / ORM. Schema includes `body` (markdown) AND `body_html` (pre-rendered, post-lint HTML — render happens at submit time, not per request).
- **Validation**: Pydantic v2 (`src/services/blog_frontmatter.py` for `BlogFrontmatter` and `BlogRow` BQ payload model).
- **CSS**: hand-written in `src/styles/{_base,_layout,_components,_pages}.py`, concatenated into `FACTORY_CSS`. Tailwind preflight comes via FrankenUI; no Tailwind config of our own.
- **No React / TypeScript / Vite / Vitest** — server-rendered Python only.

## Project structure

- `app.py` — entry point: creates the FastHTML app, registers routes, calls `serve()`.
- `src/core/` — `app_factory.py` (theme + headers), `routes.py` (route registration; static-asset reorder for `/feed.xml`).
- `src/components/` — `base/` (Card, button helpers), `layout/` (StandardPage, navigation, footer).
- `src/features/` — page features grouped by route (hero, projects, cv, feed).
- `src/services/` — `gcp/bigquery.py`, `projects.py`, `blog_frontmatter.py`, `blog_lint.py` (auto-fix mistletoe foot-guns in inline-SVG markdown), `blog_render.py` (markdown → HTML at submit time + post-render validation).
- `src/cli/` — `python -m src.cli blog {validate|submit|update|disable|list}`.
- `src/styles/` — `_base.py`, `_layout.py`, `_components.py`, `_pages.py` concatenated into `FACTORY_CSS`. (`custom_css.py` is orphan dead code, not bundled.)
- `src/models/project.py` — `Project` dataclass with `from_dict` for BQ rows. Carries both `body` (markdown) and `body_html` (rendered); auto-computes `slug` via `python-slugify` if BQ row lacks one.
- `tests/` — pytest with `mock_bq` fixture in `conftest.py`.
- `assets/blogs/*.md` — markdown blog sources (legacy `@{...}` or YAML frontmatter; both supported by parser).

## Setup / dev loop

- **Install**: `pip install -e ".[dev]"` (`requirements.txt` was deleted in Epic D — `pyproject.toml` is the source of truth).
- **Run**: `make dev` or `python app.py --port 8080`.
- **Test**: `make test` or `pytest -q`.
- **Lint**: `make lint` (`ruff check . && ruff format --check .`).
- **After pulling main**, if any new runtime deps were added, re-run `pip install -e .` — symptom otherwise is `ModuleNotFoundError` on app start.

## Coding conventions

- Python 3.13, type hints, `ruff` format/lint, Google-style docstrings.
- Format on save. Lint before commit. Pre-commit runs ruff + pytest automatically.
- Star imports are idiomatic for FastHTML/MonsterUI: `from fasthtml.common import *`, `from monsterui.all import *`. Ruff is configured to ignore `F403`/`F405`.

## Layout & CSS conventions

- **No inline `style=`** in `src/features/**`. Acceptance check: `grep -r "style=" src/features/` must return empty.
- **Use the `Card` primitive** (`src/components/base/card.py`) for any bordered container, not ad-hoc `Div(style="border: ...")`.
- **Use button helpers** (`button_primary|outline|ghost`), not inline `<A>` CTAs.
- **MonsterUI documented patterns only**: `NavBar(*A_items, brand=...)` with positional `A(...)` args. Do NOT use `Ul(Li(A(...)), cls="uk-navbar-nav")` — FrankenUI 2 dropped many UIkit classes (`.uk-navbar-nav` is a no-op).
- **`100vw` is a trap** — includes scrollbar width; use `100%` unless you specifically need scrollbar-included.
- **`overflow-x: clip` not `hidden`** when locking horizontal scroll — `hidden` creates a new containing block and breaks `position: sticky`.
- **Category colors** live as `--cat-{omics|ml|infra|viz}` CSS vars in `_base.py`. Never hardcode the hex values elsewhere; use `category_class(tag)` from `src.config.settings` to map a tag to its `cat-*` class name.

## Testing patterns

- `tests/conftest.py::mock_bq` patches `src.services.projects.BigQueryClient` (at the **import site**, not the definition site `src.services.gcp.bigquery.BigQueryClient`).
- Use `to_xml(el)` as a top-level function from `fasthtml.common`, NOT `el.to_xml()` — FastHTML's `__getattr__` resolves the latter to `None`.
- TDD red-bar: mark intentionally-failing tests `@pytest.mark.xfail(reason="ticket-id implements...")`; remove the marker when the implementing ticket lands.
- Don't double-mock `Project.from_dict` — let the model exercise itself; only mock at the BQ boundary.

## Git workflow

- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`, `build:`, `ci:`, `style:`.
- Feature branches off `main`. PRs require passing CI.
- Every commit ends with footer:
  ```
  Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
  ```
- NEVER `--amend` — make a NEW commit per fix (Git Safety Protocol).
- NEVER push to `main` directly. NEVER use `--no-verify`.
- Branch via worktree: `git worktree add ~/.config/superpowers/worktrees/app/<branch> -b <branch> origin/main`. Copy `CLAUDE.md` into the new worktree (it's untracked in `/app`).
- Do NOT commit `.env`, `node_modules/`, `__pycache__/`, `.claude/`.

## Auto-deploy

- **Push to `main` auto-deploys** to Cloud Run via `.github/workflows/deploy.yml`. ~3 min from merge to live.
- WIF auth (no JSON keys); secrets `GCP_WIF_PROVIDER` + `GCP_SERVICE_ACCOUNT` already set.
- Service `gnbio` in `us-central1`. Image: `us-central1-docker.pkg.dev/noble-office-299208/mercy-of-toren/gnbio:{prod,sha-<short>}`.
- Public URLs: `https://gabriel.navarro.bio` (custom domain), `https://gnbio-7xe35xaenq-uc.a.run.app`.
- Rollback: `gcloud run deploy gnbio --image=...:sha-<old-short> --region us-central1`.

## FastHTML / MonsterUI quick reference

(Full docs at `/app/env/llm.txt` and `/app/env/monsterui-llm.txt`. Below are the patterns this repo uses; reach for these first.)

### FastHTML

- **App init** lives in `src/core/app_factory.py`: `fast_app(hdrs=(Theme.slate.headers(highlightjs=True), Favicon(...)), title="...")`. Returns `(app, rt)`.
- **Routes**: `@rt("/path")` decorator. Function name is the URL when no path passed. Type-annotated params are query string; path params come from `{name}` in the route string (`@rt("/blogs/{blog_id}")` → `def get_blog(blog_id: str)`).
- **Return types**: FT components / tuples (auto-rendered to HTML), Starlette `Response` (used directly — see `/feed.xml` returning `Response(body, media_type="application/rss+xml")`), or JSON-serializable values.
- **Rendering rules**: components call `__ft__`; strings are HTML-escaped by default; bypass with `NotStr(html_string)` or `Safe(...)`. Used in `src/features/cv/diagrams.py` to inline raw SVG with camelCase `viewBox` (FT lowercases attributes otherwise).
- **`to_xml(el)`** is a top-level function from `fasthtml.common`, NOT `el.to_xml()`.
- **Static-asset catch-all trap**: `fast_app` registers `/{fname:path}.{ext:static}` BEFORE user routes. Extensions in `_static_exts` (`.xml`, `.css`, `.js`, ...) get captured by it. Workaround in `src/core/routes.py` reorders routes so `/feed.xml` matches first. Apply the same trick for any future `/sitemap.xml`, `/robots.txt`, etc.
- **Markdown**: `monsterui.all.render_md(text)` (mistletoe under the hood). Blog detail prefers `NotStr(project.body_html)` (rendered at submit time, see `src/services/blog_render.py`) and falls back to `render_md(project.body)` for legacy rows. `Theme.X.headers(highlightjs=True)` wires highlight.js for `<pre><code>` blocks.
- **Mistletoe foot-guns** in markdown bodies that contain inline SVG/HTML: see `src/services/blog_lint.py` for the auto-fixers (multi-line SVG opening tags, blank lines inside `<svg>`, named entities, indented widget skip). Run `python -m src.cli blog validate <path>` before submit; it lint+render+validates without hitting BQ.
- **Server**: `from fasthtml.common import serve; serve(port=N, reload=False)` — uvicorn under the hood.

### MonsterUI / FrankenUI

- **Theme**: `Theme.{slate|stone|gray|neutral|red|rose|orange|green|blue|yellow|violet|zinc}.headers(highlightjs=True)`. Currently `slate`.
- **`NavBar(*c, brand=...)`** — pass nav items as positional `A(...)` args (NOT `Ul(Li(A(...)))`). `brand=` for the left side. `sticky=True` for sticky positioning. `cls=` adds classes to the outer `<nav>`.
- **Layout helpers** (use these instead of writing flex CSS):
  - `DivLAligned`, `DivRAligned`, `DivCentered`, `DivFullySpaced` for flex rows
  - `DivVStacked`, `DivHStacked` for stacked items
  - `Grid(*children, cols_min=, cols_sm=, cols_md=, cols_lg=)` — responsive grid; cols_* per breakpoint
- **`Container(...)`** wraps page content; `cls="uk-container-large"` for max-width 1200.
- **`Card`, `CardHeader`, `CardBody`, `CardFooter`** are MonsterUI primitives — but THIS REPO uses its own `Card` at `src/components/base/card.py` (simpler API). Prefer the local one.
- **What FrankenUI 2 dropped from UIkit 3**: `.uk-navbar-nav` and several other `.uk-*` legacy classes. If you find yourself reaching for an `uk-*` class, verify it exists in `franken-ui@2.0.0/dist/css/core.min.css` first (`curl` + `grep`) — many no-op silently.

## Out of scope (do not propose)

- Adding FastAPI / SQLAlchemy / Alembic / React / Vite / Vitest / Tailwind config — none are in this stack.
- Migrating blog content out of BigQuery.
- Admin web UI for blog editing (the CLI under `python -m src.cli blog ...` is the only authoring path).
- Lockfiles (`requirements.lock` / `uv.lock`) — intentionally deferred.
- Touching `/projects` page — owner will tackle later (placeholder by design).

---
> Source: [gabenavarro/gabriel-navarro-bio](https://github.com/gabenavarro/gabriel-navarro-bio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
