## kelvrix-enara-958341

> Agent for restyling scaffolded Android Jetpack Compose apps.

# CLAUDE.md

Agent for restyling scaffolded Android Jetpack Compose apps.

## Environment

- **Runtime:** Claude Code CLI headless via cron (VPS, autonomous)
- **Supplementary files (this repo):**
  - `generate-colors-dual.sh` — automated 2-color palette generation via Huemint API
  - `quality-check.sh` — pre-push validation
  - `RESTYLING.md` — visual restyling rules

## Input

All parameters are passed by the Python orchestrator (`run-claude-agent.py`), which handles Asana task fetching, naming generation, task updates, and scaffold execution **before** invoking this agent.

| Parameter    | Description                              | Example                          |
| ------------ | ---------------------------------------- | -------------------------------- |
| **SLUG**     | Project directory name under `/tmp/`     | `cool-app-1234567890`            |
| **APP_NAME** | Generated application name               | `Cool App`                       |

The scaffolded project is already available at `/tmp/<SLUG>` when this agent starts.

## Boundaries

- **No Publication:** Do not attempt to `git push`, `git remote`, or use `gh`. These remote operations are strictly managed by the Python orchestrator after Phase 2 is complete.
- **Local commits encouraged:** Use `git add -A && git commit -m "<description>"` after each meaningful change to preserve progress.
- **No MCP/External Tools:** Operate exclusively via standard CLI commands and local file modifications. Do not attempt to call any external APIs or MCP servers unless explicitly listed here.

---

**CRITICAL INSTRUCTION:** Execute both phases sequentially in a single pass without asking any questions.

## Phase 1 — Restyle the UI

**MANDATORY FIRST STEP:** Read `RESTYLING.md` in full before doing anything else.

1. Run the orientation commands from `RESTYLING.md` to map the UI layer.
2. Read every UI file fully before editing anything.
3. Make all decisions autonomously — choose colors, radii, spacing, text, and icon swaps without asking for approval.
4. Apply restyling categories according to the Priority table in `RESTYLING.md`: mandatory ones must have visible edits; highly desirable and required ones should be applied wherever the project has applicable elements.
5. Never change navigation routes, screen arguments, or ViewModel interfaces.
6. Never ask clarifying questions — if something is ambiguous, make a reasonable choice and proceed.

**MANDATORY FINAL STEP:** Before proceeding to Phase 2, verify that every ★ mandatory category has been applied. If any is missing — go back and complete it.

## Phase 2 — Quality Gate

```bash
bash quality-check.sh /tmp/<SLUG>
```

If ERRORS > 0 → fix each issue, re-run. Loop until ERRORS = 0.

**STOP HERE:** Once `quality-check.sh` reports 0 errors, your task is complete. Exit and let the orchestrator handle the final handover.

---
> Source: [ArtCode379/kelvrix-enara-958341](https://github.com/ArtCode379/kelvrix-enara-958341) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
