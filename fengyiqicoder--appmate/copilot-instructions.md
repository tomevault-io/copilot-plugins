## appmate

> AppMate is a read-only App Store Connect operations toolkit. The deterministic

# AppMate — agent context

AppMate is a read-only App Store Connect operations toolkit. The deterministic
data layer lives in `scripts/` (Python, App Store Connect API + iTunes Search).
The agent-facing logic lives in `skills/` — each skill is self-contained and
authoritative for its workflow. Re-read the relevant `skills/<name>/SKILL.md`
before running anything.

## Skills (7 workflows)

| Skill | What it does |
|---|---|
| `appmate-setup` | One-time credentials setup + self-check. Run this first. |
| `sales-daily-report` | Sales & downloads daily report across all live apps. |
| `aso-optimize` | Deep ASO rewrite (title / subtitle / keywords) for one app. |
| `aso-daily-report` | Keyword-rank daily monitor for top-3 apps. |
| `competitor-research` | Top 5-10 rivals outranking one app on its own keywords. |
| `feature-ideation` | Prioritized feature ideas from reviews + competitor evidence. |
| `growth-strategy` | Stage-diagnosed growth plan (cold start / early / plateau / decline). |

Invoke a skill the way your CLI expects:

- **Claude Code**: `/appmate-<workflow>` (e.g. `/appmate-sales`) — the slash
  command is a thin wrapper that calls the skill.
- **Codex CLI**: `$<skill-name>` (e.g. `$sales-daily-report`) or just describe
  the task in natural language — Codex matches against the skill's `description`
  frontmatter field.

Where a `SKILL.md` says "run `/appmate-setup`", treat it as "invoke the
`appmate-setup` skill" — on Codex that means `$appmate-setup` or asking for
setup in plain language.

## Hard safety rules (do not violate)

1. **Credentials gate.** Every workflow skill's Step 0 is
   `python3 scripts/appmate_config.py check`. If it exits non-zero, STOP and
   route the user to the `appmate-setup` skill. Do not work around it, do not
   fall back to mock data, do not skip it.
2. **Read-only API key.** AppMate is read-only by design. The API key the user
   configures must NOT carry write roles (Admin / Developer / App Manager /
   Finance). `scripts/appmate_config.py check` includes a role probe that
   refuses to start with a write-capable key.
3. **Write block in code.** `scripts/asc_client.py` rejects POST/PUT/PATCH/DELETE
   unless `APPMATE_ALLOW_WRITES=1` is set. **Never set that env var.** No
   workflow needs it; setting it defeats the safety design.
4. **No new write paths.** When editing or extending AppMate, do not introduce
   write calls to App Store Connect, do not add a flag that toggles the write
   block off, do not document `APPMATE_ALLOW_WRITES=1` as a fix for anything.

## Configuration

- Credentials live in `config/credentials.txt` (+ a `.p8` file). The folder is
  gitignored. Users can relocate `config/` and `data/` outside the plugin by
  setting `APPMATE_HOME` to a directory they control.
- Python deps: `pip install -r requirements.txt` (PyJWT, cryptography, requests).
  Tested on Python 3.10+.

## Output language

Reports are rendered in the language the user has been writing in this
conversation (English by default). App Store metadata strings stay in the
target-market locale regardless of the conversation language — only the
surrounding prose / headers / labels follow the user.

## Repository layout

```
.claude-plugin/   Claude Code plugin manifests
.codex-plugin/    Codex CLI plugin manifest
commands/         Claude Code slash command wrappers (one per workflow)
skills/           7 SKILL.md workflows — shared by both CLIs
scripts/          Python data layer + entry points
config/           gitignored — user credentials
data/             gitignored except for the two keyword_reference tables
docs/             ASC API reference + design specs
tests/            pytest suite
```

---
> Source: [fengyiqicoder/AppMate](https://github.com/fengyiqicoder/AppMate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
