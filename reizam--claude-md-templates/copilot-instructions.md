## claude-md-templates

> - DRY: extract repeated logic into reusable functions/modules

# Global CLAUDE.md — ~/.claude/CLAUDE.md
# Applies to ALL your projects. Keep it short (~30 lines).
# Rule: only write what the agent CANNOT discover on its own.

# Philosophy
# [Your core engineering values — shapes HOW the agent thinks, not just what it does.]
# [Test: "Would two reasonable engineers disagree on this?" If not, too generic.]
# [BAD: "Write clean code" — no one disagrees. GOOD: "YAGNI: no speculative abstractions".]
- DRY: extract repeated logic into reusable functions/modules
- YAGNI: only implement what's needed NOW — no speculative abstractions
- Elegant code: favor simplicity and clarity over cleverness
- Fix root causes, never apply band-aid solutions that create tech debt

# Language & response style
Always respond in English.
Generate code, comments, and commits in English.

# Code conventions
- Naming: files in kebab-case, variables in camelCase, components in PascalCase
- Strict TypeScript: prefer `unknown` over `any`, no implicit returns

# Tools
- Package manager: pnpm (never npm or yarn)
- Test runner: vitest
- Run tests after every change

# Git
- Conventional commits: feat(scope): msg, fix:, refactor:, chore:
- Never amend published commits
- Never force push to main

# Workflow
- Read files before editing — understand context first
- One task at a time: finish and verify before starting the next
- Do not add explanations after completing work unless asked

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reizam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/reizam)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
