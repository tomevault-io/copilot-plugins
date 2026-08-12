## birdbinder

> > Read this before making ANY changes to this codebase. It will save you from repeating our mistakes.

# BirdBinder — Agent Development Guide

> Read this before making ANY changes to this codebase. It will save you from repeating our mistakes.

## Project Overview

BirdBinder is a self-hosted PWA that turns bird sighting photos into collectible digital cards. Users submit photos → AI identifies the species → cards are generated with AI art → users collect, trade, and organize cards in binders.

## Branching Strategy

| Branch | Purpose | CI Build |
|--------|---------|----------|
| `dev` | Day-to-day development. All PRs target here. | `ghcr.io/zarguell/birdbinder:dev` + SHA |
| `main` | Stable. Merge from dev to release. | `ghcr.io/zarguell/birdbinder:latest` + `vYYMMDD` + SHA |
| `v*` tags | Versioned releases after merging to main. | `ghcr.io/zarguell/birdbinder:X.Y.Z` + `X.Y` + SHA |

**Workflow:**
1. Develop on `dev` (or create PRs targeting `dev`)
2. When ready, merge to main: `git checkout main && git merge dev && git push`
3. Tag a release: `git tag vX.Y.Z && git push --tags` (or create a GitHub Release)
4. Renovate creates dependency PRs against `dev`

**Local dev:** Use `docker compose build` / `docker compose up` for local development. CI builds are for deployed environments.

## Repository & Architecture

```
birdbinder/
├── backend/                  # FastAPI application
│   ├── app/
│   │   ├── main.py           # App entry, CORS, router mounting, SPA serving
│   │   ├── config.py         # pydantic-settings (Settings class)
│   │   ├── db.py             # Async engine, Base, get_db()
│   │   ├── auth.py           # CF JWT decode + API key validation
│   │   ├── dependencies.py   # get_current_user() dependency
│   │   ├── ensure_schema.py  # Safety net for missing columns
│   │   ├── huey_instance.py  # SqliteHuey (data/huey.db)
│   │   ├── image.py          # HEIF→JPEG, EXIF extraction, thumbnails
│   │   ├── storage.py        # File save/retrieve helpers
│   │   ├── data/             # Static JSON (birds.json, regions.json)
│   │   ├── models/           # SQLAlchemy models (14 files)
│   │   ├── schemas/          # Pydantic request/response models
│   │   ├── routers/          # 11 router files
│   │   └── services/         # 10 service files
│   ├── migrations/           # Alembic migrations
│   └── tests/                # 25 test files
├── frontend/                 # SvelteKit SPA (adapter-static → backend/app/static/)
│   ├── src/lib/api.ts        # Typed API client
│   └── src/routes/           # 13 page routes
├── Dockerfile                # Multi-stage: Node 20 → Python 3.13
├── docker-compose.yml
└── entrypoint.sh             # alembic → ensure_schema → uvicorn + huey
```

## Tech Stack

| Layer | Tech |
|-------|------|
| Backend | Python 3.13, FastAPI, SQLAlchemy 2.0 (async), Alembic, Pydantic v2 |
| Database | SQLite (aiosqlite async) |
| Tasks | Huey (SqliteHuey, 2 process workers) |
| Frontend | Svelte 5 (runes), SvelteKit 2, Tailwind CSS 4, TypeScript |
| AI | OpenAI-compatible API (vision + image gen) |
| Auth | CF_Authorization JWT + Bearer API key |
| Build | Docker multi-stage, uv for Python deps, npm for frontend |

## Development Workflow

### Running Locally

```bash
# Backend (from backend/)
uv run uvicorn app.main:app --reload

# Frontend (from frontend/)
npm run dev

# Tests
uv run pytest tests/ -v

# Migrations
uv run alembic upgrade head
```

### TDD — ALWAYS Write Tests First

1. **RED** — Write a failing test that reproduces the bug or specifies the new behavior
2. **GREEN** — Write the minimal code to pass
3. **REFACTOR** — Clean up while tests stay green

Test command: `cd backend && .venv/bin/python -m pytest tests/test_FILE.py -v`

### Making Changes

1. Write the test first
2. Implement
3. Run the full test suite: `.venv/bin/python -m pytest tests/ -v`
4. Create migration if schema changed: `alembic revision --autogenerate -m "description"`
5. Commit and push

## Database Schema

### Tables & Relationships

