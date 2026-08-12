## mybroworld

> - For any new chat, load `.agents/docs/chat-memory-protocol.md` after this file so the session reconstructs project memory from repository artifacts before acting.

# MyBroworld Local Rules

- For any new chat, load `.agents/docs/chat-memory-protocol.md` after this file so the session reconstructs project memory from repository artifacts before acting.
- For any task involving Google Sheets, catalog review, data normalization, completeness, blockers, readiness, or inference on spreadsheet fields, load and follow `.agents/rules/mybroworld-sheet-reviewer-evolution.md`.
- Treat new explicit user instructions, clarified decisions, and resolved edge cases in sheet-review work as candidate project criteria. If the rule is reusable beyond the current row or one-off task, update the reviewer skill before finishing the task.
- Persist reusable decisions, methodologies, and operator guidance in repository artifacts before finishing the task, following `.agents/docs/chat-memory-protocol.md`.
- When a repo-local artifact referenced by these rules is missing, look for the canonical equivalent under `~/.agents/` and `/Users/saski/Code/augmentedcode-configuration/` before treating it as unavailable. `~/.agents` is the canonical shared agent configuration and normally resolves to `/Users/saski/Code/augmentedcode-configuration/.agents`; generic skill indexes such as `.agents/docs/skill-factory-skills.md` and `.agents/docs/skill-domain-routing.md` may exist there rather than in this checkout. Launch a parallel agent to investigate the missing resource when the gap is non-trivial or affects tool/rule loading, then persist any reusable fix or discovery rule.
- At the end of each important iteration, state which skills were used during that iteration. If no skills were used, state that explicitly.
- Use RTK through the shared agent rules in `~/.agents/rules/base.md`. If `rtk` is not visible on `PATH`, try `/Users/saski/.agents/bin/rtk` or `/opt/homebrew/bin/rtk` before falling back to normal shell commands.
- If `openspec` is not visible on `PATH`, try `/opt/homebrew/bin/openspec` before concluding the OpenSpec CLI is unavailable.
- For WordPress/WooCommerce development, avoid commercial paid plugins, including freemium plugins and components whose normal path is buy-to-extend modules, aggressive upsells, or frequent paid-update churn. Prefer lean owned code, WordPress core, WooCommerce core, and the smallest unavoidable third-party dependency surface. Third-party commercial components can be considered only when they have strong reputation, low lock-in, non-aggressive commercial posture, and avoid a complicated dependency tree. Open-source plugins or add-ons may be considered only when they meet a clear quality bar and preserve the lean simplicity principle.
- For WordPress/WooCommerce theme, plugin, MU-plugin, block, REST, WP-CLI, Playground, PHPStan, or performance work in this repository, load repo-local skills from `.agents/skills/` (also linked as `.cursor/skills/` and, for WordPress skills, `.claude/skills/`). Start with `.agents/skills/wordpress-router/SKILL.md`, then follow `.agents/docs/wordpress-skills-routing.md` for domain skill selection. Provenance is recorded in `skills-lock.json`.
- Use trunk-based development for safe work: keep branches short-lived, merge verified small changes into `main`, and deploy from `main` when production checks pass.
- Keep technical artifacts in English.

---
> Source: [saski/mybroworld](https://github.com/saski/mybroworld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
