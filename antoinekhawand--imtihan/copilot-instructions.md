## imtihan

> > **Read this file first** at the start of every session. It is the source of truth for Imtihan's architecture, conventions, and open decisions. Update it when anything fundamental changes.

# CLAUDE.md

> **Read this file first** at the start of every session. It is the source of truth for Imtihan's architecture, conventions, and open decisions. Update it when anything fundamental changes.

---

## 1. What Imtihan Is

**Imtihan** (امتحان — "exam" in Arabic) is an AI-powered exam generator for teachers in Lebanon, covering school curricula (Bac Libanais, Bac Français, IB) and university courses. Teachers describe what they need, optionally upload course materials, and receive a polished exam + corrigé in Word and PDF formats.

**Owner:** Founder — solo founder, based in Bsabba, Lebanon.
**Status:** MVP in development. Not yet launched.
**Target launch:** Q3 2026.

## 2. The Core Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1 — DESCRIBE                                              │
│  Teacher writes a natural-language description + optionally     │
│  uploads PDFs/DOCX/images (textbook chapter, past exam, notes). │
│                                                                 │
│  → POST /api/analyze                                            │
│  → Gemini 2.5 Flash with vision reads everything                │
│  → Returns structured ExamContext (curriculum, level, subject,   │
│    chapters, language, duration, exercise count, etc.)          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2 — CONFIRM CONTEXT                                       │
│  Auto-filled form. Teacher reviews/edits each field.            │
│  Validates against curriculum JSON (chapter must exist in level)│
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3 — STRUCTURE & STYLE                                     │
│  Points distribution, difficulty mix slider, template choice,   │
│  Version A/B toggle.                                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4 — GENERATE & REFINE                                     │
│  POST /api/generate — streams exercises from Claude.            │
│  Per-exercise actions: regenerate, edit, easier, harder, remove.│
│  Corrigé generated alongside, toggleable view.                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 5 — EXPORT                                                │
│  Download Word (docx lib), download PDF (@react-pdf/renderer),  │
│  or email. Auto-saved to Firestore under users/{uid}/exams.     │
└─────────────────────────────────────────────────────────────────┘
```

## 3. Tech Stack (Locked In)

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 15 (App Router) + React 19 | SEO, server components, API routes in same repo |
| Language | TypeScript (strict) | End-to-end type safety |
| Styling | Tailwind CSS v3.4 | Fast iteration, design tokens in `tailwind.config.ts` |
| Components | Custom — no shadcn | Editorial aesthetic, no generic look |
| Fonts | Fraunces (display serif) + Geist (sans) | Distinctive, not the usual Inter |
| Animation | Framer Motion | For stage transitions and reveals |
| Auth | Firebase Auth | Standard cloud stack |
| Database | Firestore | Teacher accounts, saved exams, exam library |
| Storage | Firebase Storage | Uploaded source documents, generated files |
| AI | Google Gemini 1.5 Flash | Best quality/cost for exam generation + vision |
| Word export | `docx` npm package | Full styling control |
| PDF export | `@react-pdf/renderer` | React-native PDF rendering |
| Payments | Stripe (v1.1+) | Monthly subscription, USD |
| Hosting | Vercel | Next.js native |

**Do not swap these without updating this file and getting approval.**

## 4. Directory Conventions

```
src/
├── app/                  # Next.js App Router — routes + API endpoints
│   ├── (marketing)/      # Public pages: landing, pricing, about
│   ├── (app)/            # Authenticated app: dashboard, create, library
│   └── api/              # Route handlers
├── components/
│   ├── ui/               # Primitives: Button, Input, Dropzone
│   ├── landing/          # Marketing-only sections
│   ├── workflow/         # The 5-step exam creation flow
│   └── layout/           # Header, Footer, Shell
├── lib/
│   ├── anthropic.ts      # Claude API client
│   ├── firebase.ts       # Client SDK init
│   ├── firebase-admin.ts # Server SDK init
│   ├── utils.ts          # cn(), formatters
│   └── prompts/          # System prompts — versioned, testable
├── types/                # Shared TS types: Exam, ExamContext, etc.
└── data/
    └── curricula/        # Curriculum JSON — single source of truth
```

**Rule:** Any curriculum chapter the AI references MUST exist in `src/data/curricula/`. If it doesn't, we're hallucinating exam content, which is unacceptable.

## 5. Prompting Philosophy

Prompts live in `src/lib/prompts/` as versioned TypeScript files, not hardcoded strings inside route handlers. Each prompt:

1. Has a **named export** describing its purpose: `buildAnalyzePrompt()`, `buildExerciseGenerationPrompt()`.
2. Takes typed inputs and returns a single string.
3. Uses explicit XML tags when structuring sub-sections (`<curriculum>`, `<teacher_description>`, etc.) — this is how Claude is trained to parse structure.
4. Asks Gemini to **return JSON only** when structured output is needed, and we parse it safely with Zod schemas.

**Never** inline a prompt as a template literal inside a route handler. It makes prompts un-testable and un-versionable.

## 6. Data Models

Core types live in `src/types/`:

- `ExamContext` — the structured description of what to generate (curriculum, level, subject, chapters, language, etc.)
- `Exam` — a generated exam with exercises, corrigé, metadata
- `Exercise` — a single question block with statement, difficulty, points, answer, methodology
- `Curriculum` — the static data defining chapters per level per subject
- `UserProfile` — Firebase Auth user + Imtihan-specific fields (exam quota, school, role)

Treat these as contracts. Changing them ripples through the whole app — do it deliberately.

## 7. Firestore Schema

```
users/{uid}
  email, displayName, role, school, createdAt
  examsGenerated: number  // for free-tier quota enforcement
  subscription: { status, tier, stripeCustomerId, renewsAt }

