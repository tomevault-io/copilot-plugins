## voice-user

> 1. Check `.session-reports/` for the most recent report


## Session Management

### On Session Start
1. Check `.session-reports/` for the most recent report
2. If it contains **CLAUDE.md Suggestions**: briefly tell the user (e.g., "Last session has 2 CLAUDE.md suggestions — want to review?")
3. If it contains **Open Items**: briefly mention them
4. Do NOT read the full report aloud unless asked — just flag what needs attention

### Before Ending a Session
Before finishing any substantive session (i.e., you wrote code, made plans, or made decisions), you MUST:
1. Run the `/session-report` command (or generate the report manually following that command's format)
2. Review `docs/plans/` and `docs/research/` for stale documents — add stale paths to `.archive-queue`
3. Confirm the report was written before signing off

If the session was trivial (quick question, no code changes), skip the report.

### File Organization
- **`docs/plans/`** — Active plans and design documents only. Name files descriptively: `feature-name-plan.md`
- **`docs/research/`** — Feature research, organized by topic subfolder: `docs/research/auth-providers/`
- **`docs/screenshots/`** — Debug screenshots from headless dev. Auto-archived after 14 days.
- **`.session-reports/`** — Session reports. Git-tracked. Do not delete.
- **`.project-archive/`** — Archived stale documents. Git-excluded.

### Archive Policy
- Files in `.project-archive/` preserve their original directory structure
- Do NOT reference archived files unless the user explicitly says to
- To archive a file mid-session: add its relative path to `.archive-queue` (processed by the Stop hook)
- When in doubt, archive rather than delete — nothing is lost

### CLAUDE.md Updates
- NEVER auto-update CLAUDE.md. Always propose changes in session reports under "CLAUDE.md Suggestions"
- Format suggestions as concrete diffs so the user can review and apply
- On session start, if pending suggestions exist, flag them for review

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akameswaran) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
