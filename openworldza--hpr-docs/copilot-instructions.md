## hpr-docs

> Markdown documentation site for the HPR (Hunt, Point, Retrieve) pedigree registry. Deployed to GitHub Pages via Jekyll.

# HPR Registry Documentation

Markdown documentation site for the HPR (Hunt, Point, Retrieve) pedigree registry. Deployed to GitHub Pages via Jekyll.

## Project structure

All docs live in `docs/`. Key files:

- `index.md` — landing page and navigation table
- `features.md` — feature overview and supported qualifications/breeds
- `flow.md` — data flow diagrams (mermaid) with explanations
- `getting-started.md` — account creation and onboarding
- `user-guide.md` — member guide (submitting dogs, searching, COI calculator)
- `kennel-owner-guide.md` — kennel owner guide (dog forms, breeders, bulk upload)
- `admin-guide.md` — admin guide (approval queue, roles, bulk upload, config)
- `user-journeys.md` — personas and key concepts
- `roles.md` — RBAC role and permission reference
- `roadmap.md` — roadmap and feature requests
- `_config.yml` — Jekyll config (Just the Docs theme)

## Display name rule

A dog's display name is computed from its registered name plus approved qualifications:

- **Champion titles** → prefix before name (e.g. "Ch. Kennel von Haus Example")
- **All other qualifications** → suffix after name (e.g. "Kennel von Haus Example VGP HZP")
- **Combined:** "Ch. Kennel von Haus Example VGP SA-HZP"
- Separator is space (not hyphen)
- Backend field: `title_position: 'none' | 'prefix' | 'suffix'`
- Admin UI: "Title position" dropdown (not a toggle)

## Conventions

- All content is Markdown with YAML frontmatter (`title`, `nav_order`)
- Diagrams use Mermaid fenced code blocks
- Internal links use relative paths (e.g. `[User guide](user-guide.md)`)
- British English spelling (e.g. "colour", "organisation")

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.

---
> Source: [openworldza/hpr-docs](https://github.com/openworldza/hpr-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
