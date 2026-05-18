## webcv

> Simple static HTML CV website. No build process, no dependencies, no tests.

# AGENTS.md

## Project Overview
Simple static HTML CV website. No build process, no dependencies, no tests.

## Structure
- `index.html` - Main file containing all HTML and embedded CSS
- `css/` and `js/` - Empty directories (not used; all code is in index.html)

## Development
- Edit `index.html` directly
- Open in any browser to preview
- No build commands, no package manager, no CI/CD

## Notes
- Spanish language site (lang="es")
- Uses Google Fonts (Inter) - requires internet to render correctly

## Git
- Commitea con mensajes significativos que expliquen el "por qué", no solo el "qué"
- Haz commits frecuentes con cambios pequeños y enfocados
- Antes de commitear, revisa los cambios con `git diff`
- No commitees secretos, credenciales ni datos sensibles
- Usa ramas de características para cambios no triviales
- No hagas force push a main/master


- Commit only when explicitly requested by the user
- Commit messages: concise, present tense, describe the exact change (e.g., "Update name to Jorge Miralles")
- Never force push to main; warn user if requested
- Push to remote only when explicitly asked

- Commit Types:
Type     Purpose
feat     New feature
fix     Bug fix
docs     Documentation only
style     Formatting/style (no logic)
refactor     Code refactor (no feature/fix)
perf     Performance improvement
test     Add/update tests
build     Build system/dependencies
ci     CI/config changes
chore     Maintenance/misc
revert     Revert commit
- Generate Commit Message
    Type: What kind of change is this?
    Scope: What area/module is affected?
    Description: One-line summary of what changed (present tense, imperative mood, <72 chars)

- Execute Commit

# Single line
git commit -m "<type>[scope]: <description>"

---
> Source: [simone0780/webcv](https://github.com/simone0780/webcv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
