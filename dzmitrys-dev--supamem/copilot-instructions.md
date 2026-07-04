## dual-memory

> Use both semantic (qdrant-find) and structural (graphify) memory before non-trivial code changes


# Dual Memory — supamem

Use **both** memory layers before any non-trivial code change.

## When to query

- New feature, refactor, debugging, or architecture decision
- Adding/modifying dependencies
- Code review of a non-trivial change

## Semantic (Qdrant tuned_hybrid)

```
qdrant-find "<topic> decisions"
qdrant-find "<topic> known issues"
qdrant-find "<topic> patterns"
```

## Refresh

If memory feels stale: `supamem index --target tuned --force`.

## Anti-patterns

- ❌ Querying after coding — check memory before implementing.
- ❌ Skipping semantic for "what calls X" — use structural for that.
- ❌ Trusting hits without checking source attribution — always inspect the
  payload's `source` field.

---
> Source: [dzmitrys-dev/supamem](https://github.com/dzmitrys-dev/supamem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