```
users (email-based profiles)
  │
sightings ──1:N──→ cards ──1:N──→ binder_cards ←──N:1── binders
    │                  │                                      │
    └──1:N──→ jobs    └── referenced by ──→ activities ←──1:N── likes
                                                                           └──1:N── comments
card_sets (card_targets as JSON array of card IDs)
trades (offered_card_ids, requested_card_ids as JSON)
species_cache (static eBird taxonomy lookup)
app_settings (key-value store)
```

### FK Cascade Summary — CRITICAL

| FK | ondelete CASCADE? | ORM cascade? | Notes |
|----|-------------------|--------------|-------|
| Card.sighting_id → Sightings.id | ✅ | ✅ | Working |
| Job.sighting_id → Sightings.id | ✅ | ✅ | Working |
| BinderCard.binder_id → Binders.id | ✅ | ✅ | Working |
| BinderCard.card_id → Cards.id | ✅ | ❌ | **Manual cleanup needed** |
| Comment.activity_id → Activities.id | ✅ | ✅ | Working |
| Like.activity_id → Activities.id | ✅ | ✅ | Working |
| Activity.reference_id | **NO FK** | ❌ | **Loose string, always manual** |
| Trade card ID arrays | **NO FK** | ❌ | JSON arrays |
| CardSet.card_targets | **NO FK** | ❌ | JSON array |
| Binder.cover_card_id | **NO FK** | ❌ | Loose string |

### Enums

```python
PoseVariant: perching, flying, swimming, foraging, singing, nesting, courtship, other
SightingStatus: pending, identified, failed, cancelled
JobType: identify, generate_card, regenerate_art
JobStatus: pending, running, completed, failed
TradeStatus: pending, accepted, declined, cancelled
```

---

## ⚠️ GOTCHAS — Read These Before Changing Anything

### 1. SQLite FK Enforcement Is Off

SQLite does NOT enforce foreign key constraints unless `PRAGMA foreign_keys=ON` is set. **This app does not set it.**

**Consequence:** `ondelete="CASCADE"` in ForeignKey definitions is decorative only. The actual cascading happens through SQLAlchemy ORM `cascade="all, delete-orphan"` on relationships.

**Rule:** Any delete endpoint MUST manually clean up cross-table references that don't have ORM cascade:
- Activities (loose `reference_id` string — no FK)
- BinderCards (FK exists but no ORM relationship from Card side)
- Trades/CardSets (card IDs stored as JSON arrays)

**Example** (from sightings delete):
```python
# 1. Collect card IDs before deleting
card_ids = [c.id for c in sighting.cards]

# 2. Delete binder_cards manually (no ORM cascade from Card side)
if card_ids:
    await db.execute(BinderCard.__table__.delete().where(BinderCard.card_id.in_(card_ids)))

# 3. Delete activities by reference_id (no FK at all)
await db.execute(Activity.__table__.delete().where(Activity.reference_id == sighting_id, ...))

# 4. NOW delete sighting (ORM cascade handles cards and jobs)
await db.delete(sighting)
```

### 2. Huey Tasks Use Synchronous SQLAlchemy

Huey workers are synchronous. Services that run as Huey tasks (`identifier.py`, `card_gen.py`) create their own `_sync_engine` by replacing `sqlite+aiosqlite` with `sqlite` in the database URL:

```python
_sync_db_url = settings.database_url.replace("sqlite+aiosqlite", "sqlite")
_sync_engine = create_engine(_sync_db_url)
# Use synchronous Session(_sync_engine), NOT AsyncSession
```

They call async AI functions via `asyncio.run()`.

**Rule:** Don't try to use `AsyncSession` in Huey tasks. Use the sync engine pattern.

### 3. Image Processing — Always Convert Before Sending

Images come in various formats (HEIF, PNG with alpha, etc.). Before sending to ANY external API or saving thumbnails:

```python
from PIL import Image as PILImage

MAX_DIMENSION = 1024

img = PILImage.open(file)
img.thumbnail((MAX_DIMENSION, MAX_DIMENSION), PILImage.LANCZOS)

# MUST convert to RGB — APIs don't accept RGBA
if img.mode in ("RGBA", "LA", "P"):
    img = img.convert("RGB")

buf = io.BytesIO()
img.save(buf, format="JPEG", quality=85)
```

**We've been burned twice:** once in `call_vision_model()` (fixed), once in `_generate_image_to_image()` (fixed). If you add any new image-to-API path, include this conversion.

### 4. HEIF/HEIC — EXIF Before Conversion

