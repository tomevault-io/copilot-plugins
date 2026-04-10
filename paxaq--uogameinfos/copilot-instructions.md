## uogameinfos

> - Root static site with no build step:

# Repository Guidelines

## Project Structure & Module Organization
- Root static site with no build step:
  - `index.html` — main page (Simplified Chinese content, zh-CN).
  - `styles.css` — global styles, responsive rules, animations.
  - `script.js` — vanilla JS for DOM interactions (class cards, smooth scroll, relationship rendering).
- Keep new assets in `assets/` (create if needed). Avoid restructuring or renaming existing classes/IDs.

## Build, Test, and Development Commands
- Serve locally (no dependencies):
  - `python3 -m http.server 8000` → open `http://localhost:8000`.
- Optional: use any static server; no bundlers required.

## Coding Style & Naming Conventions
- Indentation: 2 spaces for HTML/CSS/JS. UTF‑8 encoding.
- HTML: semantic sections; preserve `lang="zh-CN"` and existing anchors.
- CSS: reuse existing class patterns (e.g., `.class-card`, `.advantage-item`); prefer classes over inline styles.
- JS: ES6+, `const`/`let`, lowerCamelCase for functions; keep `classData` keys kebab-case where used.
- Filenames: lowercase with hyphens for new assets (`assets/class-icons/gryphon.png`).

## Testing Guidelines
- Manual checks in latest Chrome/Firefox/Safari and mobile viewport (≤ 375px):
  - Nav links smooth-scroll correctly; cards hover/click update the relationship panel.
  - No console errors; animations render smoothly; layout remains responsive.
- No test framework is configured; keep new logic small and easily reviewable.

## Commit & Pull Request Guidelines
- Commits: imperative and scoped. Examples:
  - `feat(ui): add damage chart hover effect`
  - `fix(data): correct gryphon-knight counters`
- PRs: clear description, before/after screenshots or GIFs for UI, linked issues, and manual test steps. See template below.

## Pull Request Template
```
Title: <type(scope): short summary>

Summary
- What and why in 1–2 sentences.

Changes
- Key changes (bulleted).

Screenshots
- Before | After at 375px, 768px, 1440px.

Testing
- Steps to reproduce + expected results.
- Browsers/devices checked.

Checklist
- [ ] No console errors
- [ ] Anchors/selectors unchanged or updated in JS
- [ ] Responsive at mobile/tablet/desktop
```

## Screenshot Guidelines
- Provide side‑by‑side before/after for visual changes at: 375×667, 768×1024, and ≥1440px widths.
- Include one interaction capture (hover/click on a `.class-card` updating `#relationshipDisplay`).
- Crop to the changed area when possible; keep file sizes small (PNG/JPEG, ≤1 MB each).

## Architecture Overview
- Data: `classData` and `relationshipData` in `script.js` drive UI content and counters.
- DOM contracts (do not break IDs/classes without updating JS): `#classSelect`, `.class-card`, `#visualization`, `#relationshipDisplay`.
- Interaction flow: card click → update select → render relationships → smooth scroll.

## Security & Configuration Tips
- Do not embed secrets or API keys. Prefer local assets over external CDNs. Keep third‑party JS minimal and vetted.

## Agent-Specific Instructions
- Do not introduce frameworks or build tooling without approval.
- Keep changes minimal, avoid breaking anchors/selectors used by `script.js`.
- Maintain Chinese UI copy unless an issue requests localization changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paxaq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/paxaq)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
