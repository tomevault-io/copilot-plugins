## spark-stack

> This is a React Next.js project with a FastAPI Python backend.

This is a React Next.js project with a FastAPI Python backend.

Frontend (/frontend):

- Use .js for JSX files
- Use 'next/navigation' for all navigation/routing
- Use /frontend/src/lib/api.js for all API calls
- Handlers (onClick, onSubmit, etc.) should be named handle{action} and explicitly defined `const handle{action} = (e) => { ... }`

Backend (/backend):

- Use pydantic for all data models
- Use /backend/db/models.py for all database models
  - Use alembic for generating migrations. You may modify the migration after it is created.
- Use backend/schemas/models.py for all routes input/output pydantic models
- Use backend/routers/{table}.py for all routes for that table/model
- Must use type hints

Scripts (/scripts):

- Use argparse for all scripts
- Names should start with verbs like "add", "remove", "update", "list", etc.

---
> Source: [sshh12/spark-stack](https://github.com/sshh12/spark-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
