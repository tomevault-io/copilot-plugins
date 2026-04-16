## pg-vscode-extenson-project-repo

> Before you analyze the user's request, write code, or answer with a plan, you must complete this startup sequence:

# Gemini Repo Instructions

## CRITICAL STARTUP OVERRIDE
Before you analyze the user's request, write code, or answer with a plan, you must complete this startup sequence:
0. Run `.\pg.ps1 start -Yes`
1. Read `Memory-bank/daily/LATEST.md` and the latest daily report.
2. Read `Memory-bank/project-spec.md`.
3. Read `Memory-bank/project-details.md`.
4. Read `Memory-bank/structure-and-db.md`.
5. Read latest entries in `Memory-bank/agentsGlobal-memory.md`.
6. Read `Memory-bank/tools-and-commands.md`.
7. Read `Memory-bank/coding-security-standards.md`.
8. Check `Memory-bank/mastermind.md` for open decisions.
9. If startup fails, stop and report the failure before doing any other work.

Do not defer startup until after the first answer.

## Primary Policy
Use `AGENTS.md` as the primary policy contract for this repository.

## End Of Session
Mandatory end-of-session when code changed:
1. Update relevant Memory-bank docs (`structure-and-db`, `db-schema`, `code-tree`).
2. Append `Memory-bank/agentsGlobal-memory.md`.
3. Update `Memory-bank/daily/YYYY-MM-DD.md`.
4. Update `Memory-bank/daily/LATEST.md`.

## Rules
- Never add secrets to Memory-bank or code.
- If SQL migrations change, update `Memory-bank/db-schema/*.md` in the same session.
- Respect local hook and CI Memory-bank guards.
- Explainer: `AI_ENFORCEMENT_GUIDE.md`
- Optional workflow helper: `.agents/workflows/startup.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/figchamdemb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
