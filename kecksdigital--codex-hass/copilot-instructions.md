## codex-hass

> This file contains instructions for agents working on this repository.

# Repository Instructions

This file contains instructions for agents working on this repository.

## Before Every Commit

**IMPORTANT:** Update the relevant add-on `CHANGELOG.md` with the changes being committed before making any commit. Follow the existing format:

```markdown
## [VERSION] - YYYY-MM-DD

### Added/Changed/Fixed
- Description of change
```

## Project Structure

- `repository.yaml` - Add-on repository metadata
- `codex/` - Codex add-on
  - `config.yaml` - Add-on configuration (bump version here)
  - `Dockerfile` - Container build instructions
  - `build.yaml` - Multi-architecture build settings
  - `README.md` - User documentation
  - `CHANGELOG.md` - Version history (**update before commits**)
  - `apparmor.txt` - Security profile
- `auto-monocle/` - Optional camera discovery add-on
- `playwright-browser/` - Optional browser automation add-on

## Version Bumping

When making changes that require a new release:
1. Update version in the affected add-on's `config.yaml`
2. Add entry to the affected add-on's `CHANGELOG.md`
3. Commit and push

## Home Assistant Add-on Notes

- Rebuild button only rebuilds from cached config
- To pick up config.yaml changes: uninstall/reinstall or bump version and update
- Base images use s6-overlay v3 - be careful with init configuration
- `init: true` uses Docker's tini, `init: false` uses s6-overlay's /init

---
> Source: [kecksdigital/codex-hass](https://github.com/kecksdigital/codex-hass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
