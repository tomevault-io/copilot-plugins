## jersey-farense

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Instruções gerais

- Responder sempre em português europeu (pt-PT), incluindo mensagens de erro, comentários de código e toda a comunicação com o utilizador.
- Nunca expor chaves de API no código — usar sempre variáveis de ambiente via `.env.local`. O `.gitignore` já exclui `.env` e `.env.local`; verificar se qualquer novo ficheiro de configuração com segredos está também excluído antes de fazer commit.

## Project

React 19 + TypeScript app that composites a user's photo onto a Farense soccer jersey using Google Gemini's image generation API (`gemini-3-pro-image-preview`). The user uploads a photo, picks a historical jersey, and Gemini generates a photorealistic portrait of them wearing it.

## Commands

```bash
npm run dev              # Start dev server (http://localhost:3000)
npm run build            # Production build
npm run preview          # Preview production build
npm run generate-jerseys # Auto-generate src/jerseys.ts from public/camisolas/ and public/bolas/
```

No test runner is configured. TypeScript checking: `npx tsc --noEmit`.

## Environment

Requires `.env.local` with:
```
VITE_GEMINI_API_KEY=your_key_here
```

All env access is centralized in `src/config.ts` — never read `import.meta.env` directly elsewhere.

## Architecture

```
src/
  services/geminiService.ts   # All Gemini API logic (the core of the app)
  jerseys.ts                  # FARENSE_JERSEYS array + loadJerseys() (preloads base64)
  types.ts                    # JerseyData, GeminiResponse interfaces
  config.ts                   # getGeminiApiKey(), API_CONFIG constants
  utils/
    imageUtils.ts             # urlToBase64() — converts URLs to base64 data URIs
  hooks/
    useJerseys.ts             # Loads jersey list on mount
    useFileUpload.ts          # Handles drag-drop upload + validation
    useImageGenerator.ts      # Calls generateJerseyImage, manages loading/error state
  components/
    ImageUpload.tsx           # Upload UI
    JerseySelector.tsx        # Jersey grid picker
    SelectedJersey.tsx        # Shows selected jersey
    ResultDisplay.tsx         # Shows generated image
  App.tsx                     # Composes hooks and components, no business logic
  ErrorBoundary.tsx           # Top-level crash boundary
```

## Gemini Integration (`src/services/geminiService.ts`)

The service calls the REST API directly (not the `@google/genai` SDK, which is installed but unused). Multi-image prompt strategy, order matters:

1. **Text prompt** (first) — detailed instructions for face preservation, jersey accuracy, and emblem fidelity
2. **User photo** — the person's face to preserve
3. **Jersey image** — loaded as base64 from `/public/camisolas/`
4. **Emblem image** (optional) — from `/public/emblemas/` for badge detail
5. **Stadium image** — `/public/camisolas/estadio.png` for background
6. **Ball image** (optional) — era-appropriate ball from `/public/bolas/`

The `buildPrompt()` function dynamically adjusts positional references in the text (e.g., "third image", "fourth image") based on whether an emblem is present. If you add/remove images, update the positional references in `buildPrompt()`.

Generation config: `temperature: 0.4`, `topP: 0.8`, `topK: 40` — kept low for consistent face/jersey fidelity. Response modality is `IMAGE` only. Timeout: 60 seconds.

## Adding a New Jersey

1. Add jersey image to `public/camisolas/`
2. Add entry to `FARENSE_JERSEYS` array in `src/jerseys.ts` with `path`, `description`, `ball`, and `emblem` fields
3. The `description` field feeds directly into the Gemini prompt — be specific about colors and patterns (e.g., "blue and white vertical stripes with orange trim" rather than "Camisola histórica de XXXX")

**Warning:** `npm run generate-jerseys` auto-regenerates `src/jerseys.ts` from the filesystem but produces generic descriptions and omits the `emblem` field. Never run it without manually restoring the detailed descriptions and `emblem` references afterward.

## Assets

- `public/camisolas/` — jersey images (PNG/JPG) + `estadio.png`
- `public/bolas/` — era-appropriate ball images (WebP)
- `public/emblemas/` — club badge (`farense.png`)
- `public/exemplo4.jpeg` — default user photo loaded automatically on startup (see `App.tsx`)

## Path Aliases

`@/` maps to `src/` (configured in `vite.config.ts` and `tsconfig.json`).

## Deployment

Configured for Netlify via `netlify.toml`.

---
> Source: [DrNOFX97/Jersey-Farense](https://github.com/DrNOFX97/Jersey-Farense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
