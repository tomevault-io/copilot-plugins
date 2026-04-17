## project-miri

> This project, tentatively titled **Worldview**, is an AI-driven friendship and dating platform that prioritizes privacy through a zero-knowledge architecture. The core concept is matching users based on their "worldview" (derived from AI-generated personality embeddings) rather than photos.

# GEMINI Context: Worldview AI Platform

## Project Overview
This project, tentatively titled **Worldview**, is an AI-driven friendship and dating platform that prioritizes privacy through a zero-knowledge architecture. The core concept is matching users based on their "worldview" (derived from AI-generated personality embeddings) rather than photos.

**Note:** The current workspace is in a preliminary state. While the `README.md` describes a comprehensive FastAPI and PostgreSQL (pgvector) backend, the actual source files (the `app/` directory, `requirements.txt`, etc.) are not yet present in the repository. The only tracked file is a boilerplate `index.html`.

### Primary Technologies (Target)
- **Backend:** FastAPI (Python)
- **Database:** PostgreSQL with `pgvector` for similarity search
- **AI Integration:** OpenAI (Embeddings for matching, DALL-E 3 for personality portraits)
- **Frontend:** Jinja2 templates with Vanilla CSS
- **Authentication:** JWT with zero-knowledge profile storage

## Building and Running
The following commands are inferred from the `README.md` for the target architecture:

### Manual Setup
```bash
# 1. Start Postgres with pgvector
docker run -d --name worldview-db -e POSTGRES_USER=worldview -e POSTGRES_PASSWORD=worldview -e POSTGRES_DB=worldview -p 5432:5432 pgvector/pgvector:pg16

# 2. Activate Environment
source venv/bin/activate

# 3. Install Dependencies (TODO: Create requirements.txt)
pip install -r requirements.txt

# 4. Run Application
uvicorn app.main:app --reload --port 8000
```

### Docker Setup
```bash
docker-compose up -d
```

## Development Conventions
- **Zero-Knowledge Privacy:** Raw user input must never be stored on disk or in the database. Process inputs into embeddings in memory and delete the source text immediately.
- **Async First:** The architecture favors asynchronous operations (FastAPI, SQLAlchemy asyncpg).
- **Environment Configuration:** All sensitive data (OpenAI keys, JWT secrets) must be managed via `.env` files.

## Key Files (Current)
- `README.md`: Contains the architectural blueprint and roadmap for the MVP.
- `index.html`: A placeholder HTML file (currently the only tracked source file).
- `venv/`: Pre-configured Python virtual environment containing various utility libraries (e.g., `aiogram`, `yt-dlp`, `matplotlib`).

## Current Task Status
The project is at the very beginning of its lifecycle. The immediate next steps involve scaffolding the `app/` directory and setting up the database models as described in the `README.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/monules) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
