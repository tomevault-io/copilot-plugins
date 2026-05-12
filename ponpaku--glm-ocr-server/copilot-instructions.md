## glm-ocr-server

> - This repository is a standalone GLM-OCR server.

# AGENTS.md

## Project

- This repository is a standalone GLM-OCR server.
- Base model: `zai-org/GLM-OCR`

## Core Files

- Backend: `app/main.py`
- Frontend UI: `app/static/index.html`
- Linux/macOS launcher: `run.sh`
- Windows launcher: `run.bat`

## Runtime and Dependencies

- Python: 3.10+
- API server: FastAPI + Uvicorn
- Model stack: `transformers`, `torch`, `torchvision`, `accelerate`
- PDF rendering: `pypdfium2` (do not switch to `fitz`/PyMuPDF)

## Model and Cache Policy

- Keep model/cache inside project directory.
- Primary cache paths:
  - `models/hf_cache`
  - `models/hf_home`
- Respect `GLM_MODEL_CACHE` when set.

## API Contracts

- `GET /api/status`
- `POST /api/analyze`
- `GET /api/progress/{request_id}`
- `POST /api/cancel/{request_id}`

`/api/analyze` accepts:
- `file`, `device`, `dpi`, `task`, `linebreak_mode`, `schema`, `max_new_tokens`, `temperature`, `request_id`

## Current Functional Behavior

- Tasks: `text`, `table`, `formula`, `extract_json`
- Linebreak modes:
  - `none`
  - `paragraph`
  - `compact`
- Cancellation:
  - Uses cancel API + generation-side stopping criteria
  - Stops near-immediately at token generation boundaries
- Progress messages:
  - preprocessing
  - `i/n` page OCR
  - done / error / canceled
- UI page view:
  - dropdown selector
  - `ALL` aggregated view
- Copy behavior:
  - Copies currently selected view
  - `raw` is excluded from copied content

## Environment Configuration

- `run.sh` and `run.bat` load `.env` from project root when present.
- Main variables:
  - `HOST`
  - `PORT`
  - `TORCH_CHANNEL`

## Release and Repo Hygiene

- Keep `LICENSE` as MIT.
- Keep third-party summary in `THIRD_PARTY_NOTICES.md`.
- Keep `.gitignore` aligned with:
  - virtualenv artifacts
  - model/cache directories
  - OS/editor temporary files

---
> Source: [ponpaku/GLM-OCR-server](https://github.com/ponpaku/GLM-OCR-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
