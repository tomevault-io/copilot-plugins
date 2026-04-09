## ssid-docs

> - **Repo**: SSID-docs

# CLAUDE.md — SSID-docs Governance Rules

## Identity

- **Repo**: SSID-docs
- **Purpose**: Public-facing documentation site for the SSID project
- **Scope**: User documentation, guides, API reference (generated from OpenAPI), I18N content

## Write Scope

- Documentation content (MDX files)
- Astro/Starlight configuration
- I18N translation files
- Static assets for documentation

## Forbidden

- Storing system secrets in docs
- Manual API documentation (must be generated from OpenAPI)
- System-level documentation (belongs in 16_codex in SSID main repo)
- Placing PII in any documentation content
- Writing outside this repository

## Stack

- Astro + Starlight (SSG)
- MDX content format
- pnpm package manager
- I18N support required for all user-facing content

## Ports

| Service   | G-Port (Workspace) | C-Port (Canonical) |
|-----------|--------------------|--------------------|
| SSID-docs | 4331               | 4321               |

## Rules

- **SAFE-FIX**: Permanent, non-interactive, SHA256-logged write enforcement
- **NON-CUSTODIAL**: No PII in documentation, hash-only references
- **ROOT-24-LOCK**: Documentation must reflect the canonical 24-root structure accurately
- **LOCAL-FIRST**: build, test, verify, commit, push

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EduBrainBoost)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EduBrainBoost)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
