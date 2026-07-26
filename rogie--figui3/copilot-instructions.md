## figui3

> Publish workflow — bump version, update changelog, commit, and push


# Publish Command

When the user says "publish", follow this exact sequence:

1. **Bump version** in `package.json` — patch by default, minor if new features, major if breaking. Ask only if ambiguous.
2. **Update `CHANGELOG.md`** — add a new version section at the top with all changes since the last publish, following the existing format (### Fixed, ### Changed, ### Added).
3. **Git add, commit, push** — stage all changed files, commit with message `v{version}: {short summary}`, and push to origin.

Keep changelog entries concise but descriptive. Group by Fixed/Changed/Added as appropriate.

---
> Source: [rogie/figui3](https://github.com/rogie/figui3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
