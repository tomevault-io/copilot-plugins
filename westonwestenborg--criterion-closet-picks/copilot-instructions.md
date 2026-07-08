## criterion-closet-picks

> This repository is an Astro static site for Criterion Closet Picks. Site code lives in `src/`: pages in `src/pages`, UI in `src/components`, layouts in `src/layouts`, data helpers in `src/lib`, and global styling in `src/styles/global.css`. Fonts and imported assets are in `src/assets`; static files such as `CNAME` and `tmdb-logo.svg` are in `public/`.

# Repository Guidelines

## Project Structure & Module Organization

This repository is an Astro static site for Criterion Closet Picks. Site code lives in `src/`: pages in `src/pages`, UI in `src/components`, layouts in `src/layouts`, data helpers in `src/lib`, and global styling in `src/styles/global.css`. Fonts and imported assets are in `src/assets`; static files such as `CNAME` and `tmdb-logo.svg` are in `public/`.

Committed data lives in `data/`, especially `guests.json`, `picks.json`, `picks_raw.json`, and `criterion_catalog.json`. Python pipeline and validation scripts live in `scripts/`. Build output goes to `dist/` and should not be edited by hand.

## Build, Test, and Development Commands

Use Bun for JavaScript tasks:

```bash
bun install              # install JS dependencies
bun run dev              # start Astro dev server
bun run build            # build static site into dist/
bun run preview          # preview the production build
bun run validate         # run Python data integrity tests
```

For the Python data pipeline:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r scripts/requirements.txt
python scripts/process_video.py --youtube-url "https://www.youtube.com/watch?v=..."
```

After production builds, generate search with `bunx pagefind --site dist` when needed.

## Coding Style & Naming Conventions

Follow existing Astro and TypeScript style: concise modules, named exports where practical, and PascalCase component files such as `GuestCard.astro`. Route files should match Astro conventions, for example `src/pages/guests/[slug].astro`. Keep shared CSS in `src/styles/global.css`.

Python scripts use `snake_case` filenames and functions. Prefer small pipeline steps that read and write explicit files in `data/`. Keep generated JSON stable and human-reviewable.

## Testing Guidelines

Data integrity tests are in `scripts/test_data.py` and run with `bun run validate` or `python scripts/test_data.py`. Run them after changing `data/` files or pipeline scripts. Run `bun run build` after frontend changes to catch Astro routing, import, and prerendering errors.

## Commit & Pull Request Guidelines

Recent commits use short imperative subjects, often with the affected guest and pick count, for example `Add Bob Odenkirk's Closet Picks (13 films)`. Keep commits focused: separate data updates, pipeline changes, and UI changes when practical.

Pull requests should explain what changed, list validation commands run, and note any data sources or API-backed scripts used. Include screenshots for visible UI changes.

## Security & Configuration

Keep API keys in local environment files only. The pipeline may require TMDB and Gemini credentials; never commit `.env`, transcripts, or other large regenerable artifacts. TMDB attribution in the footer must remain intact.

---
> Source: [westonwestenborg/criterion-closet-picks](https://github.com/westonwestenborg/criterion-closet-picks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
