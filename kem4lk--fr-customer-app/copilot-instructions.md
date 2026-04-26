## fr-customer-app

> General Info for API Layer

# API Layer

The API layer is organized under [app/api/](mdc:app/api) and built with FastAPI. Key features:
- Endpoints are modularized in [app/api/v1/endpoints/](mdc:app/api/v1/endpoints).
- Schema definitions are in [app/api/v1/schemas/](mdc:app/api/v1/schemas).
- The API validates incoming requests, persists them to PostgreSQL, and delegates background processing to Celery.
- Dependencies and helpers are found in files like [dependencies.py](mdc:app/api/dependencies.py) and [middleware.py](mdc:app/api/middleware.py).

The API layer responds quickly and delegates heavy work to the background for scalability and reliability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kem4lk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
