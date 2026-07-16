## the-forge

> AI-powered resume tailoring. Users upload a `.docx` resume, paste a job description, and the pipeline produces a tailored resume with skill analysis, recruiter assessment, and optional cover letter.

# CLAUDE.md — The Forge

AI-powered resume tailoring. Users upload a `.docx` resume, paste a job description, and the pipeline produces a tailored resume with skill analysis, recruiter assessment, and optional cover letter.

---

## Stack

**Backend:** Python 3.12, FastAPI, SQLModel, PostgreSQL 15, Alembic, `uv`
**Frontend:** Vue 3, TypeScript, Pinia, Vue Router, Radix Vue, SCSS
**Infra:** Docker + Docker Compose, Google Cloud Storage, Anthropic API, Gemini API

---

## Dev commands

```bash
make dev                # start full stack (docker compose up)
make test-db-up         # start isolated test DB on :5434
make test               # run pytest suite against test DB
make test-db-reset      # wipe and restart test DB after schema changes
```

| Service  | URL                        |
|----------|----------------------------|
| Frontend | http://localhost:5173      |
| Backend  | http://localhost:8000      |
| Swagger  | http://localhost:8000/docs |
| Postgres | localhost:5433             |

To run tests manually:
```bash
TEST_DATABASE_URL=postgresql://forge:forge@localhost:5433/forge_test uv run pytest -v
```

---

## Code guidelines

### Typing and enums over strings

- Always use Python enums (defined in `backend/database/models.py`) for any field with a fixed set of values. Never use raw strings for status fields, categories, roles, or types.
- In TypeScript, import and use the enums defined in `frontend/src/stores/` (e.g. `PipelineStatus`, `SkillMatchStatus`). Never compare against string literals when an enum exists.
- Declare explicit types for all function arguments, return values, and store state. Avoid `any`.
- When a prop, computed, or function parameter accepts one of a fixed set of values, define an enum or string-literal union — never widen the type to `string`.

```ts
// wrong
defineProps<{ status: string }>()

// correct — enum lives in stores/applications.ts, imported everywhere
import { PipelineStatus } from '@/stores/applications'
defineProps<{ status: PipelineStatus }>()
```

### Component-first frontend

- **Prefer extracting components over growing pages.** If a section of a view has its own state, title, or reusable surface, it belongs in `frontend/src/components/`.
- `frontend/src/views/` contains only page-level route components. Business logic and UI sections live in components.
- Before creating a new component, search `frontend/src/components/` for something that already covers the need. Reuse or extend before creating.
- Shared primitives (buttons, dialogs, spinners, badges) live in `frontend/src/components/ui/`. Add to this library rather than inlining one-off styles.

**Extract to `components/` when any of these are true:**
- The section has its own props interface.
- It contains more than a few lines of template markup.
- It could be reused on another view, even hypothetically.
- It manages its own local state.

A component defined inline inside a view file is always a signal to extract it.

### State management

| State type | Where |
|---|---|
| Auth / current user | `stores/auth.ts` |
| Applications list and detail | `stores/applications.ts` |
| Resumes list and detail | `stores/resumes.ts` |
| Toast notifications | `stores/toast.ts` |
| Form inputs, loading flags, modal open/closed | Component-local `ref()` |

Do not add new Pinia stores unless the state is genuinely shared across multiple views.

### General

- No comments unless the *why* is non-obvious. Don't comment what the code already says.
- Keep route handlers thin — delegate to repositories and services, not inline logic.
- Migrations are the source of truth for schema. Never mutate the DB outside of Alembic.

---

## Keeping docs/use-cases.md up to date

`docs/use-cases.md` is the canonical list of user-facing use cases. **Update it whenever a feature is added, changed, or removed.** This includes:

- New routes or views
- New actions in existing views (dialogs, menus, buttons)
- Removed or renamed features

Keep entries to one line per use case in the format: `- User can [action]`.

---

## Testing

### Philosophy

Integration tests that mirror real usage. If a test passes but the app is broken due to a missed DB constraint, the test has failed its purpose. Favor fewer, high-confidence tests over broad coverage.

### No-mock policy

- **Backend:** Test against a real PostgreSQL database. Never mock repository methods or services.
- **Frontend:** Use real Pinia stores. Never mock `storeToRefs` or internal state.
- **Allowed mocks (edge only):**
  - External AI APIs (Gemini / Anthropic)
  - Google OAuth callbacks
  - GCS file operations

### Backend (Pytest + SQLModel)

- Each test starts with a clean database (tables truncated by `clean_tables` fixture in `conftest.py`).
- Use factories from `backend/tests/factories.py` to seed data — do not create records inline.
- Test the API contract via `TestClient`. Assert DB side-effects by querying through the repository layer.
- AI calls are intercepted with `respx` or `unittest.mock.patch` returning static fixtures.
- Test files mirror the source tree under `backend/tests/`. A test for `backend/routers/applications.py` lives at `backend/tests/routes/test_application_routes.py`.

**Factories — pass desired state as kwargs, never mutate after creation:**

```python
# correct
user = UserFactory(is_active=False)
app = ApplicationFactory(user_id=user.id, status=PipelineStatus.FAILED)

# wrong — mutating after creation bypasses the factory defaults merge
user = UserFactory()
user.is_active = False
session.add(user)
session.commit()
```

Add a new factory to `backend/tests/factories.py` whenever a new model needs test data.

**DB assertions — query through repositories, not `session.refresh()`:**

```python
# correct
from backend.database.repositories import ApplicationRepository
persisted = ApplicationRepository(session).get_by_user_and_id(user.id, app.id)
assert persisted.status == PipelineStatus.READY

# wrong
session.refresh(app)
assert app.status == PipelineStatus.READY
```

```python
# Test the effect, not the function
def test_upload_resume_persists_to_db(client, UserFactory, session, session_cookie):
    user = UserFactory()
    resp = client.post("/api/resumes/", files=..., cookies={"session": session_cookie(str(user.id))})
    assert resp.status_code == 201
    resume = session.get(Resume, resp.json()["id"])
    assert resume.coaching_status == "analyzing"
```

### Frontend (Vitest + Testing Library) — not yet set up

- Tooling: Vitest, `@testing-library/vue`, `msw`.
- Assert on what the user sees (`findByRole`, `findByText`). Never inspect `wrapper.vm` or store state directly.
- Network: use `msw` handlers to simulate FastAPI responses.

### Workflow for new features

1. Define Given/When/Then scenarios before writing any code.
2. Write the tests first and get approval.
3. Only then implement the feature.

---
> Source: [clarabatt/the-forge](https://github.com/clarabatt/the-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
