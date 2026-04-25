## wishlist

> Whenever you make changes to the code, update the following as part of the same

# Instructions for Claude

## Keeping documentation in sync

Whenever you make changes to the code, update the following as part of the same
work — not as an afterthought:

### Inline comments (`src/`)
- Update or add comments in any file you modify.
- Comments should explain *why*, not just *what*. Focus on decisions that would
  not be obvious from reading the code alone (e.g. a specific SQL format chosen
  to match JavaScript's date output, a null-vs-undefined convention, a
  Client-Component boundary).
- Remove comments that no longer apply after your change.

### Docs (`docs/`)
Each file covers a specific topic. Update only the files affected by your change:

| File | Update when you change… |
|---|---|
| `docs/overview.md` | The app's purpose, tech stack, or how to run it |
| `docs/architecture.md` | The layer structure, data flow, or Server/Client Component split |
| `docs/database.md` | The schema, table relationships, or data storage decisions |
| `docs/auth.md` | The login flow, whitelist, NextAuth config, or session shape |
| `docs/api.md` | Any API endpoint (method, path, request body, response, errors) |
| `docs/privacy-model.md` | The visibility rules for claims or secret comments |
| `docs/testing.md` | The test strategy, helpers, or what a test file covers |
| `docs/style-guide.md` | Coding conventions adopted or changed in this project |

### README (`README.md`)
Update the README if you change anything that affects how a user interacts with
the app — features, login behaviour, ratings, or how to run it locally.

## Commit order

Include doc and comment updates in the same commit as the code change they
describe, unless the change is large enough that separating them makes the
history clearer. In that case, follow the code commit immediately with a docs
commit.

## What not to update

- Do not update docs for purely internal refactors that have no visible effect
  on behaviour, APIs, or the data model.
- Do not add comments to code you did not touch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arild-jacobsen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
