## 80-testing-quality

> Testing and code quality rules


Testing philosophy:
- Write production-quality code with tests for critical behavior.
- Prefer deterministic unit tests for business logic.
- Add integration tests for APIs, repositories, pipelines, and workflows crossing boundaries.
- Mock external services, cloud dependencies, and model providers where appropriate.

Testing expectations:
- Add tests for non-trivial service methods.
- Add tests for auth and permission-sensitive flows.
- Add tests for RAG/agent logic at the orchestration level where feasible.
- Cover validation, failure, and edge cases.
- Keep tests readable and isolated.

Code quality:
- Use linting and formatting.
- Use types wherever the stack supports them.
- Keep functions focused.
- Refactor repeated logic into shared utilities when justified.

Do not:
- Add major logic without at least basic tests.
- Create brittle tests tied to unstable implementation details.
- Depend on live external APIs in normal test flows.

---
> Source: [aiagentwithdhruv/laptop-finder-ai](https://github.com/aiagentwithdhruv/laptop-finder-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