users/{uid}/exams/{examId}
  title, context (ExamContext), exercises[], corrige, createdAt
  versions: { A?: Exam, B?: Exam }  // for Version A/B feature
  tags: string[]
```

**Security rules:** A user can only read/write their own subcollection. Server-side (Admin SDK) is the only path to cross-user operations.

## 8. Open Decisions (Ask Before Choosing)

- [x] **Domain name** — imtihan.live (purchased)
- [ ] **Accent color final** — currently emerald `#1a5e3f`. Could also be terracotta.
- [x] **Free tier** — confirmed 1 exam lifetime (not per month).
- [ ] **Pricing after free tier** — probably $7–10/month for individuals. School pricing TBD.
- [ ] **Arabic support** — deferred to v1.1. Do not start until MVP is stable.
- [ ] **Custom .docx format upload** — deferred to v1.1. Templates only in MVP.
- [ ] **Biology / SVT / Informatique subjects** — deferred to v1.1.
- [ ] **Authentication provider** — confirmed Firebase Auth. Google + email/password in MVP.

## 9. MVP Scope (Do Not Exceed)

**In MVP:**
- Curricula: Bac Libanais, Bac Français, IB, University (free-form)
- Subjects: Math, Physics, Chemistry
- Languages: French, English
- 4–5 default templates (no custom upload)
- Teacher account + exam library
- Version A/B generation
- Granular exercise regeneration
- Methodology in corrigé
- Word + PDF export
- 1 free exam, then paywall stub (paywall logic can be a TODO until Stripe is wired)

**Note on University Curriculum:** While "free-form" for the user, AI responses should be grounded in high-quality past exams (`dawrat`) and syllabi sourced according to the `docs/DATA_SOURCING.md` guide. This ensures authenticity.

**Not in MVP — defer firmly:**
- Arabic, Biology, custom format upload, school accounts, question bank, homework generator, grading assistant

If a request implies scope creep, flag it and point back to this section.

## 10. Code Conventions

- **No `any`.** Use `unknown` + narrowing, or define a type.
- **Server components by default.** Add `"use client"` only when needed (state, events, browser APIs).
- **Zod at every boundary.** Parse anything coming from the AI, from the user, from the network.
- **`cn()` from `lib/utils`** for conditional classes. Never string-concat Tailwind.
- **No default exports** except for Next.js page/layout/route files (which require them).
- **Error boundaries** around each workflow step so a Claude API failure doesn't crash the whole flow.
- **SEO First:** All public marketing pages must be server components with a full `metadata` object export including `alternates.canonical` and `openGraph`.
- **GEO (Generative Engine Optimization):** Use explicit structured data (`<SchemaOrg>`) for FAQ/Product data and update `public/llms.txt` when adding new features or curricula.

## 11. Testing Strategy (Post-MVP Priority)

- Unit tests for prompt builders (deterministic string output).
- Integration tests for the generate endpoint using recorded Gemini responses.
- E2E with Playwright for the full workflow — same pattern as previous apps.

## 12. Known Gotchas

- **Firebase Admin private key** in env vars needs literal `\n` characters, not real newlines. Handle with `.replace(/\\n/g, '\n')` on load.
- **Gemini vision for PDFs** works best when PDFs are small (<20 pages). Large syllabi should be chunked.
- **Next.js 15 + React 19** — some older libraries don't support React 19 yet. Check before adding deps.
- **Lebanese internet** — Vercel edge performs okay in Beirut, but generated file downloads must be self-contained (no re-fetching assets from cloud at print time).
- **Gemini `thinking` tokens** — The SDK can fail on "thinking" tokens in streams. `thinkingBudget: 0` is set in `gemini.ts` to suppress them.

## 13. When You're Stuck

1. Check this file first.
2. Check `ARCHITECTURE.md` for deeper technical rationale.
3. Check `ROADMAP.md` for what's planned vs. done.
4. Check `BUGS.md` for known issues before reporting a new one.
5. Ask for approval. Don't silently rewrite core decisions.

## 14. Definition of Done for Any Feature

- [ ] Types are in `src/types/` and imported everywhere they're used
- [ ] Prompts are in `src/lib/prompts/` with named exports
- [ ] Zod schema validates any AI or user input
- [ ] UI follows the editorial aesthetic (Fraunces + Geist, generous whitespace, emerald accent)
- [ ] Works in both light and dark mode
- [ ] Mobile-responsive (breakpoint test at 375px, 768px, 1440px)
- [ ] No console errors, no TypeScript errors (`npm run type-check`)
- [ ] CLAUDE.md, ROADMAP.md, or BUGS.md updated if scope or state changed

---
> Source: [AntoineKhawand/imtihan](https://github.com/AntoineKhawand/imtihan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
