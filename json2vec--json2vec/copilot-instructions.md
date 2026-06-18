## json2vec

> Read `AGENTS.md` first. It defines the public JSON2Vec API style, schema patterns, gotchas, and verification commands.

# GitHub Copilot Instructions

Read `AGENTS.md` first. It defines the public JSON2Vec API style, schema patterns, gotchas, and verification commands.

Use `import json2vec as j2v` in generated examples. Prefer `Model.from_schema(...)`, built-in tensorfield constructors, `Array(...)`, and top-level package exports. Do not invent a public `Struct(...)` API.

Keep inline docs examples and notebooks runnable and small. When changing package behavior, update tests and docs together.

---
> Source: [json2vec/json2vec](https://github.com/json2vec/json2vec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
