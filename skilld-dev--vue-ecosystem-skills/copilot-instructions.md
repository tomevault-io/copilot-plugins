## vue-ecosystem-skills

> See [README.md](README.md) for the full list of 33 available skills with versions.

# Vue Ecosystem Skills — Development Guide

## Skill Scopes

See [README.md](README.md) for the full list of 33 available skills with versions.

## Skill Structure

Each skill follows a consistent structure:

```
skills/<name>/
├── SKILL.md                    # Overview with API changes + best practices
└── references/
    ├── docs/_INDEX.md           # Official docs reference index
    ├── issues/_INDEX.md         # GitHub issues reference index
    ├── discussions/_INDEX.md    # GitHub discussions reference index
    └── releases/_INDEX.md       # Release notes reference index
```

## Guidelines

- SKILL.md must stay under 500 lines — use references for deep-dive content
- API Changes section: only version-specific additions and breaking changes with `[source]` links
- Best Practices section: patterns derived from official docs, not general JS knowledge
- Reference files must link back to their original source (docs URL, issue number, etc.)
- All code examples must be TypeScript + Composition API unless the skill specifically targets Options API

---
> Source: [skilld-dev/vue-ecosystem-skills](https://github.com/skilld-dev/vue-ecosystem-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
