## scan-serve

> These rules are **prescriptive**. Every interaction MUST follow them.

# ScanServe — Intelligent Receipt OCR Platform — Project Rules

These rules are **prescriptive**. Every interaction MUST follow them.
They combine real codebase patterns with approved architectural decisions.

---

## 0. AI Credits Efficiency

Every AI interaction costs credits. ALWAYS minimize usage:

- **Batch file reads**: Use parallel tool calls instead of sequential reads.
- **Batch questions**: Ask the user everything in ONE message. NEVER use multiple `ask_user_question` calls when a single open message covers the same ground.
- **Minimize back-and-forth**: Gather all needed context before responding. Don't ask clarification if you can infer intent.
- **Prefer code_search over multiple greps**: One targeted search beats five sequential greps.
- **Avoid redundant reads**: If file content was seen in this conversation, don't re-read it.
- **Be concise**: Shorter responses with the same information are always better.
- **Batch edits**: Use `multi_edit` for multiple changes to the same file instead of separate `edit` calls.
- **All workflows and rules MUST be written in English** — easier for the AI system.

---

## 1. Project Overview

ScanServe transforms receipt photos into structured, searchable data using dual OCR engines and a multi-agent AI pipeline with real-time NDJSON streaming. It is a **full-stack monorepo** with a React frontend and a FastAPI backend.

---

## 2. Project Structure

```
scan-serve/
├── frontend/                   # React SPA
│   ├── src/
│   │   ├── components/
│   │   │   ├── navigation/     # AppHeader, ScanResultTabs
│   │   │   ├── results/        # AiStageBar, ConfidenceMeter, OCRFields, OCRText
│   │   │   ├── uploader/       # ImageUploader, MultipleImageUploader, BatchCameraCapture
│   │   │   ├── viewer/         # BeforeAfterViewer, BoundingBoxOverlay
│   │   │   └── ui/             # shadcn/ui primitives (50+ components) — DO NOT modify
│   │   ├── features/
│   │   │   └── receipts/       # Self-contained feature module
│   │   │       ├── components/ # FileTreeExplorer, ReceiptPreviewPanel, ReceiptExplorerTab
│   │   │       ├── db/         # IndexedDB database, repos (receipts, folders, settings)
│   │   │       ├── hooks/      # TanStack Query hooks (useReceipts, useReceiptFolders)
│   │   │       ├── types/      # Receipt & Folder domain types
│   │   │       └── utils/      # Folder tree builder, ID generator, object URL manager
│   │   ├── pages/              # Home, Results, Receipts, Settings, Pricing, NotFound
│   │   ├── services/
│   │   │   └── api.ts          # Axios REST client + native fetch NDJSON stream consumer
│   │   ├── store/              # Zustand stores (OCR state, scan tabs, upload settings)
│   │   ├── types/              # Shared TypeScript interfaces
│   │   ├── hooks/              # Custom React hooks
│   │   ├── lib/                # Utilities (cn function)
│   │   └── utils/
│   │       ├── ocr/            # Custom layout engine (geometry → rows → columns → markdown)
│   │       └── receipt/        # Markdown parser + HTML renderer
│   └── public/
├── backend/                    # FastAPI + Python
│   ├── app/
│   │   ├── api/v1/endpoints/   # Route handlers (OCR, receipts, folders, notifications)
│   │   ├── core/               # Config, dependencies, middleware, logging
│   │   ├── models/             # Pydantic v2 request/response models
│   │   ├── repositories/       # JSON file-based persistence
│   │   └── services/
│   │       ├── ai_receipt_agents/  # Multi-agent LLM pipeline (Organizer → Auditor → Stylist)
│   │       ├── ai_trace_service.py # Run & revision persistence with Markdown snapshots
│   │       ├── ocr_service.py      # EasyOCR wrapper + field guessing heuristics
│   │       ├── google_vision_ocr_service.py  # Google Vision API integration
│   │       ├── ocr_queue.py        # Thread-based concurrent OCR job queue
│   │       ├── storage_service.py  # Upload handling & static file management
│   │       └── ticket_html_renderer.py  # Pixel-accurate HTML receipt generation
│   ├── data/                   # JSON file-based database
│   ├── uploads/                # Uploaded receipt images
│   └── logs/                   # Application logs
├── docs/                       # Planning docs, pricing specs
└── .windsurf/                  # IDE rules and workflows
```

