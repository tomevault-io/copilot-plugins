## indexium

> - `app.py`: Flask web UI for tagging, manual video review, and NFO metadata plan/write/history workflows.

# Repository Guidelines

## Project Structure & Module Organization
- `app.py`: Flask web UI for tagging, manual video review, and NFO metadata plan/write/history workflows.
- `scanner.py`: Video scanning, face detection/encoding, clustering, OCR extraction, auto-classify. CLI entry point for scanner commands.
- `nfo_services.py`: Active NFO metadata engine with `NfoPlanner`, `NfoWriter`, `NfoHistoryService`, and backup/parse services.
- `metadata_services.py`: Legacy ffmpeg-comment metadata services (`MetadataPlanner`, `MetadataWriter`, etc.) retained for compatibility/tests.
- `config.py`: Env-driven settings (paths, thresholds, cores, OCR config). Centralize changes here.
- `text_utils.py`: OCR text fragment ranking/filtering via `calculate_top_text_fragments()`.
- `util.py`: File hashing for content-based video tracking.
- `signal_handler.py`: `SignalHandler` class for graceful shutdown on Ctrl+C.
- `e2e_test.py`: End-to-end pipeline test runner.
- `scripts/`: Maintenance utilities (config checks, DB stats, NFO actor migration, manual review status updates).
- `tests/`: Pytest suite with `conftest.py` fixtures and test modules for each component.
- `templates/`: Jinja2 templates for the UI (tagging, manual review, metadata preview/history).
- `thumbnails/`, `video_faces.db`: Generated at runtime (gitignored).

## Build, Test, and Development Commands
- Install deps (preferred): `uv sync`  • Alt: `pip install -e .`
- Run scanner: `INDEXIUM_VIDEO_DIR=/path python scanner.py`
- Start web UI: `python app.py` → http://localhost:5001
- Run tests: `pytest -q`  • Example single test: `pytest -q tests/test_scanner.py::test_cluster_faces_updates_ids`
- End-to-end check: `python e2e_test.py test_vids` (creates temp DB/thumbs).

### Scanner Commands
```bash
python scanner.py                        # Full scan
python scanner.py retry                  # Reprocess previously failed videos
python scanner.py cleanup                # Remove orphaned/failed thumbnails
python scanner.py refresh_ocr            # Refresh OCR for all completed videos
python scanner.py refresh_ocr HASH1 ...  # Refresh one or more specific file hashes
python scanner.py continue_ocr           # Process videos missing OCR
python scanner.py cleanup_ocr            # Remove short OCR text (default <4 chars)
python scanner.py cleanup_ocr 6          # Custom minimum length
python scanner.py ocr_diagnose           # Print OCR environment diagnostics
```

## Coding Style & Naming Conventions
- Python 3.10+. Follow PEP 8 (4-space indents, 100–120 col soft limit).
- Use snake_case for functions/variables, PascalCase for classes, module names in lowercase.
- Add concise docstrings for public functions and routes; prefer type hints where practical.
- Keep functions small and side-effect-aware; prefer explicit over implicit configuration via `config.py`.

## Testing Guidelines
- Framework: Pytest. Place tests under `tests/` as `test_*.py`; name tests `test_*`.
- Use fixtures/monkeypatching to point DB/paths to temp locations (see `tests/conftest.py`).
- Run locally with `pytest -q`; target specific tests during development for speed.
- Coverage: `pytest --cov --cov-report=term-missing` • HTML report: `pytest --cov --cov-report=html`
- Test files: `test_app.py`, `test_scanner.py`, `test_nfo_services.py`, `test_metadata_services.py`, `test_metadata_writer.py`, `test_config.py`, `test_text_utils.py`, `test_util.py`, `test_signal_handler.py`, `test_e2e.py`, `test_e2e_ui.py`.

## Commit & Pull Request Guidelines
- Commit messages: imperative mood, concise summary (e.g., "Add pagination for tag_group"). Optional scope tags (e.g., `test:`) are welcome.
- PRs should include: clear description, rationale, testing steps, and screenshots/GIFs for UI changes.
- Link related issues; keep diffs focused. Update `README.md`/`config.py` docstrings when adding new settings.

## Security & Configuration Tips
- Do not commit generated assets: `video_faces.db`, `thumbnails/`, `.env` (already in `.gitignore`).
- Required tools: ffmpeg (includes ffprobe), OpenCV/dlib system deps (see README).
- For OCR: EasyOCR (preferred) or Tesseract as fallback.
- Key env vars: `INDEXIUM_VIDEO_DIR`, `INDEXIUM_DB`, `FLASK_DEBUG`, `INDEXIUM_OCR_ENABLED`, `INDEXIUM_OCR_ENGINE`, `MANUAL_VIDEO_REVIEW_ENABLED`, `METADATA_PLAN_WORKERS`, `NFO_REMOVE_STALE_ACTORS`, `NFO_BACKUP_MAX_AGE_DAYS`.

## Manual UI Testing with Playwright

To verify UI changes, use Playwright MCP tools to interact with the running app.

1. **Check if app is running**: `curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:5001/ || echo "not running"`
2. **Start app in background (if needed)**: `python app.py &` — track whether you started it
3. **Navigate and interact** using Playwright MCP tools:
   - `browser_navigate` — go to a URL (e.g., `http://127.0.0.1:5001/videos/manual`)
   - `browser_snapshot` — capture accessibility snapshot for page structure and element refs
   - `browser_take_screenshot` — visual screenshot to verify layout/styling
   - `browser_click`, `browser_type`, `browser_fill_form` — interact with elements
4. **Shut down app (if you started it)**: `pkill -f "python app.py"`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jrajasekera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jrajasekera)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
