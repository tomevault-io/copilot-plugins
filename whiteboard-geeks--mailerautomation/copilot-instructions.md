## mailerautomation

> description: Required services that must always be running for integration tests

---
description: Required services that must always be running for integration tests
globs: ["tests/**/*", "app.py", "celery_worker.py"]
alwaysApply: false
---

These services must be running for integration tests to work properly.

## 1. Redis Server (Terminal 1)

```bash
redis-server
```

- Required for all integration tests
- Default connection: `redis://localhost:6379/0`

## 2. Flask Application (Terminal 2)

```bash
source env/bin/activate & flask run --port=8080
# OR alternatively:
gunicorn app:flask_app --bind 0.0.0.0:8080
```

### Configuration
- Runs on port **8080** by default
- Tests expect `BASE_URL=http://localhost:8080`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whiteboard-geeks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
