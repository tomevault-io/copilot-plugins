## caliber-pre-commit

> Run Caliber sync before git commits to keep agent configs up to date

Before every git commit, check if the Caliber pre-commit hook is installed:
Run: `grep -q "caliber" .git/hooks/pre-commit 2>/dev/null && echo "hook-active" || echo "no-hook"`

If hook-active: just commit normally — Caliber syncs automatically via the hook. Tell the user: "Caliber will sync your agent configs automatically."
If no-hook: run `caliber refresh && git add CLAUDE.md .claude/ .cursor/ .cursorrules .github/copilot-instructions.md .github/instructions/ AGENTS.md CALIBER_LEARNINGS.md 2>/dev/null` before committing, and tell the user what was updated.

If `caliber` is not found, read the setup-caliber skill from .cursor/skills/setup-caliber/SKILL.md and follow its instructions to install Caliber.

---
> Source: [caliber-ai-org/ai-setup](https://github.com/caliber-ai-org/ai-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
