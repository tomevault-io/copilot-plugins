## idd

> This repo treats Markdown formatting as a first-class interface.

# Agent Notes

## Markdown linting

This repo treats Markdown formatting as a first-class interface.

Run markdownlint across the repo (line-length is intentionally not enforced):

- `npx --yes markdownlint-cli --disable MD013 -- "**/*.md"`

### Common fixes

- Placeholders like `<name>` are parsed as inline HTML by markdownlint. Prefer `` `<name>` `` instead of raw `<name>`.
- Fenced code blocks should have a language (` ```bash `, ` ```md `, ` ```text `, etc.) and be surrounded by blank lines.
- Ensure each Markdown file starts with a single `# ...` heading.
- Avoid duplicate headings within a file.

## Commits

This repo uses DCO sign-offs for commits:

- Use `git commit -s ...`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neutral)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neutral)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
