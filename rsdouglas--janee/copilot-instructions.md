## janee

> Guide for AI agents working on Janee.

# AGENTS.md

Guide for AI agents working on Janee.

## What Must Never Be Forgotten

### Security Guardrails
This is a **security product**. Non-negotiable rules:
- Never log credentials, tokens, or private keys
- Secrets must be encrypted at rest
- Validate all inputs (especially file paths and service names)
- Document security implications in PRs

See `docs/POLICIES.md` for full security policies.

### Documentation Requirements
**Always update:**
- `docs/CHANGELOG.md` — Add entry under `[Unreleased]`
- Tests for new features or bug fixes

**Update only if behavior changes:**
- `README.md` — New CLI commands, features, config options
- `SKILL.md` — Agent-facing changes (new tools, auth types)
- `docs/` — Significant features or architecture changes
- RFC status — Update from Draft → Implemented

See `docs/CONTRIBUTING.md` for full PR checklist.

## Release Process

### 1. Bump Version
```bash
npm version patch   # X.Y.Z → X.Y.(Z+1) — bug fixes
npm version minor   # X.Y.Z → X.(Y+1).0 — new features
npm version major   # X.Y.Z → (X+1).0.0 — breaking changes
```

### 2. Update Changelog
Move `[Unreleased]` entries to new version section:
```markdown
## [Unreleased]

_(empty)_

## [X.Y.Z] - YYYY-MM-DD

### Fixed
- Bug fix description (#issue)
```

Commit separately:
```bash
git add docs/CHANGELOG.md
git commit -m "docs: Update changelog for vX.Y.Z"
```

### 3. Publish
```bash
git push && git push --tags
npm publish --access public
```

### 4. Verify
```bash
npm view @true-and-useful/janee version
```

## Repo Invariants

- **Tests:** Colocated with source (`*.test.ts`)
- **Agent docs:** `SKILL.md` (usage), `AGENTS.md` (this file, development)
- **Preferred commit prefixes:** `feat:`, `fix:`, `docs:`, `test:`

---
> Source: [rsdouglas/janee](https://github.com/rsdouglas/janee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
