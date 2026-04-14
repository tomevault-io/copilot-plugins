## rightmove-london-portfolio

> Project Intelligence (.cursorrules)

Project Intelligence (.cursorrules)

Working norms
- Always read all Memory Bank files at the start of every task.
- Update Memory Bank after significant changes (features, architecture, workflows).
- Prefer absolute paths in tool calls; keep outputs under `out/`.

Scraping patterns & guardrails
- Use Playwright with `domcontentloaded` and then wait for property cards to render.
- Keep `MAX_CONCURRENCY` low by default; randomize delays; use realistic desktop `USER_AGENT`.
- Enforce consent gate: `ALLOW_DISCOVERY=true` and `consent.txt` file in project root.
- Write debug HTML on empty/failed discovery pages for offline inspection.

Architecture preferences
- Plan → Discover → Scrape as separate, composable steps; each writes durable artifacts.
- Batch writes frequently during scraping; resume by `rightmove_id` threshold when needed.
- Keep extractor logic in `extractors.py` narrowly focused; normalize separately in `normalize.py`.

Developer preferences
- Use Typer for CLI UX; keep prompts clear and defaults conservative.
- Tests live in `tests/` with HTML fixtures; focus on extractor robustness.
- Logging via Rich; prioritize structured, actionable messages over verbosity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinalyst) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
