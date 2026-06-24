## mini-ai-marketing-agency

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mini AI Marketing Agency Assistant — a static web app (HTML, CSS, vanilla JS) that generates a starter marketing package from a business brief form. No backend, no frameworks, no API keys. Rule-based JavaScript generation only. See `spec.md` for full requirements.

## How to Run / Develop

```bash
# Open directly in browser — no build step, no dev server
open index.html

# Or serve locally if you prefer
python3 -m http.server 8000
```

No lint, test, or build commands exist — this is a flat static site.

## Architecture

**Single-page app:** `index.html` contains both the form and the results section. The form accepts 8 fields (business name, type, product/service, target audience, main offer, platform, tone, language). On submit, `script.js` reads form values, runs rule-based generation, and renders results into the DOM.

**Generation engine (`script.js`):** Template banks are nested objects keyed by `[language][tone][section]` plus separate banks for content pillars (keyed by business type) and ad copy (keyed by `[platform][tone]`). Generation flow: validate → build context object → pick templates from matching pools → interpolate `{placeholders}` → render 6 output blocks → scroll to results.

**Styling (`style.css`):** Clean card-based layout with a neutral palette and one accent color. Responsive for mobile and desktop. Uses `navigator.clipboard.writeText()` for copy buttons. Burmese support via Noto Sans Myanmar or system font stack.

## Ch-3 Deliverables

The project requires these additional files beyond the app source:

| File | Purpose |
|------|---------|
| `.mcp.json` | MCP configuration |
| `.claude/skills/marketing-strategy/SKILL.md` | Skill definition for marketing strategy |
| `.claude/agents/marketing-assistant.md` | Agent definition |
| `slides/pitch.md` | Pitch deck outline |
| `README.md` | Project overview + 3-step commit guide |

## Key Constraints

- No backend, no login, no payment, no API keys — purely client-side.
- Output must vary when tone, platform, or language changes (different template pools).
- At least one Burmese (`my`) language output path must work.
- All 6 output sections must generate: brand positioning, 3 content pillars, 5 post ideas, 3 captions, 2 ad copies, 1 creative headline.

---
> Source: [rover-aungkhine/mini-ai-marketing-agency](https://github.com/rover-aungkhine/mini-ai-marketing-agency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
