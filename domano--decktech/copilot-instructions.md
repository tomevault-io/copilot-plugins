## decktech

> - `cmd/similarityd/`: Go REST server (`/similar`, `/healthz`, `/config`).

# Repository Guidelines

## Project Structure & Module Organization
- `cmd/similarityd/`: Go REST server (`/similar`, `/healthz`, `/config`).
- `cmd/decktech/`: Bubble Tea TUI for import/batching (Download, Schema, Single/Continuous, Clean, Re-embed, Config).
- `cmd/deckbrowser/`: Bubble Tea TUI to browse/search and run similarity queries.
- `cmd/web/`: SSR web app (search, browse, detailed card view with images/legalities/keywords/sets, all printings, similar).
- `scripts/`: Data, embedding, ingestion, and maintenance scripts.
- `weaviate/`: Schema (`schema.json`). `ops/`: Docker Compose for Weaviate.
- `docs/`, `README.md`, `ARCHITECTURE.md`: Design, setup, and flows.
- `pkg/weaviateclient/`: shared typed GraphQL client for Card queries/search.
- `pkg/progress/`: embedding checkpoint utilities (`next_offset`, totals).

## Build, Test, and Development Commands
- DB up/down: `make weaviate-up` / `make weaviate-down`
- Schema: `make schema-apply`
- Data: `make data-download`
- Sample embed+ingest: `make embed-sample ingest-sample`
- Continuous batches: `make embed-batches BATCH=1000` (resumes via checkpoint)
- Clean/restart: `make clean-embeddings` (wipes local batches/checkpoint; suggests DB reset)
- Server: `make run` (uses `WEAVIATE_URL`)
- TUIs: `make tui` (importer) / `make browser` (DB browser)
- Web SSR: `make web` (runs on :8090)

## Coding Style & Naming Conventions
- Go: `gofmt`-formatted; small handlers/helpers; env via `WEAVIATE_URL`.
- Bubble Tea: simple Update/View; stream logs with `tea.Println`; avoid blocking the UI.
- Python: PEP 8; pure functions; avoid global state; scripts runnable standalone.
- Templates: minimal HTML/CSS; avoid inline JS.
- Files/dirs: lowercase with hyphens/underscores; JSON config minified unless hand-edited.

## Testing Guidelines
- Current: manual/e2e via TUIs and web (GraphQL readiness, `/similar`, browse/search, similar from card pages).
- Additions welcome: unit tests for `cmd/similarityd` (vector averaging, query building) + script fixtures for embed/extract tags.
- Test naming: `<package>_test.go`, `test_*` functions; keep tests fast and deterministic.

## Commit & Pull Request Guidelines
- Keep PRs small and focused; include a clear description and test evidence (commands/output snippets).
- Prefer Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.
- Link issues where applicable; note any network/download requirements (models, Docker images).

## Security & Configuration Tips
- No secrets required; service uses `WEAVIATE_URL` (default `http://localhost:8080`).
- Weaviate: `vectorizer: none`, cosine distance (see `weaviate/schema.json`).
- Embedding config: Tags weight emphasized via `EMBED_TAGS_WEIGHT` (configurable in TUI).
- Large downloads (models, bulk JSON) — commit only code and small configs, not data.

## Architecture Overview
- See `ARCHITECTURE.md` for diagrams and flows; core service code lives in `cmd/similarityd/main.go` (name lookup → vector average → nearVector). UIs: `cmd/decktech`, `cmd/deckbrowser`, `cmd/web`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/domano)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/domano)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
