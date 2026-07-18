## mypdfcv

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

<!-- END:nextjs-agent-rules -->

# Project Patterns

## Monorepo Structure

Npm workspaces with two internal packages:

- **@mypdfcv/i18n** (`packages/i18n/`) — locale config, message loader, JSON translations for 8 locales
- **@mypdfcv/pdf-core** (`packages/pdf/`) — PDF generation, Zod schemas, resume/cover letter types, templates

Build order: `npm run build:packages` (i18n then pdf via tsup), then `next build`.

## i18n

**Config:** `packages/i18n/src/config.ts` — locales array and `Locale` type.
**Loader:** `packages/i18n/src/loader.ts` — static imports of all JSON files, exports `getMessages(locale)`.
**Messages:** Two identical sets — `packages/i18n/src/messages/{locale}.json` (for pdf-core server usage) and `src/messages/{locale}.json` (for Next.js app via `src/i18n/request.ts` dynamic import). Both must be updated together.

Supported locales: `en`, `pt-BR`, `es`, `it`, `zh`, `ja`, `de`, `hi`.

**Adding a new locale:** Add to `config.ts` locales array → add import in `loader.ts` → create JSON in both message directories → add to `LanguageSwitcher.tsx` and `EditorToolbar.tsx` LANGUAGES arrays → add localized sample data in `src/lib/localizedSampleData.ts` → register font in `packages/pdf/src/lib/pdfFonts.ts` if non-Latin script.

## State Management

Zustand + `persist` (localStorage) + `immer` for immutable updates.

| Store         | File                               | localStorage Key                |
| ------------- | ---------------------------------- | ------------------------------- |
| Resumes       | `src/store/useResumeStore.ts`      | `architect-suite-resumes`       |
| Cover Letters | `src/store/useCoverLetterStore.ts` | `architect-suite-cover-letters` |

Both follow the same pattern: CRUD operations, `computeStatus()` for draft/complete, analytics tracking via `track()`.

## Forms

react-hook-form + Zod schemas (from `@mypdfcv/pdf-core`).

- `src/hooks/useResumeForm.ts` — watches all fields, debounces writes to store (300ms), resets on external changes
- `src/hooks/useCoverLetterForm.ts` — same pattern for cover letters

Validation mode: `"onTouched"`. Schemas defined in `packages/pdf/src/lib/schemas.ts` and `coverLetterSchemas.ts`.

## PDF Generation

Server-side rendering via `@react-pdf/renderer`.

| Route                             | Generator                  | Document Component           |
| --------------------------------- | -------------------------- | ---------------------------- |
| `POST /api/pdf/[id]`              | `generateResumePDF()`      | `ResumePDFDocument.tsx`      |
| `POST /api/cover-letter-pdf/[id]` | `generateCoverLetterPDF()` | `CoverLetterPDFDocument.tsx` |

**Fonts:** `packages/pdf/src/lib/pdfFonts.ts` — lazy-registers locale-specific fonts from CDN (jsDelivr). Latin locales use built-in Helvetica. Hindi uses Noto Sans Devanagari, CJK uses Noto Sans CJK. Always register italic variants (even as fallback to regular) to prevent crashes.

**Templates:** `packages/pdf/src/lib/resumeTemplates.ts` (7 templates) and `coverLetterTemplates.ts` (3 templates). Each has a style config (accentColor, headerLayout, sectionDivider, etc.) and optional sampleData.

**Localized sample data:** `src/lib/localizedSampleData.ts` provides per-locale sample resume content for template previews. Used by `TemplatePicker`, `DashboardHeader`, and `ResumeGrid` when creating resumes from templates.

## Components

Feature-based organization under `src/components/`:

| Directory              | Purpose                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------- |
| `landing/`             | Landing page sections (Nav, Hero, Features, Templates, FAQ, CTA)                      |
| `editor/`              | Resume editor (EditorNav, EditorToolbar, sections/, preview/)                         |
| `cover-letter-editor/` | Cover letter editor (Toolbar, sections/, preview/)                                    |
| `dashboard/`           | Dashboard (Header, ResumeGrid/Card, CoverLetterGrid/Card, Thumbnails, TemplatePicker) |
| `ui/`                  | Shared primitives (Button, Accordion, Tabs, Dialog, FormInput, etc.)                  |
| `providers/`           | React context providers (Theme, PostHog, Session)                                     |

UI components wrap `@base-ui/react` primitives with Tailwind styling.

## Styling

- **Tailwind CSS v4** with `@tailwindcss/postcss`
- **CVA** (class-variance-authority) for component variants (e.g., Button)
- **`cn()`** utility (`src/lib/utils.ts`) — clsx + tailwind-merge
- **CSS variables** in `src/app/globals.css` via `@theme` — light/dark mode colors, radius, typography
- **Fluid typography** via `clamp()` for headings
- **GSAP + ScrollTrigger** for landing page animations (always check `prefers-reduced-motion`)

## Analytics

PostHog via `track(event, properties?)` from `src/lib/analytics.ts`. Respects Do-Not-Track. Provider in `src/components/providers/PostHogProvider.tsx`.

Event naming: `snake_case` with entity prefix (e.g., `resume_created`, `cover_letter_pdf_exported`). Common properties: `templateId`, `locale`, `source`.

## Auth

next-auth v5 beta with LinkedIn OAuth provider. Config in `src/auth.ts`. JWT callback stores `accessToken`. Used for LinkedIn profile import only — all resume data is client-side localStorage.

## Conventions

- **File naming:** PascalCase for components, camelCase for utilities/hooks
- **Import alias:** `@/` maps to `src/`
- **Server packages:** `@react-pdf/renderer` and `@mypdfcv/pdf-core` are in `serverExternalPackages` (next.config.ts)
- **Commits:** Never add co-authored-by lines
- **Locale tags:** Use `toLocaleTag()` from `packages/pdf/src/lib/utils.ts` for BCP 47 date formatting
- **NoopStorage:** Zustand stores use a noop storage fallback for SSR safety

---
> Source: [wesleyramalho/mypdfcv](https://github.com/wesleyramalho/mypdfcv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
