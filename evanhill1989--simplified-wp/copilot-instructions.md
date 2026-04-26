## simplified-wp

> A portfolio site built as a WordPress child theme of Twenty Twenty-Five, where TT25 provides structural scaffolding and our custom CSS system owns all visual design, layout, and responsive behavior.

# Simplified WP — Project Instructions

## Mission

A portfolio site built as a WordPress child theme of Twenty Twenty-Five, where TT25 provides structural scaffolding and our custom CSS system owns all visual design, layout, and responsive behavior.

## Architecture Philosophy

- WordPress is a CMS and templating engine. It is not our design system.
- `theme.json` defines design **tokens** (colors, type scale, spacing scale). Custom CSS defines how and when tokens are **applied**.
- The block editor offers **guided freedom**: editors choose from curated options, CSS has final visual authority.
- CSS methodology: **Hybrid** — BEM for components, utility classes for layout/spacing/responsive helpers.
- Responsive: **Mobile-first**, three breakpoints (768px / 1024px / 1440px).

## Non-Negotiable Rules

1. **No code auto-modification.** All code is delivered for manual copy/paste with exact file paths and placement context. Never write to project files unless explicitly instructed.
2. **Read before modifying.** Never propose changes to code you haven't read.
3. **No over-engineering.** Only make changes that are directly requested or clearly necessary. No speculative abstractions, no feature flags, no backward-compatibility shims.
4. **No security vulnerabilities.** Escape all output, sanitize all input, use nonces for forms. No raw SQL, no `eval`, no unescaped user data.
5. **Single-project scope.** Decisions are optimized for this portfolio site. Do not abstract for hypothetical reuse.
6. **No dependency on WordPress DOM structure.** Do not rely on WordPress core block class names or DOM hierarchy as stable styling hooks. Override them in the normalization layer (`_reset.css`) if needed, but all component and layout styles must target project-owned classes (`c-`, `u-`, `l-`).

## Architecture Documents

Detailed guidance lives in `/docs/architecture/`:

- [CSS Architecture](docs/architecture/css-architecture.md) — Methodology, naming, file organization
- [Responsive System](docs/architecture/responsive-system.md) — Breakpoints, media queries, approach
- [theme.json Strategy](docs/architecture/theme-json-strategy.md) — Token ownership, presets, settings
- [WordPress Engineering](docs/architecture/wordpress-engineering.md) — Templates, PHP, child theme overrides
- [Gutenberg Boundaries](docs/architecture/gutenberg-boundaries.md) — Editor controls, block restrictions
- [Workflow Bridge](docs/architecture/workflow-bridge.md) — Decision rules for editor vs CSS vs PHP

Detailed brand guidance lives in `/docs/brand/`:

- [Brand Guidelines](docs/brand/brand-guidelines.md) — Decision rules for on brand styling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evanhill1989) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