- This is a **full-stack monorepo** — `frontend/` and `backend/` at the root.
- Frontend dev server runs on port **8080** (`vite.config.ts`).
- Backend dev server runs on port **8000** (Uvicorn).

---

## 3. Tech Stack

### Frontend

| Layer           | Technology                              | Notes                                           |
| --------------- | --------------------------------------- | ----------------------------------------------- |
| Framework       | React 18 + TypeScript 5.8              | SWC compiler via `@vitejs/plugin-react-swc`     |
| Build           | Vite 5                                 | Dev port 8080, `@/` alias → `./src/*`           |
| Styling         | Tailwind CSS 3 + shadcn/ui + Radix UI  | Custom design tokens via CSS variables          |
| State           | Zustand (global) + TanStack Query v5 (server/async) | Stores in `store/`, queries in feature hooks |
| Routing         | React Router v6                        | `BrowserRouter` with Routes in `App.tsx`        |
| Forms           | React Hook Form 7 + Zod 3             | For validation                                   |
| Animations      | Framer Motion                          | Entrance animations, expandable cards           |
| Icons           | Lucide React                           | Primary icon library                            |
| Toasts          | Sonner + Radix Toast                   | Both registered in `App.tsx`                    |
| HTTP            | Axios (REST) + native `fetch` (NDJSON streaming) | `services/api.ts`                      |
| Charts          | Recharts                               | Used in analytics / stats                       |
| Offline Storage | IndexedDB (raw IDB API)                | `features/receipts/db/`                         |
| Typography      | Plus Jakarta Sans                      | Via `@fontsource/plus-jakarta-sans`             |

### Backend

| Layer           | Technology                              | Notes                                           |
| --------------- | --------------------------------------- | ----------------------------------------------- |
| Framework       | FastAPI + Uvicorn                      | Async-ready, port 8000                          |
| Validation      | Pydantic v2 + pydantic-settings        | Models in `app/models/`, config with `RV_` prefix |
| OCR             | Google Cloud Vision API + EasyOCR      | Dual engine, selectable per scan                |
| AI/LLM          | OpenAI API (gpt-5-mini / gpt-5-nano)  | Multi-agent pipeline in `services/ai_receipt_agents/` |
| Storage         | File-based JSON database + static files | `data/db.json`, `uploads/`                     |
| Image           | Pillow                                 | Image processing                                |
| Concurrency     | Thread-based OCR queue                 | Configurable max workers                        |

---

## 4. Naming Conventions

### Frontend

| Item               | Convention                                | Examples                                  |
| ------------------ | ----------------------------------------- | ----------------------------------------- |
| Page components    | `PascalCase.tsx`, `export default`        | `Home.tsx`, `Results.tsx`, `NotFound.tsx`  |
| Feature components | `PascalCase.tsx` in feature dirs          | `FileTreeExplorer.tsx`, `ReceiptPreviewPanel.tsx` |
| Shared components  | `PascalCase.tsx` in `components/`         | `NavLink.tsx`                             |
| shadcn/ui          | `kebab-case.tsx` in `ui/`                 | `button.tsx`, `card.tsx`                  |
| Custom hooks       | `use-kebab-case.tsx`                      | `use-mobile.tsx`, `use-toast.ts`          |
| Zustand stores     | `camelCaseStore.ts`                       | `ocrStore.ts`, `scanResultsStore.ts`      |
| Utilities          | `camelCase.ts`                            | `utils.ts`, `image.ts`                    |

### Backend

| Item               | Convention                                | Examples                                  |
| ------------------ | ----------------------------------------- | ----------------------------------------- |
| Modules            | `snake_case.py`                           | `ocr_service.py`, `ai_trace_service.py`  |
| Pydantic models    | `PascalCase` classes                      | `OcrResult`, `ReceiptCreate`              |
| Route handlers     | `snake_case` functions                    | `create_receipt`, `process_ocr`           |
| Settings           | `snake_case` fields with `RV_` env prefix | `openai_receipt_model`, `cors_origins`    |

