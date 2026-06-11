## sqlmodel

> Satisfying the type checker when working with SQLModel

Complex SQLModel queries sometimes cause the type checker to choke, even though the queries are valid.

For instance, this error sometimes arises when using `selectinload`:

'error: Argument 1 to "selectinload" has incompatible type "SomeModel"; expected "Literal['*'] | QueryableAttribute[Any]"'

The solution is to explicitly coerce the argument to the appropriate SQLModel type.

E.g., we can resolve the error above by casting the eager-loaded relationship to InstrumentedAttribute:

```python
session.exec(select(SomeOtherModel).options(selectinload(cast(InstrumentedAttribute, SomeOtherModel.some_model))))
```

Similarly, sometimes we get type checker errors when using `delete` or comparison operators like `in_`:

'error: Item "int" of "Optional[int]" has no attribute "in_"'

These can be resolved by wrapping the column in `col` to let the type checker know these are column objects:

```python
session.exec(select(SomeModel).where(col(SomeModel.id).in_([1,2])))
```

---
> Source: [Promptly-Technologies-LLC/fastapi-jinja2-postgres-webapp](https://github.com/Promptly-Technologies-LLC/fastapi-jinja2-postgres-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
