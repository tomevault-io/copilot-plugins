## unit-test-coverage-95

> Unit test coverage ≥95% per Go package (binding rule)


# Unit test coverage ≥95% (binding)

> Canonical text lives in **`CLAUDE.md`**. This rule is the IDE-side surfacing; when this file disagrees with `CLAUDE.md`, `CLAUDE.md` wins.

## The rule

Every Go package in `packages/**` must hit **≥95% statement coverage** under `go test -cover -count=1 ./...`, OR be listed in `scripts/.coverage-allowlist` with a concrete category and rationale.

## Enforcement

- **Pre-commit hook** runs `scripts/check-go-coverage.sh --staged` on the Go packages with staged changes. Blocks the commit if any non-exempt package falls below 95%.
- **Full sweep** runs `npm run check:coverage` (also covered by `npm run check:all`). CI is wired to fail on threshold misses.
- **Strict-allowlist mode**: `scripts/check-go-coverage.sh --strict-allowlist` flags allowlisted packages that have since reached the threshold and can be removed from the file.

## Allowlist policy

Adding a package to `scripts/.coverage-allowlist` requires:

1. **Explicit user approval** in chat (echo the reason before committing).
2. **One of these categories** in the trailing comment:
   - **(A)** `cmd/*` entry point — only `main()` wiring, no logic.
   - **(B)** Test helper (e.g. `bufconn`, `testutil`, `idptest`, `storetest`).
   - **(C)** DB-bound — tests require live PostgreSQL.
   - **(D)** OS-bound — tests need kernel APIs / system keychain / packet capture.
   - **(E)** Network-infra-bound — tests need real S3 / NATS / Redis Sentinel.
   - **(F)** Integration-only — existing tests live behind build tags.

Packages that are *currently below threshold but on the open-source readiness roadmap* may be allowlisted under the "Open-source readiness backlog" section, with a follow-up reference. The long-term goal is an empty allowlist.

## Writing tests that count

Tests must assert **observable business behavior** and **named failure modes** — not just exercise code paths. Coverage padding (calling a function only to bump the percentage without assertions, or asserting only `err == nil`) defeats the rule's purpose.

The full **5-step audit methodology**, allowlist categories with examples, and the worked sweep example live in [`docs/developers/workflow/coverage-allowlist-methodology.md`](../../docs/developers/workflow/coverage-allowlist-methodology.md). Read it before writing a new test pass or proposing an allowlist entry.

## Carve-outs that are NOT allowed

- "Test would be slow." — slow tests still count. Move to `_integration_test.go` with build tag if needed; allowlist under category (F).
- "We'll add tests later." — adding to allowlist requires the category + a tracking reference. "Later" is not a category.
- "The code is too coupled." — refactor for testability. Tight coupling that prevents tests is itself a defect.

## Related rules

- **CLAUDE.md → Mandatory rules → Unit test coverage ≥95%** (canonical).
- **`completion-time-self-audit.mdc`** Q3: "every changed code path is exercised by a real test OR explicitly acknowledged as untested with a reason".
- **`sdd-workflow.mdc`** Step 6: unit tests required before reporting work done.

---
> Source: [AlphaBitCore/nexus-gateway](https://github.com/AlphaBitCore/nexus-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