---

## 5. Import Rules

### Frontend

- ALWAYS use **`@/` alias** for imports (maps to `./src/*`):
  ```typescript
  import { Button } from "@/components/ui/button";
  import { cn } from "@/lib/utils";
  ```
- Imports ALWAYS at the top of the file — NEVER import mid-file.
- **Relative imports within the same feature/directory**, `@/` for cross-directory:
  ```typescript
  // Inside features/receipts/hooks/useReceipts.ts
  import { receiptRepo } from '../db/receiptRepo';     // relative (same feature)
  import { useToast } from '@/hooks/use-toast';         // alias (cross-directory)
  ```

### Backend

- Use **absolute imports** from the `app` package:
  ```python
  from app.core.config import settings
  from app.services.ocr_service import OcrService
  from app.models.ocr import OcrResult
  ```

---

## 6. Architecture Patterns

### 6.1 Feature-Sliced Architecture (Frontend)

The `features/receipts/` module is **self-contained** with its own:
- `components/` — UI components specific to the feature
- `db/` — IndexedDB repositories (data access)
- `hooks/` — TanStack Query hooks (data fetching)
- `types/` — Domain types
- `utils/` — Feature-specific utilities

New features SHOULD follow this pattern: `features/<name>/`.

### 6.2 Service Layer (Backend)

- **Endpoints** (`api/v1/endpoints/`) handle HTTP concerns only (request/response).
- **Services** (`services/`) contain business logic — one service per domain.
- **Models** (`models/`) define Pydantic v2 request/response schemas.
- **Repositories** (`repositories/`) handle data persistence (JSON file-based).

### 6.3 State Management (Frontend)

| Type            | Tool                 | Location                       |
| --------------- | -------------------- | ------------------------------ |
| Global UI state | Zustand              | `store/ocrStore.ts`, `store/scanResultsStore.ts`, `store/uploadSettingsStore.ts` |
| Server state    | TanStack Query v5    | `features/receipts/hooks/`     |
| Offline data    | IndexedDB            | `features/receipts/db/`        |

### 6.4 Multi-Agent AI Pipeline (Backend)

3-stage pipeline: **Organizer → Auditor → Stylist**
- Located in `services/ai_receipt_agents/`
- Supports both sync (`/ocr/ai/parse`) and streaming (`/ocr/ai/parse/stream`) modes
- Streaming uses NDJSON protocol with typed events (`pipeline_start`, `stage_start`, `stage_result`, `handoff`, `result`, `pipeline_done`)
- Every run produces immutable revision snapshots via `ai_trace_service.py`

---

## 7. Styling Rules

- **Tailwind CSS utility classes exclusively** — NO inline styles, NO CSS modules.
- Use semantic tokens: `text-primary`, `text-muted-foreground`, `bg-card`.
- **`cn()` from `@/lib/utils`** for conditional class merging:
  ```typescript
  import { cn } from "@/lib/utils";
  <div className={cn("base-class", condition && "conditional-class")} />
  ```
- **Responsive**: mobile-first with `sm:`, `md:`, `lg:` breakpoints.
- **Dark/Light theme**: Class-based switching via `next-themes`.
- **shadcn/ui** components in `components/ui/` — **DO NOT modify** these files.

---

## 8. Routes

```typescript
{ path: "/",          element: <Home /> },
{ path: "/results",   element: <Results /> },
{ path: "/receipts",  element: <Receipts /> },
{ path: "/pricing",   element: <Pricing /> },
{ path: "/settings",  element: <Settings /> },
{ path: "*",          element: <NotFound /> },
```

- Add new routes ABOVE the `*` catch-all in `App.tsx`.

---

## 9. API Endpoints

