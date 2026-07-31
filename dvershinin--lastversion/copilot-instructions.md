## lastversion

> Project maintenance standards (Always)


- Keep these rules up to date. When you add architecture decisions, new modules/services, or workflows, update or add a rule file so future iterations are faster and consistent.
- Use /Generate Cursor Rules after major changes to persist new guidance.
- Split large rules into smaller, focused rules (under ~500 lines).
- Reference files with @ so current code is attached to the context when the rule triggers.
- For PRs: summarize rule updates in the description; new features MUST update or add rules alongside tests.
- Prefer nested rules (`.cursor/rules` in subdirectories) for module-specific guidance; they’ll auto-attach when those files are in context.

---
> Source: [dvershinin/lastversion](https://github.com/dvershinin/lastversion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
