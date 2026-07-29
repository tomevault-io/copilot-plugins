## openclaw-command-center

> **DO NOT COMMIT** the following files — they contain user-specific data:

# public/data/ — User-Specific Data

**DO NOT COMMIT** the following files — they contain user-specific data:

| File                    | Purpose                          | Template                        |
| ----------------------- | -------------------------------- | ------------------------------- |
| `operators.json`        | User/operator info from sessions | `operators.json.example`        |
| `privacy-settings.json` | Hidden topics/sessions for demos | `privacy-settings.json.example` |

## Why?

These files are generated at runtime and contain:

- User IDs and usernames
- Session counts and activity
- Privacy preferences (what the user hides)

Committing them would leak user data to the public repo.

## For New Installations

Copy the `.example` files to get started:

```bash
cp operators.json.example operators.json
cp privacy-settings.json.example privacy-settings.json
```

The dashboard will populate these automatically on first run.

---
> Source: [jontsai/openclaw-command-center](https://github.com/jontsai/openclaw-command-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
