## cottage

> - **Command Safety**: AI agents are strictly forbidden from running `ctg` or `ctgx` commands under any circumstances.

# Workspace Instructions for AI agents

- **Command Safety**: AI agents are strictly forbidden from running `ctg` or `ctgx` commands under any circumstances.
- **Secret Files**: AI agents must never view, read, edit, create, or otherwise access:
  - Anything inside a `.cottage/` directory (identities, recipients, project state).
  - Any file matching `*.cott.*` (e.g. `*.cott.age` encrypted blobs, `*.cott.toml` redacted previews).
  - Any decrypted file `{file}` that has a corresponding `{file}.cott.age` encrypted counterpart on disk.

---
> Source: [sayanarijit/cottage](https://github.com/sayanarijit/cottage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
