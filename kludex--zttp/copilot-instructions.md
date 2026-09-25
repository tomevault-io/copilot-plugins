## zttp

> 1. When testing `pytest.raises(ValueError)`, always use the `match` parameter (`match=...`).

1. When testing `pytest.raises(ValueError)`, always use the `match` parameter (`match=...`).
2. Avoid small wrapper functions in tests, such as `make_client` and `make_server`.
3. Use `TypedDict` for parameters and dataclasses for output data.

# Pull Requests

- Include a concise `## Summary`.
- For relevant HTTP behavior changes, add `## Other HTTP Parsers` with a `Parser | Language | Behavior | Reference` table. Link exact evidence.
- Omit `## Other HTTP Parsers` when the comparison is not useful.
- Do not include a `## Tests` section.

---
> Source: [Kludex/zttp](https://github.com/Kludex/zttp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
