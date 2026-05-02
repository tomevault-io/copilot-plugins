## rapidly

> This reference captures the conventions, architecture, and workflow knowledge that AI agents need when contributing to the Rapidly codebase. Treat it as your onboarding guide to the project.

# Rapidly — Agent Development Reference

This reference captures the conventions, architecture, and workflow knowledge that AI agents need when contributing to the Rapidly codebase. Treat it as your onboarding guide to the project.

## Architecture Overview

Rapidly is a file-sharing and paid-content distribution platform. The codebase is structured as a monorepo with three main areas:

### Backend (`server/`)

A Python application powered by **FastAPI**. Core source lives under `server/rapidly/`, organized into domain modules. **PostgreSQL** is the primary data store, accessed through **SQLAlchemy** (ORM). Database models are centralized in `server/rapidly/models` rather than scattered across modules. Background processing uses **Dramatiq** workers.

### Frontend (`clients/`)

A **Next.js** application orchestrated with **Turborepo** and **pnpm**:

- `clients/apps/web/` — the main dashboard where users manage channels, shares, and payments.
- `clients/packages/ui/` — reusable React component library (Radix UI + Tailwind CSS).
- `clients/packages/client/` — auto-generated TypeScript API client and data-fetching hooks.

### Supporting Directories

- `dev/` — development scripts and tooling.
- `docs/` — developer and user documentation, built with Mintlify (https://mintlify.com/docs/llms.txt).

## File Sharing Domain

Rapidly's core domain revolves around secure file sharing with optional monetization. Key concepts:

- **Channels**: A channel is a container owned by a user or organization. It groups related shares together and defines access policies and payment settings.
- **Paid Shares**: Files or content bundles that require payment before the recipient can access them. Payments flow through Stripe Connect, with the channel owner as the connected account.
- **Secret Exchange**: A protocol for sharing sensitive content that ensures data is encrypted in transit and at rest. The server never holds plaintext secrets — it only brokers the handshake.
- **WebRTC Signaling**: For direct peer-to-peer file transfers, the backend acts as a signaling server. It coordinates the connection setup between sender and receiver without relaying the actual file data.

## Authentication

The backend implements a custom auth layer on top of FastAPI's dependency injection.

### Core Concepts

- **`AuthSubject[T]`** — represents the authenticated caller. `T` may be `User`, `Organization`, `Customer`, or `Anonymous`. Inject it into endpoint signatures; endpoints without an `auth_subject` parameter are publicly accessible.
- **Scopes** — fine-grained permissions attached to each `AuthSubject`. An `Authenticator` declares which scopes it requires; access is granted when the subject holds at least one matching scope.

### Per-Module Authenticators

Most modules define their own authenticators in an `auth.py` file, specifying the required scopes and allowed subject types:

```python
# server/rapidly/share/auth.py
_ShareWrite = Authenticator(
    required_scopes={Scope.web_default, Scope.shares_write},
    allowed_subjects={User, Organization},
)
ShareWrite = Annotated[AuthSubject[User | Organization], Depends(_ShareWrite)]
```

### Built-in Authenticator Shortcuts

For endpoints tied to the web dashboard or internal backoffice, use the predefined dependencies from `server/rapidly/auth/dependencies.py`:

- `WebUser` — requires a logged-in user (`AuthSubject[User]`).
- `WebUserOrAnonymous` — accepts either an authenticated user or an anonymous visitor.
- `AdminUser` — restricted to users with admin privileges.

### Usage Example

```python
from rapidly.models import User
from rapidly.share.auth import ShareWrite

@router.post("/shares")
def create_share(auth_subject: ShareWrite) -> Share:
    # Access restricted to users/orgs with web_default or shares_write scope
    ...
```

### Credential Resolution Order

The system resolves the caller's identity by checking, in order: customer session token, member session token, user session cookie, Workspace Access Token, OAuth2 token. When none match, the subject defaults to `Anonymous`. The endpoint's authenticator then decides whether the resolved subject type and scopes are sufficient.

## Code Quality Standards

- Write self-documenting code — add comments only when the intent cannot be expressed through naming and structure.
- Choose descriptive names for variables, functions, and classes.
- Adhere to established conventions and industry best practices.
- All new code should be maintainable and follow SOLID principles.
- Limit changes to what the current task requires; avoid drive-by refactors of unrelated code.

## Backend Development

### Module Layout

Each domain lives in its own directory under `server/rapidly/`. A typical module includes:

| File | Purpose |
|------|---------|
| `endpoints.py` | FastAPI route handlers |
| `service.py` | Business logic, encapsulated in service classes |
| `schemas.py` | Pydantic models for request/response validation |
| `repository.py` | Database access through SQLAlchemy queries |

SQLAlchemy models are the exception — they live centrally in `server/rapidly/models` instead of inside each module.

After changing any API surface, regenerate the frontend client by running `pnpm run generate` inside `clients/packages/client`.

### Import Rules

- Place all imports at the top of the file. Never import inside a function or method body.
- Reference models via `rapidly.models`, and import services from their respective modules.
- Wire up repositories and services through FastAPI's dependency injection.

### Repository Layer

- Prefer passing full domain objects (e.g., `Share`) rather than bare UUIDs when the caller already has the object loaded.

    ```python
    # Good — caller already holds the share object
    async def revoke_access(self, share: Share, *, flush: bool = False) -> Share:
        return await self.update(share, update_dict={"revoked_at": utc_now()}, flush=flush)

    # Avoid — forces an unnecessary lookup
    async def revoke_access(self, share_id: UUID) -> None:
        # ...
    ```

- Return the updated object so callers always have the latest state.
- Expose a keyword-only `flush: bool = False` parameter for explicit transaction control.
- Extend `RepositoryBase` to inherit shared query helpers.

### Service Layer

- Map errors to appropriate HTTP status codes: `409` for conflicts, `422` for validation failures, `404` for missing resources.

    ```python
    class ShareAlreadyRevoked(ShareError):
        def __init__(self, share: Share) -> None:
            self.share = share
            message = f"Share {share.id} has already been revoked"
            super().__init__(message, 409)
    ```

- Embed entity identifiers and context in error messages to aid debugging.
- Use `async`/`await` consistently with SQLAlchemy's async session.
- Obtain database sessions through dependency injection — never instantiate them manually.

### Exception Design

- Derive custom exceptions from domain-specific base classes (e.g., `ShareError`, `ChannelError`).
- Supply the HTTP status code as the second argument to the base constructor.
- Attach the relevant domain object as an attribute for downstream error handling.

### Database and ORM Practices

- Favor SQLAlchemy ORM operations over raw SQL.
- Use `AsyncSession` provided via dependency injection.
- Keep all query logic inside repository classes that inherit from `RepositoryBase`.

Avoid calling `session.commit()` in business logic. The framework handles commits automatically: the API layer commits after each request completes, and background workers commit after each task finishes. This guarantees the database stays consistent when exceptions occur. If you find yourself needing an explicit commit, document the reason clearly.

When you need database-generated values (constraints, defaults) to be available immediately, call `session.flush()`. Note that SQLAlchemy automatically flushes before read operations, so an explicit flush is often unnecessary.

### Testing

- Mirror the source tree in the test directory: `server/rapidly/foo/endpoints.py` maps to `tests/foo/test_endpoints.py`.
- Rely on existing fixtures instead of manually constructing data they already provide.
- Name test methods to describe the exact behavior under test.
- Group tests into classes — one class per method being tested, one method per scenario.
- `test_task` and `test_endpoints` files are end-to-end tests that exercise real database interactions without mocking. Use them to verify actual application behavior.
- Mock external services (Stripe, etc.) with `MagicMock`, following the patterns already in the codebase.
- Use `SaveFixture`, `AsyncSession`, `pytest.mark.asyncio`, and the other established test utilities.

## Frontend Stack

- Features are grouped into directories that mirror the domain.
- **Key page locations**:
    - `apps/web/src/app/(main)/dashboard` — the logged-in user's home.
    - `apps/web/src/app/(main)/[organization]` — per-organization views.
- **Data layer**: **TanStack Query** powers all server state. Hooks and the API client come from the `@rapidly-tech/client` package, auto-generated from the backend's OpenAPI spec.
- **Global state**: managed with **Zustand** stores.
- **Component library**: prefer components from `clients/packages/ui` (Tailwind CSS + Radix UI) over ad-hoc implementations.
- **Styling**: **Tailwind CSS** exclusively.
- Follow React and Next.js best practices throughout.

## Local Development

Detailed setup instructions live in `DEVELOPMENT.md`.

### Starting Services

- **Backend** (from `server/`): open two terminals —
    - `uv run task api` — launches the FastAPI server at http://127.0.0.1:8000
    - `uv run task worker` — launches the Dramatiq background worker
- **Frontend** (from `clients/`): run `pnpm run dev` to start the Next.js dev server at http://127.0.0.1:3000

### Database Migrations

Alembic manages schema migrations under `server/migrations/`. Apply pending migrations with:

```bash
uv run task db_migrate
```

To auto-generate a migration from model changes:

```bash
uv run alembic revision --autogenerate -m "<description>"
```

To create a blank migration (useful for data migrations):

```bash
uv run alembic revision -m "<description>"
```

### Quality Checks

Run linting and type checking for the backend:

```bash
uv run task lint && uv run task lint_types
```

Run the full test suite:

```bash
uv run task test
```

Target a specific file:

```bash
uv run pytest tests/path/to/test_file.py
```

Target a specific class or method:

```bash
uv run pytest tests/path/to/test_file.py::TestClassName::test_method_name
```

### Python Environment Note

**Always prefix Python commands with `uv run`.** This guarantees the correct Python version (3.14), project dependencies, environment variables, and virtual environment are all in effect.

## Key Integrations

- **Stripe**: Powers payments for paid file shares via Stripe Connect. Channel owners are onboarded as connected accounts, and the platform facilitates payouts. Requires `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, and `STRIPE_WEBHOOK_SECRET` in `server/.env`.
- **GitHub**: Provides OAuth-based authentication and repository integration. Requires a GitHub App configured for local development.

## Documentation

Project documentation lives in `docs/` and is built with Mintlify. To preview locally:

```bash
pnpm dev
```

---
> Source: [rapidly-tech/rapidly](https://github.com/rapidly-tech/rapidly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
