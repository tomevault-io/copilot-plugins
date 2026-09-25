## sci-brain

> Read and follow [CLAUDE.md](CLAUDE.md) before changing this repository. It is

# AGENTS.md

Read and follow [CLAUDE.md](CLAUDE.md) before changing this repository. It is
the canonical project guide shared by Claude Code, Codex, OpenCode, and pi; do not
duplicate its skill catalog here.

Repository-wide checks:

- Treat each `skills/<name>/SKILL.md` frontmatter `name` as the public skill
  name, and keep the containing directory, documentation, and tests aligned.
- Preserve the canonical knowledge-base layout under `.knowledge/`, including
  `.knowledge/references.bib`.
- Run `python3 scripts/validate_skills.py`, `pytest -q`, and `git diff --check`
  before opening a pull request (the same checks CI runs in `.github/workflows/ci.yml`).

---
> Source: [QuantumBFS/sci-brain](https://github.com/QuantumBFS/sci-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
