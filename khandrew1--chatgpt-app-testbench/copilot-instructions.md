## chatgpt-app-testbench

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
npm run build      # Build widgets (esbuild + Tailwind CSS)
npm run dev        # Start local Wrangler dev server
npm run deploy     # Deploy to Cloudflare Workers
npm test           # Run tests with Vitest
npm run cf-typegen # Regenerate Cloudflare types
```

## Architecture Overview

This is a ChatGPT Apps (widgets) template using MCP server on Cloudflare Workers. The system has two main parts:

### ChatGPT Apps SDK
ChatGPT App SDK documentation can be found here using context7: https://developers.openai.com/apps-sdk
For a quick reference to the `window.openai` API you can use this: https://developers.openai.com/apps-sdk/reference#windowopenai-component-bridge

**Widget Development (`web/`)**: React components built with esbuild, styled with Tailwind CSS and OpenAI Apps SDK UI components (`@openai/apps-sdk-ui`). Each widget is a standalone `.tsx` file in `web/widgets/` that mounts itself via `createRoot()`.

**MCP Server (`server/`)**: Cloudflare Worker that serves widgets as MCP resources. Widget registration is declarative via `WIDGET_CONFIGS` array in `server/widgets/config.ts`.

### Request Flow

1. ChatGPT requests a widget resource (e.g., `ui://widget/reservation-card.html`)
2. `server/widgets/registry.ts` handles the resource request
3. `server/widgets/utils.ts` loads assets from `web/dist/` via Cloudflare Assets binding
4. Returns HTML with globals.css, widget CSS, and widget JS embedded inline

### Key Conventions

- Widget naming: kebab-case matching filename (e.g., `flight-status.tsx` → `name: 'flight-status'`)
- Root element ID: `{widget-name}-root` convention (e.g., `flight-root`)
- Widget URI format: `ui://widget/{name}.html`
- Widgets use OpenAI design tokens: `border-default`, `bg-surface`, `text-secondary`, etc.

## Adding a New Widget

1. Create `web/widgets/{name}.tsx` with React component + `createRoot()` mount
2. Add build script to `web/package.json`
3. Add config to `WIDGET_CONFIGS` in `server/widgets/config.ts`
4. Optionally register a tool in `server/index.ts` with `openai/outputTemplate` pointing to widget URI

Always use the OpenAI developer documentation MCP server if you need to work with the OpenAI API, ChatGPT Apps SDK, Codex,… without me having to explicitly ask.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khandrew1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/khandrew1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
