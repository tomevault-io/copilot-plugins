## discussion-design-docs

> Where to put internal discussion and design documentation


# Discussion & design documentation

## Location

Create and maintain **discussion**, **design**, and **RFC / ADR-style** notes under:

**`.discussion-docs/`** (repository root)

Use **descriptive kebab-case** filenames (e.g. `rale-node-lookup.md`, `feature-foo-design.md`).

## What belongs here

- Exploratory write-ups, options, and tradeoffs before implementation  
- Design docs that describe *how* something should work in this repo (not end-user product help)  
- Short internal specs that are not the primary shipped `docs/` site

## What does not belong here

- End-user or public documentation that should live under **`docs/`** or package READMEs (keep those in their existing homes)  
- Generated or build output

When a design graduates to “shipped” user-facing documentation, mirror or move the relevant parts into **`docs/`** as appropriate.

---
> Source: [paramount-engineering/roku-dev-studio](https://github.com/paramount-engineering/roku-dev-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
