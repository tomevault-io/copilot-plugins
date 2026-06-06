## lt-prompter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LT-Prompter is an Angular 18 PWA for language teachers to generate AI prompts for creating language learning exercises. It generates structured prompts for various exercise types across multiple target languages (English, Spanish, French, Italian) at different CEFR levels (A1-C2).

## Commands

```bash
npm start          # Dev server at http://localhost:4200
npm run build      # Production build to dist/lt-prompter
npm test           # Unit tests via Karma
ng generate component components/<name>  # New component
```

## Architecture

### Core Patterns

- **Standalone Components**: All components use `standalone: true` (Angular 18 default)
- **Lazy Loading**: All routes use `loadComponent()` for code splitting
- **IndexedDB Storage**: User data persisted via `ngx-indexed-db` with stores: `preferences`, `templates`, `versions`, `library`
- **PWA**: Service worker enabled in production for offline support

### Key Services

- `PromptTemplateService` - Generates exercise prompts by combining language-specific templates with user configuration
- `PreferencesService` - Manages user settings (language, CEFR level, theme)
- `TemplatesService` - CRUD operations for custom prompt templates
- `LibraryService` - Manages saved prompts library

### Exercise Types

Located in `src/app/templates/`:
- **Vocabulary** (`vocabulary-prompts.ts`) - Word list contextualization exercises
- **Grammar** (`grammar-con-prompts.ts`) - Grammar phenomenon practice
- **Comprehension** (`comprehension-prompts.ts`) - Reading comprehension exercises
- **Clone** (`clone-prompts.ts`) - Exercise replication from source materials
- **Wordfield** (`wordfield-prompts.ts`) - Semantic field vocabulary exercises

### Models

All models in `src/app/models/`:
- `preferences.model.ts` - Language, CEFRLevel, ExerciseType types
- `template.model.ts` - Template structure with exercise type, language, CEFR level

## Important Conventions

- **UI Language**: Always German for all UI labels, messages, and instructions
- **Angular Material**: Use Material components for UI consistency
- **SCSS**: Default stylesheet language configured in `angular.json`
- **Reactive Forms**: Use for all form handling
- **Signals**: Prefer for local component state
- **Built-in Control Flow**: Use `@if`, `@for`, `@switch` syntax (not `*ngIf`, `*ngFor`)

## Deployment

Hosted on Netlify with automatic CI/CD from Git. Build output goes to `dist/lt-prompter`.

---
> Source: [BensAppForge/lt-prompter](https://github.com/BensAppForge/lt-prompter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
