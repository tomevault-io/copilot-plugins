## repo-context

> Shared repository context entrypoint for yuanrong-datasystem


# Repository Context

Use the shared repository-local context under `.repo_context/`.

Read order:

1. `.repo_context/README.md`
2. `.repo_context/index.md`
3. `.repo_context/maintenance.md`
4. `.repo_context/generated/repo_index.md`
5. relevant `.repo_context/modules/<domain>/*.md` or `.repo_context/playbooks/<category>/...`

Rules:

- source code is the final source of truth;
- `.repo_context/` is an index and memory layer, not a substitute for reading code;
- if touched module structure or behavior changed, update `.repo_context/` in the same task when practical;
- if context is stale, correct it instead of ignoring the mismatch.

---
> Source: [openyuanrong-project/yuanrong-datasystem](https://github.com/openyuanrong-project/yuanrong-datasystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
