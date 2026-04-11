## evaluador

> Monorepo with two independent projects:

# AGENTS.md - UML Evaluator

## Project Structure

Monorepo with two independent projects:

- `app/` — React + TypeScript frontend (Vite, shadcn/ui, Tailwind CSS)
- `uml-evaluator/backend/` — Python backend (FastAPI, uvicorn)

## Commands

### Frontend (`app/`)

```bash
cd app
npm install              # Install dependencies
npm run dev              # Start dev server (Vite)
npm run build            # Type-check + production build
npm run lint             # Run ESLint
npm run preview          # Preview production build
```

**No test framework is configured.** To add tests, use Vitest (`npm i -D vitest @testing-library/react @testing-library/jest-dom`) and add `"test": "vitest"` to scripts.

### Backend (`uml-evaluator/backend/`)

```bash
cd uml-evaluator/backend
python -m venv venv      # Create virtual env (if not exists)
source venv/bin/activate # Activate (Linux/Mac)
venv\Scripts\activate    # Activate (Windows)
pip install -r requirements.txt
python run.py            # Start dev server with hot reload
```

**No test framework is configured.** To add tests, use pytest (`pip install pytest httpx`) and run `pytest`.

## Code Style — Frontend (TypeScript/React)

### Imports
- Use `@/*` path alias for `src/*` (e.g., `@/components/ui/button`)
- Group imports: React → third-party → internal (`@/`) → CSS
- Use named imports for React hooks: `import { useState, useCallback } from 'react'`
- Use `type` keyword for type-only imports: `import type { ComparisonResult } from '@/types/comparison'`

### Formatting
- 2-space indentation
- Semicolons at end of statements
- Single quotes for strings (JSX props use double quotes)
- Trailing commas in multi-line objects/arrays
- Max line length: follow ESLint defaults

### TypeScript
- `strict: true` — no `any`, no implicit `any`
- `noUnusedLocals` and `noUnusedParameters` enabled — remove unused code
- `verbatimModuleSyntax` enabled — use proper `import type`
- Prefer interfaces for object shapes, type unions for discriminated types
- Export types from dedicated `types/` files

### Naming Conventions
- **Components:** PascalCase (`FileUploadZone`, `SimilarityGauge`)
- **Hooks:** camelCase with `use` prefix (`useIsMobile`)
- **Types/Interfaces:** PascalCase (`ComparisonResult`, `DiagramClass`)
- **Variables/functions:** camelCase (`handleCompare`, `weightsTotal`)
- **Constants:** camelCase for lookup objects (`visSymbol`, `relLabel`)
- **CSS classes:** kebab-case via Tailwind utility classes

### React Patterns
- Functional components only — no class components
- Use `useCallback` for event handlers passed as props
- Use `useState` for local state, lift state up when shared
- Props defined as interfaces or inline type annotations
- JSX: self-closing tags for elements without children
- Component files co-located in `components/` or inline in `App.tsx` for simple apps

### Error Handling
- Use try/catch for async operations (fetch calls)
- Display errors via `Alert` component with `AlertCircle` icon
- Validate inputs before submission (e.g., weights must sum to 100%)
- Use `instanceof Error` check when catching unknown errors

### Styling
- Tailwind CSS utility classes exclusively — no custom CSS except `App.css`/`index.css` globals
- Use `cn()` from `@/lib/utils` for conditional class merging
- shadcn/ui components from `@/components/ui/` (Radix UI primitives)
- Responsive prefixes: `md:`, `lg:` for breakpoints
- Dark mode via `next-themes` with CSS variables

## Code Style — Backend (Python/FastAPI)

### Imports
- Standard library → third-party → local modules
- Use absolute imports from `app.` package

### Naming Conventions
- **Functions/variables:** snake_case (`handle_upload`, `expected_file`)
- **Classes:** PascalCase (`UMLParser`, `ComparisonEngine`)
- **Constants:** UPPER_SNAKE_CASE
- **Files:** snake_case (`main.py`, `xmi_parser.py`)

### FastAPI Patterns
- Use Pydantic models for request/response validation
- Use `python-multipart` for file uploads
- Define routes in `app/api/main.py`
- Use type hints on all function signatures

### Error Handling
- Raise `HTTPException` with appropriate status codes
- Use Pydantic validation errors for bad input
- Return structured JSON error responses with `detail` field

## General Rules
- No emojis in code (comments may use them sparingly)
- UI text and comments are in Spanish — maintain consistency
- Do not commit `node_modules/`, `venv/`, `dist/`, `__pycache__/`, `.env`
- Follow existing file organization — add new files to appropriate subdirectories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davlillo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davlillo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
