## oh-my-ai-diary

> - Asked to **install** this? Follow [`INSTALL-AI.md`](INSTALL-AI.md) step by step.

# For AI agents opening this repository

- Asked to **install** this? Follow [`INSTALL-AI.md`](INSTALL-AI.md) step by step.
- Asked to **remove** it? `INSTALL-AI.md` has the uninstall section. Never touch the user's storybook vault.
- Asked to **develop or modify** this? Rules:
  - [`DESIGN.md`](DESIGN.md) is the authoritative spec. Read it before architectural changes.
  - `npm test` and `npm run build` must pass before you are done.
  - The behavior contract has one source of truth: `adapters/generic/INSTRUCTIONS.md`. The Claude Code
    skill body is generated from it at build time; edit the source, then run `npm run build`.
  - Writing style everywhere in this repo: no em-dashes (U+2014), in prose, docs, commit messages,
    code comments, and generated output. Use a comma, colon, parentheses, or split the sentence.
    If a regex or fixture must represent a literal em-dash, spell it as a Unicode escape in source.

---
> Source: [chldbwnstm/oh-my-AI-diary](https://github.com/chldbwnstm/oh-my-AI-diary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
