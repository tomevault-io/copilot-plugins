## designkit

> This repo is a Claude Code plugin. The `skills/designkit/` directory contains the skill and all supporting scripts. **Do not modify the scripts unless explicitly asked** — they are the shipped tool, not generated artifacts.

# Design Companion — Plugin Guidelines

## If You Are Claude

This repo is a Claude Code plugin. The `skills/designkit/` directory contains the skill and all supporting scripts. **Do not modify the scripts unless explicitly asked** — they are the shipped tool, not generated artifacts.

When the designkit skill is activated, follow the instructions in `skills/designkit/SKILL.md` exactly.

## Prototype Authoring Standards

When generating HTML prototypes for the Design Companion:

1. **Use CSS classes, not inline styles.** The Tune panel's "Apply to all" matches elements by class.
2. **Use CSS custom properties (tokens).** The Tune panel detects tokens and adjusts them globally.
3. **Use semantic class names.** `.metric-card` not `.p-4.bg-white`.
4. **Always include a token block** in the `<style>` at the top of the prototype.

## Selector Namespacing

The Design Companion's own UI uses `#dc-` prefixed IDs and `.annotation-` prefixed classes. **Never use these in prototype content** — they will collide with the companion chrome.

## Contributing

- The core files are `helper.js`, `frame-template.html`, and `server.cjs` in `skills/designkit/scripts/`
- `helper.js` is the largest file (~1400 lines) — all client-side interaction logic
- `frame-template.html` is the CSS design system for the companion chrome
- `server.cjs` is a zero-dependency Node.js HTTP + WebSocket server
- The server caches nothing — edits to helper.js and frame-template.html take effect on browser refresh
- Server changes (server.cjs) require a restart

---
> Source: [leroybbad/designkit](https://github.com/leroybbad/designkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
