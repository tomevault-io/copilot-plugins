## cc-skills-nanobananapro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chinese New Year Card Generator - A Next.js 15 app that transforms photos into festive Lunar New Year cards using Google Gemini image generation, with an interactive 3D card preview.

## Commands

```bash
npm run dev      # Start dev server at localhost:3000
npm run build    # Production build
npm run start    # Run production server
npm run lint     # Run Next.js linter
```

## Architecture

### Data Flow
1. User uploads selfie → FileReader converts to base64
2. Server action calls Gemini API → generates 16:9 two-panel card image
3. Image displayed in UI → user can trigger 3D preview or download

### Key Files
- `app/page.tsx` - Main UI with upload zone, loading states, result display, snowfall effect
- `app/actions/generate.ts` - Server action using `gemini-3-pro-image-preview` model
- `app/components/Card3DPreview.tsx` - Three.js fold-out card with OrbitControls

### AI Model
Always use exact model string: `gemini-3-pro-image-preview` (Nano Banana Pro)

### Three.js Pattern
Uses dynamic imports (`await import('three')`) for code splitting. The 16:9 generated image is UV-mapped: left half (0-0.5) to left panel, right half (0.5-1.0) to right panel.

### Styling
Tailwind CSS extended with custom colors (burgundy, gold, forest, cream), plus CSS classes in globals.css for glassmorphism (`.glass-card`), buttons (`.btn-gold`, `.btn-burgundy`), and animations (snowfall, shimmer).

## Environment

Requires `GOOGLE_GENERATIVE_AI_API_KEY` in `.env.local`

## Available Skills

- `/frontend-design` - Create distinctive, high-quality UI
- `/nano-banana-builder` - Gemini image generation patterns
- `/threejs-builder` - Three.js scene creation

## Research Guidelines

Prefer Exa search (`get_code_context_exa` first, then `web_search_exa`). Use Ref only when explicitly asked or after Exa attempts fail.

---
> Source: [chongdashu/cc-skills-nanobananapro](https://github.com/chongdashu/cc-skills-nanobananapro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
