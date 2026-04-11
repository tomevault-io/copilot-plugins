## utils

> **Generated:** 2026-03-09

# LazyGophers Utils - Project Knowledge Base

**Generated:** 2026-03-09
**Scope:** repo root

## OVERVIEW
- Go 1.25 utility library with 20+ public root-level packages.
- Flat layout: no `cmd/`, `pkg/`, or `internal/`; packages live directly under repo root.
- Main themes: generics, performance helpers, multilingual data/validation, concurrency, cache algorithms.

## DOC BOUNDARIES
- Read this file first for repo-wide rules.
- Read [cache/AGENTS.md](cache/AGENTS.md) for cache algorithm selection and trade-offs.
- Read [atexit/AGENTS.md](atexit/AGENTS.md) for shutdown callback behavior and platform-specific exit handling.
- Read [xtime/AGENTS.md](xtime/AGENTS.md) for lunar calendar, solar term, and work-schedule time logic.
- Packages without a local `AGENTS.md` inherit this root guidance.

## REPO MAP
- Root files: `must.go`, `orm.go`, `validate.go`.
- Data + conversion: `candy/`, `json/`, `stringx/`, `anyx/`, `defaults/`.
- Time + schedules: `xtime/`.
- System + lifecycle: `runtime/`, `atexit/`, `app/`, `osx/`, `config/`.
- Reliability + concurrency: `cache/`, `routine/`, `wait/`, `hystrix/`, `singledo/`, `event/`.
- Security + network: `cryptox/`, `pgp/`, `network/`, `urlx/`.
- Test/support data: `fake/`, `randx/`, `pyroscope/`, `validator/`.

## WHERE TO LOOK
| Task | Location | Notes |
|---|---|---|
| fail-fast helpers | `must.go` | `Must`, `MustSuccess`, `MustOk` |
| DB JSON scan/value | `orm.go` | wraps JSON encode/decode for DB fields |
| root validation entry | `validate.go`, `validator/` | root helper delegates to validator package |
| cache policy choice | `cache/` | algorithm-heavy; use child doc |
| graceful shutdown | `atexit/` | build-tagged platform implementations |
| advanced time logic | `xtime/` | lunar calendar, solar terms, 007/955/996 subpackages |
| fake data generation | `fake/` | locale-specific embedded datasets and faker defaults |
| runtime helpers | `runtime/` | panic capture, stack dumping, executable/user dir helpers |
| i18n validation engine | `validator/` | locale files + custom validation engine |

## PROJECT CONVENTIONS
- Prefer existing root-level package structure; do not introduce deep hierarchy casually.
- Code is primarily English; docs should support English and Simplified Chinese.
- Many packages keep `llms.txt`; treat them as hints, not source of truth.
- Use generics where the package already does.
- Library code should return errors explicitly; only `Must*` helpers are expected to panic.
- This repo prefers `github.com/lazygophers/log` for logging.
- Tests are typically English-named and often split into normal tests plus `*_coverage_test.go` edge coverage files.

## PACKAGE-SPECIFIC PATTERNS
- `cache/`: multiple algorithms with different eviction semantics; avoid generic advice here.
- `fake/`: heavy locale/embed surface area; verify actual dataset loaders before changing behavior.
- `validator/`: custom engine + per-locale messages; check field-name strategy before editing messages.
- `runtime/`: includes OS-specific signal/path helpers; distinguish generic helpers from platform files.
- `atexit/`: callback execution semantics vary by OS family.
- `xtime/`: domain rules are calendar-specific, not generic date helpers.

## ANTI-PATTERNS
- Do not add child `AGENTS.md` files just because a package is large; add them only when local rules diverge materially.
- Do not trust generated or stale `llms.txt` content over source.
- Do not use `panic()` in ordinary library paths outside the `Must*` model.
- Do not ignore returned errors in non-Must code.
- Do not introduce circular dependencies between root packages.

## QUALITY CHECKS
```bash
make fmt
make lint
make test
make check
```

## NOTES
- Coverage badge in `README.md` currently shows overall coverage below the aspirational threshold; preserve existing test style instead of inventing new structure.
- `human/` is excluded from some coverage flows.
- Multi-platform behavior is real in this repo: build tags and cross-platform files are common enough to inspect before editing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lazygophers)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lazygophers)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
