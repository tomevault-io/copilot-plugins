## ggrun

> Treat these paths as a protected subsystem, not a convenient place for broad

# ggrun agent instructions

## Protected core engine

Treat these paths as a protected subsystem, not a convenient place for broad
cleanup:

- `go/pkg/placement/`
- `go/pkg/benchmark/`
- `go/cmd/ggrun/calibrate.go` and its tests
- exact launch/admission code in `go/cmd/ggrun/main.go`, `preflight*.go`, and
  `memory_probe_cmd.go`
- scheduler-facing progress polling in `go/cmd/ggrun/claude_progress*.go`
- optimizer-facing hardware evidence in `go/pkg/detect/`
- memory admission/recovery code in `go/pkg/recovery/` and `go/pkg/server/`

Unless the user's task explicitly requires a core-engine change, inspect these
paths read-only and implement elsewhere. Do not refactor, rename, simplify, or
"clean up" core behavior opportunistically.

Before an explicitly requested core change, read
`docs/core-engine-change-contract.md`, `docs/optimizer-theory.md`, and the
relevant entries in `docs/core-standard-launch-todos.md`. Preserve unrelated
working-tree edits. A predicted improvement is not promotion evidence.

Every core change must add or update an invariant-focused test and pass the
uncached core gate:

```sh
scripts/verify-core-engine.sh
```

Do not mark performance work complete from unit tests alone. Exact admission,
matched live agent-workload evidence, phase regression guards, and clean
relaunch remain required where the contract says they are required.

---
> Source: [raketenkater/ggrun](https://github.com/raketenkater/ggrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
