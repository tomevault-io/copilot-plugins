## hilips

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HILIPS (Hierarchical Labeling with Iterative Pseudo-Labeling System) is a monorepo implementing a research-backed hierarchical labeling system with three phases:

1. **Phase 1 - Cold-start Labeling**: Gemini LLM detects objects → SAM2 creates segmentation masks → User reviews/edits → COCO format saved
2. **Phase 2 - Knowledge Distillation**: Train YOLOv8 on Phase 1 annotations, validate mAP@0.7 ≥ 0.7 for production readiness
3. **Phase 3 - Iterative Refinement**: YOLO batch inference → confidence ≥ 0.8 auto-labels → confidence < 0.8 goes to review queue → accumulate data → retrain

## Repository Structure

```
HILIPS/
├── keti-labeling/     # Next.js 15 frontend (React 19, TypeScript, Tailwind, shadcn/ui)
│   ├── app/           # App Router pages
│   ├── components/    # UI components
│   ├── hooks/         # Custom React hooks
│   └── lib/           # API clients, utilities
├── backend/           # FastAPI backend (Python, PyTorch, SAM2, YOLO)
│   ├── routers/       # API endpoints (coldstart, training, active_learning, models, workflow)
│   ├── services/      # Business logic (llm_sam_pipeline, yolo_service, active_learning, model_registry)
│   ├── models/        # ML model loaders (sam2_loader)
│   └── schemas/       # Pydantic models
└── SOFTWARE_FUNCTIONALITIES.md  # Detailed phase descriptions
```

## Build, Lint, Test Commands

### Frontend (`keti-labeling/`)

```bash
pnpm install              # Install dependencies
pnpm dev                  # Dev server (localhost:3000)
pnpm build                # Production build
pnpm lint                 # ESLint check
pnpm lint --fix           # Auto-fix lint issues
npx tsc --noEmit          # Type check

# Testing (Jest + React Testing Library)
pnpm exec jest                        # All tests
pnpm exec jest path/to/file.test.tsx  # Single file
pnpm exec jest -t "test name"         # Single test by name
pnpm exec jest --coverage             # With coverage
```

### Backend (`backend/`)

```bash
pip install -r requirements.txt                         # Install deps
uvicorn main:app --reload --host 0.0.0.0 --port 8000   # Dev server
ruff check . --fix                                      # Lint + fix
```

## Architecture Decisions

- **No database**: File-based JSON persistence (annotations, model registry, review queues)
- **Service layer**: Routers delegate to singleton services via `get_*_service()` pattern
- **API proxy**: Frontend calls rewritten through Next.js to backend (port 3000 → 8000)
- **shadcn/ui**: Reuse components from `components/ui/`
- **COCO format**: All annotations stored in COCO format, converted to YOLO format for training

## Key Implementation Files

| Phase | Backend Service | Backend Router |
|-------|-----------------|----------------|
| Phase 1 | `services/llm_sam_pipeline.py` | `routers/coldstart.py`, `routers/segmentation.py` |
| Phase 2 | `services/yolo_service.py`, `services/model_registry.py` | `routers/training.py` |
| Phase 3 | `services/active_learning.py` | `routers/active_learning.py`, `routers/models.py` |

## Code Conventions

### TypeScript/React

- Components: `PascalCase.tsx`, Hooks: `use-kebab-case.ts`, Utils: `kebab-case.ts`
- Always add `"use client"` for components using hooks
- Styling: Tailwind only, use `cn()` for conditional classes
- Types: Explicit types on exports, use Zod for runtime validation, avoid `any`

### Python

- Files: `snake_case.py`, Classes: `PascalCase`, Constants: `UPPER_SNAKE_CASE`
- Routers: raise `HTTPException(status_code, detail)`
- Services: `logger.error()` + raise `ValueError`

## ML Model Notes

- **SAM2 checkpoint**: `backend/checkpoints/sam2.1_hiera_large.pt` (download from Meta)
- **GPU memory**: SAM2 auto-unloads during YOLO training to free VRAM
- **mAP@0.7 threshold**: Models with mAP@0.7 ≥ 0.7 are production-ready
- **Confidence threshold**: Phase 3 uses 0.8 for auto-labeling, < 0.8 for review queue

## API Endpoints

- **Phase 1**: `POST /api/coldstart/label`, `POST /api/coldstart/label/batch`
- **Phase 2**: `POST /api/train-model`, `GET /api/training/status`, `GET /api/models`
- **Phase 3**: `GET /api/active-learning/review-queue`, `POST /api/active-learning/review-queue/mark`
- **Workflow**: `GET /api/workflow/state`, `GET /api/workflow/summary`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yestaehyung) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
