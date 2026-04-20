## abstime

> This file contains instructions for Claude Code to work effectively with this

# Claude Code Instructions

This file contains instructions for Claude Code to work effectively with this
project.

## Project Overview

AbsTime is a Chrome extension that converts relative dates to absolute dates on
web pages. It's built with:

- TypeScript/TSX
- SolidJS for the options page
- Vite for bundling
- Chrome Extension Manifest V3

## Development Commands

### Essential Commands

- `bun run build` - Build the extension for production
- `bun run dev` - Start development server with hot reload
- `bun run lint --fix` - Fix linting issues
- `bun typecheck` - Check TypeScript types
- `bunx prettier -w .` - Format all code

### Testing and Quality

Always run these commands before committing:

1. `bunx prettier -w .`
2. `bun run lint --fix`
3. `bun typecheck`

## File Structure

### Core Files

- `src/index.ts` - Main content script that processes web pages
- `src/dateUtils.ts` - Date parsing and formatting utilities
- `src/textReplacer.ts` - DOM manipulation and text replacement logic
- `src/options.tsx` - SolidJS-based options page UI
- `src/options-entry.tsx` - Entry point for options page
- `src/manifest.config.ts` - Chrome extension manifest configuration

### Configuration

- `vite.config.ts` - Vite bundler configuration
- `eslint.config.ts` - ESLint configuration with SolidJS rules
- `tailwind.config.js` - Tailwind CSS configuration
- `tsconfig.json` - TypeScript configuration

## Code Style Guidelines

### Comments

- Avoid trivial comments that state the obvious
- Use TSDoc format (`/** */`) for function documentation
- Keep performance-critical comments that explain non-obvious optimizations
- Remove UI section comments like `{/* Header */}` in React/SolidJS components

### TypeScript

- Use strict type checking
- Prefer explicit types over `any` when possible
- Use type assertions carefully (e.g., `(element as any).shadowRoot` for DOM
  APIs)

### Patterns

- Use top-level await instead of promise chains where appropriate
- Prefer early returns to reduce nesting
- Use WeakSet/WeakMap for DOM node tracking to prevent memory leaks

## Extension Architecture

### Content Script (`src/index.ts`)

- Loads user settings from chrome.storage
- Processes pages based on allowlist/blocklist
- Sets up MutationObserver for dynamic content
- Handles settings changes reactively

### Date Processing (`src/dateUtils.ts`, `src/textReplacer.ts`)

- Caches date calculations for performance
- Uses TreeWalker for efficient DOM traversal
- Skips processing inappropriate elements (scripts, styles, etc.)
- Implements configurable minimum age thresholds

### Options Page (`src/options.tsx`)

- SolidJS-based reactive UI
- Real-time validation with visual feedback
- Chrome storage integration
- Dark mode support via Tailwind

## Performance Considerations

- Date calculations are cached and updated every 5 minutes
- DOM processing is limited to 10,000 nodes maximum
- Uses WeakSet to track processed nodes and avoid reprocessing
- Debounced mutation handling (500ms) for dynamic content

## Debugging

The extension includes debug logging when `__DEBUG__` is true (development
builds):

- Performance timing for operations
- Settings loading/saving status
- URL processing decisions
- Text replacement details

Use browser developer tools Console tab to view debug output when testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theopak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
