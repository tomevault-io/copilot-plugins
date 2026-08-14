## oss-steward

> These rules are loaded by the pull-request review workflow. Coding agents and

# Agent instructions for oss-steward

These rules are loaded by the pull-request review workflow. Coding agents and
human reviewers should follow them.

## Rules

- Do not commit secrets, API keys, tokens, or private key material.
- Prefer parameterized queries; do not concatenate untrusted input into SQL.
- Do not use `eval`, `new Function`, or interpolated `child_process` calls.
- Do not disable TLS verification (`rejectUnauthorized: false`).
- Every change under `src/` should include or update a test under `tests/`.
- Keep workflow logic pure: no network calls inside `src/workflows/`.
- Fail closed on invalid model JSON; never auto-approve when parsing fails.
- Do not scan third-party repositories the applicant does not own.
- Record usage and adoption limits honestly in the readiness report. Do not
  invent stars, downloads, or OpenAI selection.
- Treat model output as untrusted input. Validate it against a schema before
  merge. Never let the model drive a shell command.

## Review focus

1. Correctness of maintainer workflow outputs (findings, labels, version notes).
2. Offline path remains complete when `OPENAI_API_KEY` is absent.
3. Criteria alignment scores real files and real function outputs.

---
> Source: [thangnq111203/oss-steward](https://github.com/thangnq111203/oss-steward) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
