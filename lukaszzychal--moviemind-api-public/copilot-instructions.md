## 080-language-policy

> - `.cursor/rules/*.mdc` files must be in English.


# Language policy

## Core Rules (English)

- `.cursor/rules/*.mdc` files must be in English.
- `CLAUDE.md` must be in English.
- Code comments should be in English.

## Documentation (English + Polish)

Documentation in `docs/` should be provided in both languages when practical:

- **English (`docs/en/`):** Primary language for technical documentation
- **Polish (`docs/pl/`):** Translations for learning/reference

### Documentation Structure

- `docs/en/` - English documentation (primary)
- `docs/pl/` - Polish documentation (translations)
- `docs/knowledge/` - Technical knowledge base (can be en/pl based on content)
- `docs/issue/` - Task tracking (usually in Polish for internal use)

### When to Create Both Versions

- **Always:** API specifications, tutorials, architecture decisions
- **When practical:** Technical guides, reference documentation
- **Optional:** Internal notes, task tracking

### File Naming

- English files: `FILENAME.md` or `FILENAME.en.md`
- Polish files: `FILENAME.pl.md` or in `docs/pl/` directory
- When both exist: Keep them synchronized when content changes

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
