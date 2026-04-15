## devtools-suite

> This document provides a high-level map of the DevTools Suite project to help AI agents understand the codebase without full re-indexing.

# DevTools Suite - Technical Context for AI

This document provides a high-level map of the DevTools Suite project to help AI agents understand the codebase without full re-indexing.

## 🚀 Tech Stack
- **Framework**: Next.js 15 (App Router)
- **Styling**: Tailwind CSS + ShadCN UI
- **AI**: Genkit (Google AI / Gemini 2.5 Flash)
- **Security**: Node-Forge, OpenPGP, Jose, CryptoJS, noble-ciphers
- **Infrastructure**: Firebase (Analytics/App Hosting), Cloudflare Turnstile

## 📁 Project Structure
- `/src/app`: Next.js App Router pages and layouts.
- `/src/components/ui`: Reusable ShadCN components.
- `/src/components/tools`: Logic-heavy components for specific tool categories.
- `/src/lib`: Pure utility functions (The "Brain" of the tools).
- `/src/ai`: Genkit flows and prompts.
- `/src/i18n`: Multi-language JSON dictionaries (English & Indonesian).
- `/src/tools`: Master tool registry (`config.ts`).

## ⚠️ STRICT CONSTRAINTS (DO NOT MODIFY)
1. **Brand Identity**: `src/components/ui/logo.tsx` contains hardcoded inline SVG paths. **NEVER** change these paths or replace them with generic icons unless explicitly requested.
2. **Security Verification**: `src/app/networking/dns-lookup/page.tsx` uses Cloudflare Turnstile. It **MUST** use `execution: 'render'` and the `.reset()` method to prevent "400 INVALID_ARGUMENT" errors on multiple lookups.
3. **Site Config**: `src/lib/seo.ts` is the single source of truth for the site URL (`SITE_CONFIG`).
4. **Hydration**: Use the `useEffect` hook pattern for operations involving `window`, `localStorage`, or `Math.random()` to avoid hydration mismatches.

## 🛠 Adding a New Tool
1. Create logic in `src/lib/<category>.ts`.
2. Add translation keys in `src/i18n/en.json` and `id.json`.
3. Register the tool in `src/tools/config.ts`.
4. Create the page in `src/app/<category>/<tool-name>/page.tsx`.
5. Add metadata in a `layout.tsx` file within the tool folder using `generateMetadata` from `@/lib/seo`.

## 🎨 Theming
The app uses a custom theme engine in `src/app/globals.css`. 
- Base colors are defined in `:root` and `.dark`.
- Specific schemes (Catppuccin, Seasonal, Cultural) are defined using `[data-color-scheme="name"]`.
- When adding UI, always use semantic Tailwind classes (e.g., `bg-primary`, `text-muted-foreground`) to ensure theme compatibility.

## 🔍 SEO Strategy
- Every tool category has a `layout.tsx` providing metadata.
- Most tools include a `ToolSEOContent` component at the bottom for long-form educational text.
- Sitemaps are dynamically generated based on the `TOOLS` config.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gilang-as)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gilang-as)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