When handling HEIF uploads (`app/image.py`), extract EXIF data BEFORE converting to JPEG. Pillow strips EXIF during `save()`.

### 5. AI Response Parsing Is Fragile

AI models return JSON in unpredictable formats. The parser in `ai.py` handles:
- Clean JSON
- Markdown code fences (` ```json ... ``` `)
- Prose with embedded JSON blocks
- Balanced-brace extraction for nested JSON
- Reasoning models (Kimi K2) that put analysis in `message.reasoning` with empty `content`

**Rule:** Always use the existing parsing functions. Don't try to `json.loads()` raw AI output directly.

### 6. Pydantic Circular Schemas

`SightingRead` contains `list[CardRead]` and `CardRead` contains optional `SightingRead` (via the sighting relationship). This was resolved with a base class pattern — see `schemas/sighting.py` for `SightingInfo` / `SightingRead`.

**Rule:** If you add a new model that references another model which references it back, use the base-class extraction pattern.

### 7. Auth Fallback Chain — Don't Break It

```
1. Cf-Access-Jwt-Assertion header → CF_Authorization header → CF_Authorization cookie
2. If CF token found but can't decode AND cf_access_enabled=true → 401
3. Authorization: Bearer <key> → validate against API_KEYS env var
4. No auth configured → "local-user" (dev mode)
5. Otherwise → 401
```

**Rule:** All protected endpoints use `Depends(get_current_user)`. It returns a **string** (email or `api-key:XXXX` or `local-user`). Don't add auth logic directly in routers.

### 8. ensure_schema.py Is a Safety Net, Not a Migration

`ensure_schema.py` runs at startup after Alembic. It auto-adds missing **nullable** columns. It does NOT handle NOT NULL columns, new tables, or constraint changes.

**Rule:** For schema changes, always create a proper Alembic migration. ensure_schema is defense-in-depth only.

### 8b. SQLite Alembic Migrations: No Named FK Constraints

SQLite does not name foreign key constraints. Alembic's `batch_alter_table` with `drop_constraint('some_fk_name', type_='foreignkey')` will fail with `KeyError` because SQLite has no FK names to reference.

**Rule:** To modify an existing FK (e.g., add `ondelete='CASCADE'`), use `recreate='batch_alter_table'` to rebuild the table:

```python
with op.batch_alter_table('mytable', recreate='batch_alter_table') as batch_op:
    batch_op.create_foreign_key(
        'fk_name', 'other_table', ['col'], ['id'],
        ondelete='CASCADE',
    )
```

`recreate='always'` tells Alembic to create a new table, copy data, swap, and drop — which naturally picks up the new FK definition from the model.

### 9. Test API Key Is Hardcoded

Tests use `TEST_API_KEY="***"` (literal string). Auth is mocked via `unittest.mock.patch` on `app.dependencies.validate_api_key` and `app.auth.validate_api_key`.

**Rule:** When writing new tests that hit authenticated endpoints, use the `auth_client` fixture. Don't try to test real auth in unit tests.

### 10. In-Memory SQLite for Tests

Tests use `sqlite+aiosqlite:///:memory:` with `StaticPool`. Each test function gets a fresh DB via `create_all`. This means:
- No cross-test data leakage
- No Alembic migrations run (models create tables directly)
- FK constraints are NOT enforced (same as production)

**Rule:** Tests must clean up after themselves if they share a `db_session` fixture. The `db_engine` fixture is function-scoped so you get a clean DB each time.

### 11. File Cleanup on Delete Is Missing

When deleting sightings, the photo and thumbnail files on disk are **not** deleted. This is a known gap.

**Rule:** If you add file cleanup to any delete endpoint, use `storage.py` helpers and handle `FileNotFoundError` gracefully.

### 12. Binder.cover_card_id Is a Loose String

`Binder.cover_card_id` has no FK constraint. It can reference a deleted card.

**Rule:** When deleting cards, check if any binder uses it as `cover_card_id` and null it out.

### 13. AI Art Prompts: Describe the Component, Not the Product

The AI generates a **bird illustration** — the app composites it into a card with borders, holographic effects, text overlays, etc. Do NOT tell the AI it's making a "trading card" — weaker models will draw borders, text, and card frames that clash with the app's own chrome.

**Rule:** Prompts should describe only what the AI needs to produce: the bird illustration, the style, and a clean background. Explicitly say "Do NOT add any text, borders, frames, or card-like elements." See `TEXT_TO_ART_PROMPT` and `IMAGE_TO_ART_PROMPT` in `ai.py`.

