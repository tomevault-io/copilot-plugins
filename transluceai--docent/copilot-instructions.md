## backend

> Application services should accept a DB session and required services. This way, transaction management is handled across services.

# Sessions and services

Application services should accept a DB session and required services. This way, transaction management is handled across services.

```
class Service:
    def __init__(self, session: AsyncSession, ...services):
        self.session = session
        ...
```

In cases where services need to immediately commit results (e.g., computing searches), you can pass an additional writer_session_ctx factory:

```
class DiffService:
    def __init__(
        self,
        session: AsyncSession,
        writer_session_ctx: Callable[[], AsyncContextManager[AsyncSession]],
        ...services,
    ):
        self.session = session
        self.writer_session_ctx = writer_session_ctx
        ...
```

Other notes:
- Flushing and committing are the responsibility of the service *callers* (e.g., workers, HTTP handlers), not the service itself, except in rare cases.
- Methods in services should accept SQLAlchemy model instances as input, *not* IDs. The caller is responsible for pulling the instance from the database and validating its existence. When used together with dependency injection, this also reduces calls to the database.
- In services, make sure to distinguish variables pointing to SQLAlchemy model instances from Pydantic objects. Prefix variables pointing to SQLAlchemy model instances with `sq_`.

# Polling vs pushing

TODO(mengk)

---
> Source: [TransluceAI/docent](https://github.com/TransluceAI/docent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
