## key

> Use when Codex needs to store, retrieve, check, delete, or import local-only API keys in macOS Keychain through the `key` CLI without exposing secret values.


# key

Use `key` for local-only credentials that scripts and agents need on this Mac.

## Rules

- Never print secret values in chat, logs, issues, PRs, or final responses.
- Prefer `key get <service> <account>` for local API-key reads.
- Prefer `pbpaste | key set <service> <account>` or `key import-op ...` for writes.
- Do not pass secret values as command-line arguments.

## Commands

```bash
key get <service> <account>
key set <service> <account>      # reads secret from stdin
key check <service> <account>
key delete <service> <account>
key import-op <service> <account> <op-ref>
key doctor
```

Use command substitution when injecting into a single process:

```bash
API_TOKEN="$(key get codex.service credential)" command
```

---
> Source: [xiaotianxt/key](https://github.com/xiaotianxt/key) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
