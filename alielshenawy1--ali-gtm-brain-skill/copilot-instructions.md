## ali-gtm-brain-skill

> This repository packages two skills. It is not a go-to-market itself, it is the thing that builds

# GTM Brain, agent entry point

This repository packages two skills. It is not a go-to-market itself, it is the thing that builds
and maintains one. Codex, Cursor, Zed, opencode, Jules, and most newer agents read this file, so
it is the widest door in.

## If a project needs go-to-market work done

Read `skills/go-to-market-brain/SKILL.md` first and follow its procedure. It routes to three modes:

- **BUILD**, turn source material (dashboards, decks, strategy docs) into a governed context repository.
- **UPDATE**, take a signal from a lead, customer, or competitor, or a periodic snapshot, triage it, and route it to the right ledger.
- **OPERATE**, produce, check, and ship an asset against a brain that already exists.

Load the reference file that `SKILL.md` names for the mode you are in, at the point you need it,
rather than reading everything up front.

For copy rules alone, with no brain in play, read `skills/gtm-slop-filter/SKILL.md`.

## If you are being asked to install this

Run the installer with the flag for the tool you are running in:

```bash
curl -fsSL https://raw.githubusercontent.com/alielshenawy1/Ali-GTM-Brain-Skill/main/install.sh \
  | bash -s -- --codex          # or --cursor, --windsurf, --copilot, --gemini, --agents, --all
```

Add `--user` for every project on the machine, `--dir <path>` to place the skill files somewhere
other than `.claude/skills`, and `--dry-run` to list what would be written first.

No network access? Clone the repository and run `GTM_BRAIN_SRC=/path/to/clone ./install.sh` with
the same flags.

Cannot read files at all, or working with a model that cannot? Generate the paste-in version:

```bash
python3 skills/go-to-market-brain/scripts/bundle_skill.py --mode system-prompt
```

## If you are being asked to make this work somewhere else

Read `PORTING.md`. It carries the per-tool recipes (Codex, Cursor, Windsurf, Copilot, Gemini,
Grok, custom GPTs, Aider, Cline, Roo, Continue, Zed, agent frameworks, plain chat), the four ways
the skill can be loaded and how to choose, the pointer templates in
`skills/go-to-market-brain/adapters/skill/`, and a five-prompt check that proves the wiring
actually works. Run that check before reporting the job done.

Two rules when you write a pointer file. Substitute the real install path for `{{SKILL_PATH}}` and
`{{SKILLS_DIR}}`, and never copy strategy, claims, or rules into it. The pointer points. Content
that gets duplicated into a config file drifts, and a stale guardrail still carries authority.

## Repository layout

| Path | What it is |
|---|---|
| `skills/go-to-market-brain/` | The main skill: procedure, references, asset skeletons, adapters, scripts |
| `skills/gtm-slop-filter/` | Writing rules on their own, installable separately |
| `free/` | Six standalone tools, no install |
| `install.sh` | Installer and adapter writer |
| `PORTING.md` | How to run all of it in any agent |
| `docs/` | Landing page and brand assets |

---
> Source: [alielshenawy1/Ali-GTM-Brain-Skill](https://github.com/alielshenawy1/Ali-GTM-Brain-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
