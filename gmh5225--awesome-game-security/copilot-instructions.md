## awesome-game-security

> Load only the repository layer needed by the task:

# Awesome Game Security agent guide

Load only the repository layer needed by the task:

- Use `README.md` as the canonical resource index and category taxonomy.
- Use `.claude/skills/overview/SKILL.md` for repository discovery or collection maintenance, then load one matching domain skill.
- Use `description/<owner>/<repo>/description_en.txt` for maintained summaries; inspect `archive/` only when the task requires primary repository content that the summary cannot establish.
- Follow `wiki/AGENTS.md` for any work under `wiki/`. Its generated `wiki/sources/**` projections are not commit targets.
- Read the relevant script and workflow together when changing discovery, archival, description generation, translation, or wiki automation.

Collection membership is not an endorsement or proof of capability. Distinguish repository metadata, maintained summaries, primary-source evidence, inference, and recommendations. For consequential or disputed technical claims, use the research-rigor skill with the relevant domain skill and state version, environment, limitations, and uncertainty.

Keep skill descriptions short and discriminating. Keep each `SKILL.md` as a routing and constraint layer; put topic-specific catalogs and procedures in references, and read only the reference needed for the current task.

Local indexing, lint, dry-run, and skill-validation commands are safe to run without repeated approval. Start with the narrowest check that covers the change, fix failures caused by the requested work, and broaden only when shared automation or generated outputs are affected. Do not launch networked discovery, archive updates, wiki agents, pushes, or workflows unless the task authorizes those external effects.

Preserve unrelated working-tree files, generated projections, archives, and descriptions. Do not rewrite `README.md`, wiki pages, or source summaries as incidental cleanup.

---
> Source: [gmh5225/awesome-game-security](https://github.com/gmh5225/awesome-game-security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
