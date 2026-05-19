## vikash-wms

> **This file is NOT the primary context file.** It was created during a session but is redundant.

# CLAUDE.md — REDIRECT + CRITICAL RULES

**This file is NOT the primary context file.** It was created during a session but is redundant.

**Read these three files instead (on OneDrive, in the parent `WMS Claude/` folder):**
1. `WMS-CONTEXT.md` — architecture, files, functions, UI standards, NEXT STEPS / TODO, recently resolved archive, and the copyable **Startup Prompt** at the end of the file.
2. `WMS-SCHEMA.md` — DB tables, migrations, investor vs trader audit.
3. `WMS-LESSONS.md` — rules, patterns, settled decisions (hierarchical A.1.1 format).

Module-level canonical docs (e.g. the Statements engine formulas) live inside `WMS-CONTEXT.md` under the relevant sub-module section — no separate per-module `.md` files.

---

## ⛔ CRITICAL: DEV/PROD GIT WORKFLOW (MANDATORY — READ EVERY SESSION)

**Two branches exist. Two live sites. One database. Violating these rules can break production.**

| | Production | Dev |
|---|---|---|
| Branch | `main` | `dev` |
| Site | vikashbagla.github.io/Vikash-WMS/ | vikash-wms.pages.dev |
| Data tables | `transactions`, `ledger_entries` | `transactions_dev`, `ledger_entries_dev` |
| Master data | Shared (investors, brokers, securities, IBAs, etc.) | Shared (same tables) |

### HARD RULES:

1. **ALL new code goes to the `dev` branch. NEVER push directly to `main`.** No exceptions. Not for "small fixes," not for "quick patches," not for "just this one thing."
2. **Code reaches `main` ONLY via a merge from `dev`, and ONLY after the owner explicitly approves.** Ask "Ready to push this to production?" and wait for a clear yes.
3. **When committing, always verify which branch you are pushing to.** Use `git push origin dev` — never `git push origin main` unless the owner has approved a production deploy in the current conversation.
4. **If the local repo is on `main` (lock file issues, fresh clone, etc.), push to dev explicitly:** `git push origin main:dev` does NOT work for this — you must push committed changes to `origin dev`. Use a temp clone on the `dev` branch if needed.
5. **Test on the dev site (vikash-wms.pages.dev) before any merge to main.**
6. **Database migrations:** Test on dev tables first. Only run against production tables after owner approval.
7. **Master data is shared.** Changes to investors, brokers, securities, IBAs, charges, portfolio_views affect BOTH sites. This is by design — but be aware of it.

### IF IN DOUBT:
- Push to `dev`. Always.
- Ask the owner before touching `main`.

See `WMS-LESSONS.md §A.8` for the full specification and `§B.20` for technical details.

---

**⚠️ NAMESPACE RULES:** The Trading → Statements module uses `lg` JS prefix and `ledger_*` DB tables. The Accounting module (when built) MUST use `ac` prefix and separate table names. See `WMS-LESSONS.md §A.6` for the full boundary specification — read this BEFORE writing any Accounting code.

**⚠️ EXPORT RULES:** All Excel exports MUST use linked formulas by default (not hardcoded values). Use the global `wmsExportExcel()` engine in `wms-export.js` — never build ad-hoc export code. See `WMS-LESSONS.md §A.7` for the full specification.

---
> Source: [vikashbagla/Vikash-WMS](https://github.com/vikashbagla/Vikash-WMS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