| Endpoint                              | Method         | Description                                      |
| ------------------------------------- | -------------- | ------------------------------------------------ |
| `POST /ocr`                         | multipart      | Single image OCR (EasyOCR / Vision / both)       |
| `POST /ocr/ticket`                  | multipart      | HTML ticket render from OCR                      |
| `POST /ocr/ai/parse`                | JSON           | AI receipt parsing (synchronous)                 |
| `POST /ocr/ai/parse/stream`         | JSON → NDJSON | AI receipt parsing (streaming, real-time stages) |
| `POST /api/v1/receipts`             | multipart      | Create receipt + trigger queued OCR              |
| `GET /api/v1/receipts/:id`          | —             | Poll receipt status / get OCR result             |
| `CRUD /api/v1/folders`              | JSON           | Folder management                                |
| `POST /api/v1/notifications/notify` | JSON           | Email notification triggers                      |
| `GET /health`                        | —             | Health check                                     |

---

## 10. Environment Variables

### Frontend (`frontend/.env.local`)

| Variable         | Default                   | Description          |
| ---------------- | ------------------------- | -------------------- |
| `VITE_API_URL` | `http://localhost:8000` | Backend API base URL |

### Backend (`backend/.env`)

| Variable                              | Default                       | Description                        |
| ------------------------------------- | ----------------------------- | ---------------------------------- |
| `RV_CORS_ORIGINS`                   | `["http://localhost:8080"]` | Allowed CORS origins               |
| `GOOGLE_VISION_OCR_API_KEY`         | —                            | Google Cloud Vision API key        |
| `GPT_5_MINI_API`                    | —                            | OpenAI API key                     |
| `RV_OPENAI_RECEIPT_MODEL`           | `gpt-5-mini`                | Default LLM model for all agents   |
| `RV_OPENAI_RECEIPT_ORGANIZER_MODEL` | (inherits)                    | Override model for organizer agent |
| `RV_OPENAI_RECEIPT_AUDITOR_MODEL`   | (inherits)                    | Override model for auditor agent   |
| `RV_OPENAI_RECEIPT_STYLIST_MODEL`   | (inherits)                    | Override model for stylist agent   |
| `RV_OPENAI_RECEIPT_JSON_RETRIES`    | `1`                         | Retries on invalid JSON from LLM   |
| `OCR_QUEUE_MAX_CONCURRENT`          | `1`                         | Max parallel OCR workers           |

- NEVER hardcode API keys or secrets.
- Frontend env vars with `VITE_` prefix are compile-time — must be set BEFORE `npm run build`.
- Backend uses `RV_` prefix via pydantic-settings (`SettingsConfigDict`).

---

## 11. Code Quality Standards

### TypeScript (Frontend)

- **`interface` for component props**, **`type` for data shapes and unions**.
- Prefer typed generics, discriminated unions, or `unknown` with type guards over `any`.
- `as const` for literal objects.
- Imports ALWAYS at the top — NEVER mid-file.

### Python (Backend)

- Use **Pydantic v2** models for all request/response schemas.
- Use **type hints** on all function signatures.
- Follow `pydantic-settings` pattern for configuration (`app/core/config.py`).
- Structured logging via `app/core/logger.py`.

### Shared Rules

- **No TODO without context** — every TODO must include what and why.
- **Single responsibility** — one component/service per concern.
- **Explicit error handling** — NEVER swallow exceptions silently.

---

## 12. Running the Project

```powershell
# Terminal 1 — Backend
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# Terminal 2 — Frontend
cd frontend
npm install
npm run dev
```

- Frontend: `http://localhost:8080`
- Backend: `http://localhost:8000`
- Backend health check: `GET http://localhost:8000/health`

---

## 13. Planned Features

| Feature                                    | Status         |
| ------------------------------------------ | -------------- |
| Supabase integration for cloud persistence | 🔲 Not started |
| Export receipts to CSV / Excel / PDF       | 🔲 Not started |
| Receipt analytics dashboard                | 🔲 Not started |
| Drag-and-drop receipt reordering           | 🔲 Not started |
| End-to-end tests with Playwright           | 🔲 Not started |
| PWA support for mobile installation        | 🔲 Not started |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabriel-klettur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
