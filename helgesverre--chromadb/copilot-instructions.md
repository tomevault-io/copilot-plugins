## chromadb

> This file contains project-specific preferences and guidelines for Claude Code when working on this repository.

# Claude Code Guidelines for This Project

This file contains project-specific preferences and guidelines for Claude Code when working on this repository.

## Release Management

### GitHub Releases

**Release Titles:**

- Use **version number only**: `v2.0.0`
- Do NOT add emojis or extra text to the title
- Save emojis and descriptive text for the release body/notes

**Example:**

```bash
# ✅ Correct
gh release create v2.0.0 --title "v2.0.0"

# ❌ Incorrect
gh release create v2.0.0 --title "v2.0.0 - Complete ChromaDB v2 API Support 🚀"
```

### Git Tags

**Tag Format:**

- Use annotated tags: `git tag -a v2.0.0 -m "..."`
- Tag message can include emojis and descriptions
- Tag name should be version only: `v2.0.0`

### Versioning

- Follow [Semantic Versioning](https://semver.org/)
- MAJOR.MINOR.PATCH format (e.g., `2.0.0`)
- Always prefix with `v` in git tags and releases

## General Preferences

### Code Style

- Follow existing code style in the project
- Use PHP 8.2+ features
- Named parameters preferred for method calls

### Documentation

- Keep documentation in sync with code
- Update CHANGELOG.md for all releases
- Include migration guides for breaking changes

### Commit Messages

- Use conventional commits format
- Include emojis in commit messages is acceptable
- Keep messages clear and descriptive

---

_Last updated: 2025-10-22_

---
> Source: [HelgeSverre/chromadb](https://github.com/HelgeSverre/chromadb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
