## popart

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PopArt is an AI-powered infinite canvas web application for image generation and editing. It uses tldraw as the canvas SDK with AI image generation (Gemini & Seedream models).

## Commands

```bash
npm run dev      # Start dev server on port 3000
npm run build    # TypeScript check + Vite build
npm run lint     # ESLint
npm run preview  # Preview production build
```

## Architecture

### Core Stack
- **Canvas**: tldraw v2 - handles infinite canvas, pan/zoom, shape management, persistence
- **AI Image Generation**: Gemini & Seedream models
- **State**: Zustand for AI generation state; tldraw manages canvas state with localStorage persistence
- **Styling**: TailwindCSS

### Key Architectural Decisions

1. **tldraw Integration**: The app uses tldraw's `TLComponents` system to inject custom UI (`InFrontOfTheCanvas`) for the floating toolbar and prompt panel. The `useEditor()` hook provides access to the tldraw editor instance.

2. **Image Assets**: All images are stored as base64 data URLs in tldraw's asset store (`src/utils/imageAssets.ts`). The `createImageAssetStore()` function handles image uploads and resolution.

3. **AI Services**: Located in `src/services/ai/`:
   - `imageGeneration.ts` - Gemini & Seedream models

4. **UI Components**: Main floating UI components:
   - `BottomPromptPanel` - AI generation input with model selection and reference image support
   - `FloatingToolbar` - Image actions (copy, download) positioned above selected image
   - `SettingsModal` - API Key configuration modal

5. **API Key Management**: Users must configure their own API Key via the settings modal (gear icon at bottom right). Keys are stored in localStorage (`src/utils/apiKeyStorage.ts`).

### Data Flow for Image Generation

1. User enters prompt in `BottomPromptPanel`
2. `useAIStore.generateImage()` calls `imageGeneration.ts`
3. Base64 response converted to data URL
4. `addImageToCanvas()` creates tldraw asset + shape

### API Key Configuration

Users configure their API Key through the in-app settings modal (no environment variables needed). The key is stored in localStorage.

## Code Patterns

### Working with tldraw Editor

```typescript
const editor = useEditor()

// Create image asset
editor.createAssets([{ id: assetId, type: 'image', ... }])

// Create image shape
editor.createShape({ type: 'image', x, y, props: { assetId, w, h } })

// Get selected shapes
editor.getSelectedShapes()

// Convert coordinates
editor.pageToScreen({ x, y })
editor.getViewportScreenCenter()
```

### Adding Images to Canvas

Use `addImageToCanvas()` from `src/utils/imageAssets.ts` - handles asset creation, dimension scaling, and positioning.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Howell5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
