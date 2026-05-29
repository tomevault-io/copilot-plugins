## howaboua-pi-stuff

> This repo publishes with Changesets. Do not write "upcoming release", "unreleased", or speculative future release notes.

# AGENTS.md

This repo publishes with Changesets. Do not write "upcoming release", "unreleased", or speculative future release notes.

When a change is meant to ship, add a changeset for the package(s). On merge to `main`, the version PR and publish workflow turn that into the next concrete version immediately.

Use concrete version language only:

- good: `0.0.1 initial release`
- good: `adds a patch changeset for @howaboua/pi-vent`
- bad: `upcoming release`
- bad: `unreleased changes`

For package work, prefer:

```bash
bun run check:changed
bun changeset
```

Do not manually bump aggregate package versions. CI derives aggregate changesets with:

```bash
bun run changeset:aggregates
```

---
> Source: [IgorWarzocha/howaboua-pi-stuff](https://github.com/IgorWarzocha/howaboua-pi-stuff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
