## invariants

> Non-negotiable invariants — verification, filesystem safety, output correctness


# zapstore-cli — Invariants

## Verification

- A binary MUST NEVER be installed unless its SHA-256 hash matches the `x` tag in the signed asset event (3063).
- Hash verification must happen after download, before any filesystem placement.
- Signed Nostr events must be signature-verified before their content is trusted.

## Version Comparison

- Update availability MUST be determined solely by comparing `version_code` integers.
- Never compare version name strings (semver, etc.) for update logic.
- If either the installed or available `version_code` is missing, do not consider it an update.

## Filesystem Safety

- Partial downloads must be written to a temp path and moved atomically on success.
- A failed or interrupted install must never leave a partial binary in the packages directory.
- Symlinks must point to the correct versioned binary after every install/update.

## Output

- Status messages (progress, errors) go to stderr.
- Data output (`list --json`, `search --json`) goes to stdout.
- `--quiet` mode suppresses all stderr output except errors.
- Exit 0 = success. Exit 1 = error.

## Context Cancellation

- Downloads and relay queries must respect `context.Context`.
- Ctrl+C must cancel in-flight operations cleanly — no partial installs left behind.

## Platform Filtering

- Only assets matching the current OS and architecture must be considered.
- If no matching asset exists, fail with a clear "no compatible asset" error.

---
> Source: [zapstore/zapstore-cli](https://github.com/zapstore/zapstore-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
