## globalattack

> Always-on instructions for working in the globalattack project


# Global Attack Review Platform

This is a client-side HTML application for reviewing global events and security incidents. All code resides in a single `index.html` file with embedded CSS and JavaScript.

## Key Guidelines

- **Single File Architecture**: Modify [index.html](index.html) for all changes. No external CSS/JS files.
- **Validation First**: Run `npm test` after any HTML changes to ensure compliance with html-validate rules.
- **Client-Side Only**: No backend or persistence. Form submissions exist only in memory.
- **Semantic HTML**: Use proper semantic elements and maintain accessibility standards.
- **Vanilla JavaScript**: Keep code simple, functional, and compatible with modern browsers.

## Common Patterns

- Form handling: Use `addEventListener` with `preventDefault()` for submissions.
- Dynamic content: Append new elements to DOM containers like `#reviews`.
- Styling: Embedded CSS in `<style>` block; consider extracting if project grows.

## Pitfalls to Avoid

- HTML linting errors: Check `npm test` output for validation issues.
- Missing form validation: Ensure all required fields are validated client-side.
- Broken links: Verify document references in `#documents` section exist.

See [README.md](README.md) for project overview and [.github/agents/globalattack-assistant.agent.md](.github/agents/globalattack-assistant.agent.md) for specialized assistance.

---
> Source: [red19jag-blip/globalattack](https://github.com/red19jag-blip/globalattack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
