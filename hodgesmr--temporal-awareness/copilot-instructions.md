## temporal-awareness

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains a Claude Code skill called "temporal-awareness" that ensures temporal accuracy by using Unix date commands. The skill should be invoked whenever Claude needs date/time information: current date, day of week calculations, date arithmetic, or deadline computations.

## Structure

```
skills/temporal-awareness/SKILL.md  # Skill definition and documentation
```

## Key Concepts

- **GNU vs BSD date**: The skill handles both GNU date (Linux) and BSD date (macOS/FreeBSD), which have different syntax for date parsing and arithmetic
- **Trigger conditions**: Use this skill when users ask about dates, days of week, deadlines, durations, or any temporal context
- **Verification first**: Always run `date` commands before responding to date-sensitive queries, as Claude's system prompt date may be stale

## Releases

Releases use semver tags (e.g., `1.0.0`) plus a `latest` tag that always points to the most recent stable release.

### Release Process

1. Update the version in `skills/temporal-awareness/SKILL.md` frontmatter:
   ```yaml
   metadata:
     author: Matt Hodges
     version: "1.1.0"
   ```
2. Update `CHANGELOG.md`:
   - Add a new section at the top (below the header) for the new version
   - Use the format `## [X.Y.Z] - YYYY-MM-DD`
   - Document changes under `### Added`, `### Changed`, `### Fixed`, or `### Removed` as appropriate
   - Add a comparison link at the bottom: `[X.Y.Z]: https://github.com/hodgesmr/temporal-awareness/compare/PREV...X.Y.Z`
3. Commit the version bump and changelog update
4. Run the release script:
   ```bash
   ./release.sh
   ```

The script extracts the version from SKILL.md, creates the version tag, updates the `latest` tag, and pushes both.

---
> Source: [hodgesmr/temporal-awareness](https://github.com/hodgesmr/temporal-awareness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
