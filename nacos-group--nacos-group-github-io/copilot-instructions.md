## nacos-group-github-io

> Enforce single request-params table for API docs


# Nacos API Doc Request Parameters Structure

When updating API docs under `src/content/docs/next/**/manual/**/**/*-api.md`:

- Keep exactly one `#### 请求参数` section per API.
- If an API has both query and body parameters, merge them into one parameter table.
- Do not add a second `#### 请求参数` heading for body/query separation.
- In merged table descriptions, annotate parameter source when needed (for example: `Query 参数` / `Body 参数`), but keep one table only.
- Preserve existing manual descriptions and examples; only adjust fields required by swagger alignment.

---
> Source: [nacos-group/nacos-group.github.io](https://github.com/nacos-group/nacos-group.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
