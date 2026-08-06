## peakvox

> @docs/.agents/README.md

@docs/.agents/README.md
@docs/.agents/CONSTITUTION.md

---

## Superpowers Integration Rules

This Project uses Superpowers workflows:

- Brainstorming
- Specification
- Design
- Tasks
- Validation
- Subagent Execution
- TDD

However, This Project overrides the default Superpowers filesystem layout.

Agents must NEVER create:

docs/superpowers/specs/
docs/superpowers/plans/
docs/superpowers/tasks/

Instead, use:

Specifications

docs/.agents/SPECS/FEATURES/<feature-name>/

Files:

SPEC.md
DESIGN.md
TASKS.md
VALIDATION.md
STATUS.md

---

Implementation Plans

docs/.agents/IMPLEMENTATION/PLANS/

Examples:

peakvox-phase-1-foundations.md
peakvox-phase-2-model-registry.md

---

Execution Tasks

docs/.agents/IMPLEMENTATION/TASKS/

Examples:

task-voice-source-assets.md
task-provider-validation.md

---

Validation Reports

docs/.agents/VALIDATION/

Examples:

provider-validations/
retrospectives/
audits/

---

Current Superpowers Session

docs/.agents/SDD/

CURRENT_SPEC.md
CURRENT_DESIGN.md
CURRENT_TASKS.md
CURRENT_VALIDATION.md

---

# MANDATORY RULE

When using any Superpowers skill:

- brainstorm
- specification
- design
- planning
- task generation
- validation
- subagent execution

the generated artifacts must be written into the This Project documentation architecture.

Never use the default Superpowers folders.

The This Project documentation architecture supersedes the default Superpowers layout.

---

# SPEC GENERATION RULE

Whenever a new feature is created, the agent must generate:

docs/.agents/SPECS/FEATURES/<feature-name>/

SPEC.md
DESIGN.md
TASKS.md
VALIDATION.md
STATUS.md

before implementation begins.

Implementation without a SPEC is prohibited.

---

# IMPLEMENTATION RULE

Every implementation task must reference:

- Related ADRs
- Related Architecture Documents
- Related Spec

before coding begins.

---

# VALIDATION RULE

Every completed feature must generate:

VALIDATION.md

inside the feature folder.

Validation is considered part of implementation.

---

# STATUS RULE

Every feature folder must contain:

STATUS.md

Allowed statuses:

NOT_STARTED
PLANNED
APPROVED
IN_PROGRESS
PARTIAL
IMPLEMENTED
VALIDATED
SUPERSEDED
ARCHIVED

This becomes the feature-level implementation state.

---

# DEVELOPMENT WORKFLOW (HOT RELOAD)

Use this instead of rebuilding Docker images during development. The
edit → save → see-the-change loop requires **no image rebuilds**.

## Start development mode

```bash
scripts/start-dev.sh
```

What it does:

- **Backend + MinIO** run in Docker via `docker-compose.yml` +
  `docker-compose.dev.yml` (`--profile dev`). The override bind-mounts
  `backend/app/` into the container and runs uvicorn with `--reload`
  (watchfiles): saving any `.py` file under `backend/app/` reloads the
  API in a few seconds. All production mounts are preserved (shared
  `/data` volume, `docker.sock` for the DockerRuntimeDriver, read-only
  `runtime-registry/`), so the Runtime Registry lifecycle works
  identically in dev.
- **Frontend** runs on the host with `next dev` (http://localhost:3000)
  for native HMR — component changes appear in the browser on save.
  `NEXT_PUBLIC_API_URL` falls back to `http://localhost:8000`.
- **Ctrl-C** stops the frontend and brings the compose services down.
  Runtime containers (`peakvox-runtime-*`) are driver-managed, not
  compose services — start-dev never touches them.

## When a rebuild IS needed

- `backend/requirements.txt` or `backend/Dockerfile` changed:
  `scripts/start-dev.sh --build`
- `frontend/package.json` changed: `cd frontend && npm install`
  (the script auto-installs only when `node_modules/` is missing)
- Runtime server code (`runtime-registry/*/server.py`) changed: rebuild
  that runtime image via the Models page (Remove + Install) — runtime
  containers are intentionally immutable in dev.

## Production (unchanged)

```bash
docker compose --profile production up -d --build
```

The dev override file is opt-in only; production compose, Dockerfiles,
and deployment behavior are unaffected.

## Rule for agents

When iterating on `backend/app/` or `frontend/src/`, use development
mode. Do **not** run `docker compose build` per change — that is the
expensive path this workflow replaces.

---
> Source: [brunos3d/peakvox](https://github.com/brunos3d/peakvox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
