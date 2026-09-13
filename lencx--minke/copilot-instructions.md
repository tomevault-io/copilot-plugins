## minke

> - Keep tracked files in `docs/` user-facing: setup, operation, workflows, troubleshooting, and other material needed to use Minke.

# Documentation placement

- Keep tracked files in `docs/` user-facing: setup, operation, workflows, troubleshooting, and other material needed to use Minke.
- Treat `docs/research/` as a local-only workspace covered by the root `.gitignore`. Leave its documents, indexes, assets, and generated output outside Git's index; never force-add them.
- Place architecture and upstream design under `research/architecture/`; solution paths and evaluations under `research/solutions/`; use `compatibility/`, `engineering/`, `evolution/`, and `sharing/` for their corresponding technical material.
- Split mixed-audience material: keep executable user steps in tracked `docs/`, and keep implementation detail and decision rationale in local `docs/research/`. Tracked documentation must remain self-contained and must not link to ignored research files.
- Complete a documentation move when `git status` contains no `docs/research/` entries and every local link in tracked Markdown resolves.

---
> Source: [lencx/Minke](https://github.com/lencx/Minke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
