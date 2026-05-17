## coding-guidelines

> - We want thin files that aim to have a single responsibility and a single export.


Guiding principles:
- Clean Code
- SRP
- YAGNI
- KISS

DIRECTIVES:
- We want thin files that aim to have a single responsibility and a single export.
- We should have a single export per file.
- Files should be organized/grouped by feature
- Code should be modular
- Functions should be small
- Classes should be small
- functions, classes, methods should have TSDoc with example usage
- top-level directories represent core features, and should each have a README.md that describes the feature and its purpose.
- Never use .js extensions in import statements
- Always use package init commands rather than writing configs manually (e.g., use `npx eslint --init` or `pnpm exec husky init` instead of creating config files directly)

---
> Source: [langwatch/better-agents](https://github.com/langwatch/better-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