### 14. DB Setting Overrides Must Be Passed Explicitly

Settings like `ai_image_model` can be overridden in the `app_settings` DB table. Card generation reads these overrides in `_run_card_generation` and `_run_card_art_regeneration`, but each call site must explicitly pass them to `generate_card_art()`. Adding a new override? Check every call site.

**We were burned:** `_run_card_art_regeneration` read `image_model_override` from DB but forgot to pass it to `generate_card_art()`, so regeneration always used the default env var model.

### 15. eBird API Integration Is a Stub

`fetch_region_frequencies()` returns `{}`. Rarity tiers use static family-based heuristics with deterministic hash-based variation. The real eBird API call is commented out.

**Rule:** Don't assume rarity tiers are based on real observation data. They're deterministic from species_code + family.

### 16. Lazy Loading Crashes in Async Endpoints

SQLAlchemy's `default` lazy loading strategy emits synchronous SQL, which raises `MissingGreenlet` or similar errors inside async endpoints. **Line 195 in `sightings.py`** has this exact bug — `sighting.cards` triggers a lazy load on an async session that's already expired.

**Rule:** Always use `selectinload()` or `joinedload()` for relationships you need to access in async endpoints. Import from `sqlalchemy.orm`:
```python
from sqlalchemy.orm import selectinload
sighting = (await db.execute(
    select(Sighting)
    .options(selectinload(Sighting.cards))
    .where(Sighting.id == sighting_id)
)).scalar_one_or_none()
```

**Affected endpoints:** `DELETE /api/sightings/{id}` (line 195) — known to return 500 in production.

---

## API Conventions

- All endpoints prefixed with `/api`
- Pagination: `?limit=20&offset=0` → `{"items": [...], "total": N, "limit": N, "offset": N}`
- Auth: `Authorization: Bearer <key>` or CF_Authorization JWT
- File uploads: `multipart/form-data`
- Response models: Pydantic v2 with `model_config = {"from_attributes": True}`
- List endpoints return `{items, total, limit, offset}`
- Delete endpoints return `204 No Content` (no body)
- Static files: `/storage/*` for uploads
- Docs: `/api/docs`, `/api/redoc`

## Testing Patterns

```python
# Always use auth_client fixture for authenticated tests
async def test_something(auth_client, db_session):
    # Create test data directly in db_session
    sighting = Sighting(id=str(uuid.uuid4()), user_identifier=TEST_USER, ...)
    db_session.add(sighting)
    await db_session.commit()

    # Make API call
    resp = await auth_client.get(f"/api/sightings/{sighting.id}")
    assert resp.status_code == 200

    # Verify with direct DB query
    result = await db_session.execute(select(Sighting).where(Sighting.id == sighting.id))
    assert result.scalar_one_or_none() is not None
```

### Key Fixtures

| Fixture | Scope | Purpose |
|---------|-------|---------|
| `auth_client` | function | Authenticated httpx AsyncClient with in-memory DB override |
| `client` | function | Unauthenticated AsyncClient |
| `db_session` | function | Async SQLAlchemy session (in-memory SQLite) |
| `db_engine` | function | Async engine (in-memory, StaticPool) |
| `sighting` | function | Pre-created test sighting |
| `auth_headers` | function | Dict with Authorization header |

## Deployment

The Dockerfile builds frontend (Node 20) then backend (Python 3.13). Entrypoint:
1. Fix volume permissions
2. `alembic upgrade head`
3. `ensure_schema.py`
4. `uvicorn` (port 8000) + `huey_consumer` (2 workers)

Volumes: `./backend/data:/app/data` (DBs), `./backend/storage:/app/storage` (uploads)

**CI/CD:** GitHub Actions builds and pushes to GHCR on every push to `dev`, `main`, and `v*` tags. See [Branching Strategy](#branching-strategy) for details.

## Checklist Before Changing Anything

- [ ] Did I write the test first?
- [ ] Does my change involve deleting records? → Check FK cascade table above
- [ ] Does my change involve images? → Use the Pillow conversion pattern
- [ ] Does my change involve Huey tasks? → Use sync SQLAlchemy, not async
- [ ] Does my change touch auth? → Use the dependency, don't reinvent
- [ ] Does my change add a new table/column? → Create Alembic migration
- [ ] Did I run the full test suite after? → `.venv/bin/python -m pytest tests/ -v`

---
> Source: [zarguell/birdbinder](https://github.com/zarguell/birdbinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
