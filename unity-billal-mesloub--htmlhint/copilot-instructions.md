## htmlhint

> General rules for the project


- When adding a new rule to website ensure that it has the frontmatter: `pagefind: false   sidebar:   badge: New  hidden: true`.
- Always run `npm run lint` before declaring that a task is complete (if you've changed any files).
- Always use lf for line endings.
- Code is formatted with prettier.
- As a general rule, rules should be listed alphabetically.

## GitHub Actions

- The GitHub Actions workflows should be placed in the .github/workflows directory.
- The workflows should be named `<workflow-name>.yml`.
- All GitHub Actions should be pinned versions to avoid breaking changes (SHA-1).
- If using actions/checkout, it should have `persist-credentials: false` set.

## Documentation

- All documentation should be written in US English.
- For the 'Why this rule is important' section, always include a link to the relevant documentation and prefer MDN and Google documentation along with official documentation from the W3C.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Unity-Billal-mesloub)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Unity-Billal-mesloub)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
