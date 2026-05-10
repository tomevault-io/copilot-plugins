## data-modeling

> - **Pydantic BaseModel** - Use for all serializable data structures, API schemas, and persistence models


# Data Modeling and Architecture

- **Pydantic BaseModel** - Use for all serializable data structures, API schemas, and persistence models
- **@dataclass** - Use for lightweight, performance-critical structures (environment tiles, graph nodes)
- **Protocol** - Use for interfaces and abstract contracts
- **Field validation** - Use Pydantic `Field()` for constraints, descriptions, and metadata
- **Type unions** - Use `str | None` syntax for optional fields, not `Optional[str]`

---
> Source: [miniverse-ai/miniverse](https://github.com/miniverse-ai/miniverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
