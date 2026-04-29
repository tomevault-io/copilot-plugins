## statechartx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run
```
go build ./...
go run examples/basic/main.go
go run examples/realtime/game_loop/main.go
go run cmd/scxml_downloader/main.go
```

## Test
```
make test              # All tests (short)
make test-race         # Race detector (MANDATORY for parallel states)
go test -v -run ^TestName$ ./...  # Single test
go test ./realtime/    # Package tests
make coverage          # Coverage report
make fuzz              # Fuzz tests
make bench             # Benchmarks
```

## Lint & Quality
```
make check             # format + vet + staticcheck + lint + test-race
make format            # gofmt + goimports
make lint              # revive
```

## Architecture
- **Core**: `statechart.go` implements hierarchical state machines (1.7k LOC). States have IDs, transitions (guarded/eventless), actions. `Runtime` handles event-driven macrosteps: LCA exits/enters, history (shallow/deep), final/done events, microstep loop (MAX_MICROSTEPS=100).
- **Parallel**: Regions run in goroutines (event-driven) with event targeting.
- **Realtime**: `realtime/` embeds core `Runtime` for tick-based deterministic execution (sequential parallel regions, event batching).
- **Tests**: SCXML conformance (`statechart_scxml_*_test.go`), parallel (`statechart_parallel_test.go`), stress/bench. Custom skill `scxml-translator` generates tests from W3C suite.
- **Key Patterns**: Numeric StateIDs, NO_EVENT transitions, internal event queue priority, thread-safe submits.

---
> Source: [comalice/statechartx](https://github.com/comalice/statechartx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
