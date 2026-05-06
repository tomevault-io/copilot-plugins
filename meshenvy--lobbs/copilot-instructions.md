## commit-message-convention

> Monorepo conventional commit message format


# Commit Message Convention

Use monorepo-style conventional commit messages for all git commits.

## Format

`type(scope): subject`

- `type` is a conventional commit type like `feat`, `fix`, `chore`, `docs`, `refactor`, or `test`.
- `scope` should identify the monorepo package or area (for example `lotato`, `lo-star`, `meshtastic`).
- `subject` must be concise, imperative, and lowercase when possible.

## Limits

- Keep the full commit subject line to **50 characters or fewer**.

## Examples

- `feat(lotato): add meshcore cli gateway`
- `fix(lo-star): correct readme typo`

---
> Source: [MeshEnvy/lobbs](https://github.com/MeshEnvy/lobbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
