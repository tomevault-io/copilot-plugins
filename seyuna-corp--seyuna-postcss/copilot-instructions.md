## seyuna-postcss

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build        # Compile TypeScript to dist/ and copy styles
npm run dev          # Watch mode TypeScript compilation
npm run test:run     # Run tests once (CI)
npm test             # Run tests in watch mode
```

Run a single test by name:
```bash
npm run test:run -- -t "SeyunaTone"
```

## Architecture

`@seyuna/postcss` is a PostCSS plugin that compiles Seyuna-specific CSS syntax into standard CSS. It enables theme-aware OKLCH colors, light/dark mode handling, container query shortcuts, and palette iteration — all resolved at build time with no runtime JavaScript.

### Processing Pipeline

The plugin's `Once()` hook (in `src/plugin.ts`) loads configuration, then processes the AST in two passes:

1. **At-rule pass** — handlers in `src/at-rules/` transform or remove custom at-rules. Execution order matters and is defined in `src/at-rules/index.ts`:
   - `@config "seyuna"` — parsed into a `SeyunaConfig` object
   - `@seyuna;` — injects CSS variables, resets, and light/dark mode rules
   - `@light` / `@dark` — wraps content in mode selectors and `prefers-color-scheme` media queries
   - `@xs` / `@sm` / `@md` / `@lg` / `@xl` / `@2xl` — expanded to `@container (min-width: …)` rules
   - `@each-tone` / `@each-swatch` — clone and repeat blocks per palette entry

2. **Function pass** — `src/parser.ts` walks all declarations, uses `postcss-value-parser` to find Seyuna function calls, and replaces them with OKLCH values or config lookups. Functions are registered in `src/functions/index.ts`:
   - `SeyunaTone`, `SeyunaSwatch` — standard OKLCH colors from the active palette
   - `SeyunaAlpha`, `SeyunaTint`, `SeyunaShade`, `SeyunaContrast` — color variants

### Configuration Model

The `@config "seyuna"` block is a CSS rule containing CSS custom properties. `src/at-rules/config.ts` reads these properties using prefix patterns:

| Prefix | Purpose |
|--------|---------|
| `--hue-*` | Maps color name → hue angle (OKLCH H) |
| `--color-*` | Fixed colors (not palette-driven) |
| `--light-*` / `--dark-*` | Palette lightness, chroma, and surface colors per mode |

### Key Files

| File | Role |
|------|------|
| `src/plugin.ts` | Plugin orchestrator; manages `PluginContext` lifecycle |
| `src/parser.ts` | Declaration function parser using `postcss-value-parser` |
| `src/types.ts` | All TypeScript interfaces (`Color`, `Palette`, `Theme`, `PluginContext`, etc.) |
| `src/config.ts` | Config loading and defaults |
| `src/at-rules/import.ts` | Injects CSS variables, surface colors, resets, and mode rules |
| `src/functions/color.ts` | OKLCH variable generation; distinguishes standard vs fixed colors |
| `src/styles/seyuna-global.css` | CSS reset + `@layer` definitions; copied to `dist/styles/` at build time |

### Testing

Tests use Vitest with `postcss.process()` for integration-style testing. Mock config objects are passed directly to the plugin. Test file: `tests/plugin.test.ts`.

---
> Source: [seyuna-corp/seyuna-postcss](https://github.com/seyuna-corp/seyuna-postcss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
