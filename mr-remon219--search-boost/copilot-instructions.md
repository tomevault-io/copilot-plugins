## commit-authorship

> description: Git commits must be authored by the user only (no Cursor co-author)

﻿---
description: Git commits must be authored by the user only (no Cursor co-author)
alwaysApply: true
---

# Commit authorship

When committing in this repo:

- Author and committer must be the human maintainer (Mr-remon219 / lemonkun211@gmail.com), not Cursor Agent or `cursoragent@cursor.com`.
- Never add `Co-authored-by:` lines for Cursor, Cursor Agent, or any AI tool.
- Do not use inverted co-author trailers; the user is the sole author even when an agent wrote the patch.

Prefer `git commit` with the user's configured identity and a message only; no agent attribution footers.

---
> Source: [Mr-remon219/search-boost](https://github.com/Mr-remon219/search-boost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
