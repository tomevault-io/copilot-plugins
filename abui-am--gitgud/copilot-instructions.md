## database-interaction-best-practices

> description: Best practices when interacting with databases in backend Go code.

---
description: Best practices when interacting with databases in backend Go code.
globs: */db/**/*.go
---
When interacting with databases:
- Use prepared statements to prevent SQL injection.
- Handle database errors gracefully.
- Consider using an ORM for complex queries and data modeling.
- Close database connections when they are no longer needed.

- Use connection pooling to improve performance.

---
> Source: [abui-am/gitgud](https://github.com/abui-am/gitgud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
