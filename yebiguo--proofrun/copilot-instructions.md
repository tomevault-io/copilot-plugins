## proofrun

> ProofRun is a local verification receipt for AI coding agents. It binds the result of a

# AGENTS.md

## What this project is

ProofRun is a local verification receipt for AI coding agents. It binds the result of a
command execution (exit code, duration) to the exact git state (HEAD commit + working-tree
diff fingerprint) at the moment it ran, and stores that as `receipt.json`. If the code
changes by even one byte, the receipt is immediately STALE.

This is a single-maintainer open-source CLI tool. It does not process any user-sensitive
data (no telemetry, no network calls, no accounts). It is built for public release.

Go, single-file binary. No server, no database, no LLM calls, ever, in this tool's core
logic — that is a product guarantee, not an implementation detail. See `.proofrun.yml` /
`.proofrun/receipt.json` for the on-disk formats.

## Product philosophy (read before changing status logic)

1. ProofRun never judges whether code is correct — only whether a check command was
   actually observed executing against the exact code that exists right now.
2. Status can only come from an observed execution, never from inference. There are
   exactly four statuses: `PASS`, `FAIL`, `STALE`, `NOT RUN`. Do not add a fifth
   (especially not an "INFERRED" or AI-guessed status) without an explicit product
   decision — this is the core trust guarantee of the whole tool.
3. When in doubt, report conservatively (`NOT RUN` / `STALE`) rather than optimistically
   (`PASS`). A false PASS is a severity-critical bug here, not a normal bug — it breaks
   the reason this tool exists.

## Collaboration tier

**Exploration tier.** You may work autonomously and push branches / open Draft PRs
without asking first. This is a young, low-stakes, pre-1.0 project.

## High-risk changes (slow down, be extra careful here)

- STALE determination logic (`internal/git`, the fingerprint comparison in
  `internal/receipt`) — this is the entire product. Needs strong test coverage,
  including boundary cases like whitespace-only diffs, line-ending differences, and
  untracked files.
- `receipt.json` schema (`internal/receipt`) — once released, schema changes must
  consider backward compatibility (old receipts should not crash new binaries).

## Before calling anything done

- `go test ./...` must pass.
- Cross-compile for the three release targets (windows/amd64, darwin/amd64 or arm64,
  linux/amd64) to confirm nothing platform-specific broke:
  `GOOS=windows GOARCH=amd64 go build ./...`, `GOOS=darwin GOARCH=arm64 go build ./...`,
  `GOOS=linux GOARCH=amd64 go build ./...`
- For anything touching STALE detection: manually verify by running a check, editing a
  tracked file, and confirming `proofrun status` reports STALE — don't rely on unit
  tests alone for this one.

## Explicitly out of scope for v0.1 (do not add without a product decision)

No INFERRED status, no parsing of test/build output content (exit code only), no
GitHub Action yet, no signing/encryption/OIDC, no web UI, no full coding agent, no
AI/LLM judging code correctness, no auto-fix, no MCP server, no telemetry, no database
or server component.

---
> Source: [yebiguo/ProofRun](https://github.com/yebiguo/ProofRun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
