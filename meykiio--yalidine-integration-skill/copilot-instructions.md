## yalidine-integration-skill

> This repository holds one thing: the **`yalidine-integration/`** Agent

# Agent instructions for this repo

This repository holds one thing: the **`yalidine-integration/`** Agent
Skill — a `SKILL.md` package for integrating the Yalidine (Guepex)
Algerian delivery API into a codebase (parcel creation, tracking,
wilaya/commune/stop-desk lookups, delivery fees, and signed webhooks).

## If you're an agent working in this repo

- **Before writing any code, editing docs, or answering a question about
  Yalidine/Guepex integration**, read
  [`yalidine-integration/SKILL.md`](yalidine-integration/SKILL.md) first.
  It loads its own reference files (`references/*.md`) as needed — don't
  read all of them up front, just the ones the current task calls for.
- **Never invent or fabricate an API ID, API token, webhook secret, or
  account-specific URL.** These only exist in the user's own Guepex
  dashboard. If a task needs one and it isn't in an env var, stop and ask
  — see `yalidine-integration/references/human-only-steps.md` for exactly
  what the user has to do themselves.
- **The API token is backend-only.** Never place it in client-side/browser
  code, regardless of what file you're editing.
- If you're touching `yalidine-integration/` itself (fixing a stale field,
  adding a framework example, etc.), see
  [`CONTRIBUTING.md`](CONTRIBUTING.md) — the same "no real credentials,
  ever" rule applies to example code you write there too.

## Repo layout

```
README.md                    — human-facing overview (this file is for agents; that one's for people)
yalidine-integration/        — the actual skill; copy/zip *this folder* into your agent's skills directory
├── SKILL.md
├── references/
├── scripts/
└── assets/
```

---
> Source: [Meykiio/yalidine-integration-skill](https://github.com/Meykiio/yalidine-integration-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
