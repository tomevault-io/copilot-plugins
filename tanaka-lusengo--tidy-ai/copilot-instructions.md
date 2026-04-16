## tidy-ai

> Tidy AI is an AI-powered spreadsheet cleaning and merging tool. Users upload CSV/TSV/XLSX files, the app analyses data quality issues using Claude, lets users refine a unified schema, then produces cleaned/merged output with export options (CSV, PDF report, email draft).

# CLAUDE.md — Tidy AI

## Project Overview

Tidy AI is an AI-powered spreadsheet cleaning and merging tool. Users upload CSV/TSV/XLSX files, the app analyses data quality issues using Claude, lets users refine a unified schema, then produces cleaned/merged output with export options (CSV, PDF report, email draft).

## Tech Stack

- **Framework:** Next.js 16 (App Router) with React 19 and TypeScript 5
- **Package Manager:** pnpm 10 (pinned in `package.json`)
- **Styling:** Tailwind CSS 4 (CSS-first config via `@tailwindcss/postcss`), Motion.js for animations
- **UI:** shadcn/ui (New York style), Radix UI primitives, Lucide icons
- **AI:** Vercel AI SDK (`ai` + `@ai-sdk/gateway`) routing to Anthropic Claude models
- **Validation:** Zod 4
- **File Parsing:** PapaParse (CSV/TSV), XLSX (Excel)
- **PDF Generation:** jsPDF + jspdf-autotable

## Commands

```bash
pnpm dev          # Start dev server (localhost:3000)
pnpm build        # Production build
pnpm start        # Start production server
pnpm lint         # Run ESLint
```

## Architecture

### Directory Structure

```plaintext
app/
  layout.tsx          — Root layout, metadata, fonts
  page.tsx            — Main wizard UI (composes step components + useWizard)
  error.tsx           — Error boundary
  globals.css         — Global styles, CSS custom properties, glassmorphism theme
  api/
    analyze/route.ts  — POST: AI analysis via NDJSON stream (Sonnet 4.5, partial object updates)
    clean/route.ts    — POST: AI cleaning via NDJSON stream (Haiku 4.5 for rows, Sonnet 4.5 for summary)
components/
  FileUpload.tsx        — Drag-drop file uploader
  DataPreview.tsx       — Reusable table preview
  AnalysisResults.tsx   — Display analysis findings, issues, questions
  SchemaEditor.tsx      — Editable unified schema table
  CleanedOutput.tsx     — Cleaned data preview + export actions
  Stepper.tsx           — Step progress indicator
  LoadingSkeleton.tsx   — Shimmer loading skeleton
  TransformEditDialog.tsx — Dialog for editing column transforms
  steps/
    UploadStep.tsx      — Step 0: file upload + notes
    AnalysisStep.tsx    — Step 1: AI analysis results
    SchemaStep.tsx      — Step 2: schema editor wrapper
    OutputStep.tsx      — Step 3: cleaned output + exports
    ErrorBanner.tsx     — Inline error display
  ai-elements/
    shimmer.tsx         — Animated shimmer text effect
  ui/
    animated-icon.tsx   — Factory for animated Lucide icon wrappers
    glass-card.tsx      — Glassmorphism card component
    section-heading.tsx — Section heading component
    (+ shadcn primitives: button, badge, dialog, input, table, textarea)
    (+ animated icons: check, chevron-left, chevron-right, download, file-text, rotate-ccw, sparkles, upload, x)
hooks/
  wizard/
    useWizard.ts       — Main wizard hook (composes reducer + actions)
    reducer.ts         — useReducer state machine with step guards
    actions.ts         — Async actions (analyseFiles, cleanFiles) with AbortController
    index.ts           — Barrel export
lib/
  ai.ts              — Model config (Sonnet/Haiku), BYOK gateway, concurrency helper, error response helper, prompt sanitisation
  types.ts           — TypeScript interfaces, Zod schemas, request validation schemas
  parseFile.ts       — CSV/XLSX/TSV file parsing
  downloadUtils.ts   — CSV/PDF export, email draft generation
  schemaUtils.ts     — Column mapping lookup utilities
  utils.ts           — cn() utility (clsx + tailwind-merge)
```

### State Management

The app uses a `useReducer`-based wizard state machine extracted into `hooks/wizard/`. The reducer enforces step transition guards (e.g. cannot advance to Analysis without files), handles error recovery (routes back to the appropriate step), and uses exhaustive action matching.

The wizard has 4 steps (constants in `WIZARD_STEPS`):

- **UPLOAD (0):** File upload, preview, optional notes
- **ANALYSIS (1):** AI analyses data quality, generates column mappings
- **SCHEMA (2):** User refines unified schema, answers clarifying questions
- **OUTPUT (3):** Preview cleaned data, download CSV/PDF, copy email draft

Async actions (`analyseFiles`, `cleanFiles`) use `AbortController` to prevent race conditions when the user triggers multiple requests.

### AI Models

Configured in `lib/ai.ts` via Vercel AI Gateway:

- **Sonnet 4.5** (`analysisModel`): Reasoning-heavy tasks — analysis, schema design, summary generation
- **Haiku 4.5** (`cleaningModel`): Fast mechanical tasks — row-level cleaning & type coercion
- **BYOK support:** Set `ENABLE_BYOK=true` and `ANTHROPIC_API_KEY` to route through user's own key

### Data Flow

1. Upload files → `parseFile()` extracts headers/rows
2. `/api/analyze` receives sample (first 50 rows) → streams partial analysis via NDJSON → returns column mappings, issues, questions
3. User edits schema + answers questions
4. `/api/clean` receives full rows (max 10,000) → batched processing (75 rows/batch, max 3 concurrent) → streams progress via NDJSON
5. Summary call (Sonnet 4.5) aggregates changes → export as CSV/PDF/email

Both API routes use NDJSON streaming (`application/x-ndjson`). The analyze endpoint streams partial structured objects as they build up, while the clean endpoint streams `__progress` events with batch completion status. Final results are sent as `{ __final: true, data: ... }` and errors as `{ __error: true, message: ... }`.

## Environment Variables

| Variable            | Required      | Description                                          |
|---------------------|---------------|------------------------------------------------------|
| `AI_GATEWAY_API_KEY`| Yes           | Vercel AI Gateway key (authenticates with the gateway)|
| `ENABLE_BYOK`      | No            | Set to `"true"` to enable Bring Your Own Key mode    |
| `ANTHROPIC_API_KEY` | Only if BYOK  | Your Anthropic API key for direct billing            |

## Conventions

- Path alias: `@/*` maps to project root (configured in `tsconfig.json`)
- All components are client-side (`"use client"`)
- CSS custom properties for theming (defined in `globals.css`)
- Anthropic brand accent: `#D97757`
- Console logs prefixed with `[TidyAI]`
- File size limit: 10 MB per file
- Total row limit: 10,000 rows across all files (enforced by `cleanRequestZ`)
- Supported formats: CSV, TSV, XLSX, XLS
- **Language:** All user-facing text uses British English spelling (e.g. "analyse" not "analyze", "normalise" not "normalize", "summarise" not "summarize"). Code identifiers and API route paths follow American English per industry convention (e.g. `isAnalyzing`, `/api/analyze`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tanaka-lusengo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
