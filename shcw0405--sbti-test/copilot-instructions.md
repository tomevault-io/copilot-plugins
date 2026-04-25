## sbti-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SBTI 人格测试 — a Chinese-language interactive personality test web application. Mirror of https://sbti.unun.dev. Original author: B站@蛆肉儿串儿.

## Development

No build process, package manager, or dependencies. Open `index.html` directly in a browser. Any local HTTP server works:

```bash
python3 -m http.server 8080
# or
npx serve .
```

## Architecture

Everything lives in one file: `index.html` (~1,746 lines) with embedded CSS and vanilla JavaScript. Images are in `image/`.

**Three views** rendered in-place (no routing):
1. **Intro screen** — landing page with start button
2. **Test screen** — 30 questions rendered one at a time
3. **Result screen** — computed personality type with image, scores, and dimension breakdowns

**Scoring algorithm:**
- 15 personality dimensions across 4 axes: E (Extraversion), A (Agreeableness), Ac (Achievability), So (Sociability) — each with 3 sub-dimensions
- Each answer increments raw scores per dimension
- Euclidean distance matching against `TYPE_LIBRARY` standard patterns to find the closest personality type
- Special cases: DRUNK type triggers on specific "drunk question" answers; HHHH fallback fires when best match score < 60%

**Key JS data structures:**
- `questions` — 30 main questions, each mapped to a dimension and answer weights
- `specialQuestions` — Easter egg trigger questions (DRUNK mode)
- `TYPE_LIBRARY` — personality type definitions (code, Chinese name, description, image filename)
- `dimensionMeta` / `DIM_EXPLANATIONS` — metadata and text for each dimension and level

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shcw0405) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
