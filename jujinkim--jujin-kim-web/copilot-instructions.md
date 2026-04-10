## jujin-kim-web

> - Terminal-inspired single-page portfolio for Jujin Kim.

# Project Agent Guide

## Snapshot
- Terminal-inspired single-page portfolio for Jujin Kim.
- Pure HTML, CSS, and vanilla ES modules with no build tooling.
- Entry point: `index.html` -> `js/main.js` -> modules in `js/modules/`.
- Shared constants live in `js/constants.js` (e.g., `CONTACT_EMAIL`).

## Active Focus
- Keep the contact address authoritative via `CONTACT_EMAIL`.
- Preserve terminal aesthetic (monospace fonts, green accent palette, ASCII styling).
- Ensure content sections render via `load*Section` helpers in `js/modules/content.js`.
- Terminal input is home-only; keep commands in `js/modules/terminal-input.js` aligned with content.
- Theme switching is driven by `window.setTheme` (footer links) in `js/modules/theme.js`.

## Coordination Checklist
1. Content updates -> edit the relevant load function in `js/modules/content.js`.
2. Shared values -> prefer adding to/consuming from `js/constants.js`.
3. Terminal commands -> adjust command handlers in `js/modules/terminal-input.js`.
4. Navigation & section switching -> check `js/modules/navigation.js`.
5. Animations (starfield/matrix rain) -> update `js/modules/animations.js`.
6. Theme behavior -> update `js/modules/theme.js` (and `window.setTheme` usage).
7. Visual styling -> touch only the scoped CSS file (`css/terminal.css`, `css/responsive.css`, etc.).

## Coding Guidelines
- Stick to ES6 modules and template literals; avoid introducing frameworks.
- Use ASCII characters unless a file already mixes in Unicode (e.g., emoji in content templates).
- Keep inline comments minimal and purposeful.
- When adding files, favor descriptive names and module-scoped exports.
- Manual testing: open `index.html` in a browser or run a static server (`python3 -m http.server 8000`).

## Collaboration Notes
- Document notable deviations or new constants in this file so other agents stay aligned.
- Update `CLAUDE.md` when high-level project details shift (features, contact info, roadmap).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jujinkim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jujinkim)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
