## claude-context-meter

> Instructions for AI assistants (Claude, Gemini) working on this repo.

# CLAUDE.md — Claude Context Meter

Instructions for AI assistants (Claude, Gemini) working on this repo.

## Security Gate (every change)

Run `./scripts/scan.sh` before every commit. All three tools must pass:
- Gitleaks — secret detection
- Semgrep — static analysis
- Trivy — dependency vulnerability scan

The pre-commit hook runs Gitleaks automatically as a backstop.

## CI Pipeline Order

Gitleaks runs first. Semgrep, Trivy, and SwiftLint will not start until Gitleaks passes. No tokens or keys may ever reach the repo.

## PR Review Checklist (CSSLP Audit)

When reviewing a pull request, act as a CSSLP auditor and check all of the following:

1. **File I/O paths** — The app reads `~/.claude/projects/` (intentionally unsandboxed). Flag any writes to paths outside the user's home directory or any hardcoded absolute paths.
2. **No secrets in logs** — Reject any code that logs billing amounts, token counts, or file contents to syslog, os_log, or print statements at non-debug levels.
3. **Billing window edge cases** — Verify that the 5-hour window anchor logic handles: system clock changes, power sleep/wake cycles, daylight saving transitions, and leap seconds.
4. **Integer safety** — Check token math for integer overflow. Token counts can reach tens of millions; use Int64 or guard against overflow.
5. **Non-standard crypto** — Flag any custom hashing, encoding, or encryption. Use system APIs only.

## Hardened Runtime (Xcode 26.4)

Hardened Runtime is required for Apple notarization and is independent of App Sandbox.

**Status: already enabled.** Hardened Runtime is active in Signing & Capabilities with all exceptions unchecked — correct for this app.

The section has two sub-groups:
- **Runtime Exceptions** — relaxations for JIT, unsigned memory, library validation (leave all off for this app)
- **Resource Access** — TCC entitlements for camera, mic, location, etc. (leave all off for this app)

This app reads files from `~/.claude/` via Foundation APIs (String, FileManager) and FSEvents. No Hardened Runtime entitlements are needed for file system access — those restrictions belong to App Sandbox, not Hardened Runtime. Enable the capability as-is with no boxes checked.

**Note:** Xcode 26 also introduces a separate "Enhanced Security" capability (new in 2026, opt-in, adds pointer authentication and memory integrity enforcement). It is not the same as Hardened Runtime and is not required for this app.

## Architecture Notes

- No App Sandbox (required to read `~/.claude/projects/` — sandboxed apps cannot access arbitrary home directory paths)
- No network calls, no API keys, no remote data
- Reads JSONL files via FSEvents + 30s heartbeat timer fallback
- Deduplicates by `requestId`; sums output_tokens from all complete assistant records
- Bundle ID: `com.scottbly.ClaudeContextMeter`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scottblydotcom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
