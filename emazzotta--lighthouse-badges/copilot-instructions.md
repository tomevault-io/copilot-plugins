## lighthouse-badges

> - Any commit that changes `version` in `package.json` must be tagged with that exact version.

# Code Guidelines

## Versioning

- Any commit that changes `version` in `package.json` must be tagged with that exact version.
- Tags are lightweight and carry no `v` prefix: `1.5.6`, not `v1.5.6`.
- Tag the commit that contains the bump, not a later one: `git tag 1.5.6 <sha>`.
- Never push a tag without explicit confirmation - pushing it is what cuts a release.

---
> Source: [emazzotta/lighthouse-badges](https://github.com/emazzotta/lighthouse-badges) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
