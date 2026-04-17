## spectrum-eager

> Jekyll site with section-based architecture and YAML-first customization


# Jekyll Section-Based Architecture Philosophy

## Core Principles

**Section-based organization** - Sections are defined in `_sections/` with config files. All content lives in `_posts/` and uses a `section` field to indicate which section it belongs to.

**YAML-first customization** - Non-technical users should be able to add sections and customize the site by editing only YAML/Markdown files, never touching HTML/CSS/JS.

**Automatic discovery** - The site automatically discovers and displays sections from `_sections/` folders. Adding a new section requires only creating its config file.

**Zero hardcoding** - Never hardcode section names, content, or settings in templates. Everything must be configurable through YAML.

## Key Architecture Decisions

- **Unified post collection**: All posts live in `_posts/`, not separate collections per section
- **Section filtering**: Use `section` field in post front matter to categorize content
- **Config-driven sections**: Each section's display settings live in `_sections/<name>/config.yml`
- **Modular templates**: Sections can use different template types based on `type` field

## Adding New Sections

1. Create `_sections/new-section/config.yml` with section settings
2. Add posts to `_posts/` with `section: "new-section"`
3. Section appears automatically on homepage (if `enabled: true`)

## Template Guidelines

- Filter posts by section: `{% assign posts = site.posts | where: "section", section.key %}`
- Handle empty sections gracefully with fallback messages
- Support toggle flags: `enabled: true/false` for all optional features
- Use `| default:` filters for all YAML values

## Quality Standards

- Can non-technical users add content/sections without code changes?
- Does it work with 0, 1, and 100+ posts in a section?
- Is mobile responsive?
- Are defaults sensible when YAML values are missing?

## MUST
- WRITE CODE IN ENGLISH ONLY.
- COMMENTS ALSO ENGLSIH.

## Forbidden

- DO NOT! Hardcoding section lists
- DO NOT! Separate collection folders per section
- DO NOT! Requiring server restart for content changes
- DO NOT! Inline styles/scripts in content files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dudududukim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
