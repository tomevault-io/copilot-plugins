## figma-aso-localizer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ASO Screenshot Localizer v2 - A Figma plugin that automates localization of App Store Optimization screenshots. It extracts text from Figma frames, translates them via AI (OpenRouter API), and creates localized copies with translated text.

## Tech Stack

- **Platform:** Figma Plugin API v1.0.0
- **Language:** Vanilla JavaScript (ES5 compatible, no transpilation)
- **UI:** Plain HTML/CSS embedded in ui.html
- **Translation API:** OpenRouter.ai (supports Gemini, GPT, Claude models)

## Project Structure

```
manifest.json    # Plugin metadata, entry points, network permissions
code.js          # Backend - Figma API interactions (frame cloning, text replacement)
ui.html          # Frontend - UI + all client-side logic including API calls
```

No build process, no npm dependencies, no package.json.

## Development

### Testing Changes

Load the plugin in Figma:
1. Figma → Plugins → Development → Import plugin from manifest
2. Select the `manifest.json` file
3. Run via Plugins menu or right-click → Plugins

Changes to code.js or ui.html take effect on plugin restart.

### Debugging

- Use `console.log()` in code.js → visible in Figma's DevTools (Plugins → Development → Open Console)
- Use `console.log()` in ui.html → visible in plugin UI's DevTools (right-click plugin window → Inspect)

## Architecture

### Two-Process Model

1. **Backend (code.js):** Runs in Figma's sandbox, has access to document API
2. **Frontend (ui.html):** Runs in iframe, handles UI and external API calls

Communication via `figma.ui.postMessage()` (backend→frontend) and `parent.postMessage()` (frontend→backend).

### Core Workflow

1. User selects frames → clicks "Scan"
2. Backend traverses frame hierarchy via `collectTextNodes()`, sends text data to UI
3. User configures translation settings, clicks "Translate"
4. UI calls OpenRouter API, stores translations in `translations` object
5. User clicks "Create" → backend clones frames, replaces text via path-based node mapping

### Key Implementation Details

- **Font loading:** Fonts must be loaded async before modifying text nodes
- **Node mapping:** Uses parent-child index paths (not IDs) to find corresponding nodes in cloned frames
- **Settings persistence:** localStorage under key `aso_localizer_settings`
- **Translation parsing:** Regex extracts JSON array from AI response to handle markdown wrapping

## Important Constraints

- ES5 syntax only (Figma's embedded Chromium)
- Cannot access fonts not in user's Figma workspace
- Mixed fonts in text nodes require per-character font handling
- UI text is in Russian; code identifiers in English

---
> Source: [sabrysmk/figma-aso-localizer](https://github.com/sabrysmk/figma-aso-localizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
