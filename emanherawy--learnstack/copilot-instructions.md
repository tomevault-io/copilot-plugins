## learnstack

> learnstack turns Claude Code into a personalized learning mentor system.

# learnstack — development

## What this is

learnstack turns Claude Code into a personalized learning mentor system.
All slash commands read from `LEARNER.md` (your profile) and write to `study-notes/`.

## Skill reference

```
/profile            Build or update your learner profile → LEARNER.md
/orchestrate-learn  Start your daily session (reads history, continues or advances)
/daily              Set today's goal and session plan
/socratic-tutor     Main teaching block with Socratic questioning
/hands-on           Practice exercises from your confirmed resources
/memory-drill       Anki cards + mnemonics → study-notes/anki/YYYY-MM-DD.csv
/examiner           Gap finder (--session) or weekly review (--weekly)
/interview-sim      Interview simulation — scored Q&A from covered topics, tracks progress across runs
/feynman            Simplify a concept and rebuild it with citations
/unstuck            Diagnose why you're stuck and fix the root cause
/notes              Save full session notes → study-notes/sessions/YYYY-MM-DD.md
```

## Study notes structure

```
study-notes/
├── LEARNER.md                    ← learner profile (written by /profile)
├── PROGRESS.md                   ← running lesson log (updated every session)
├── INTERVIEW_PROGRESS.md         ← interview simulation score history
├── sessions/
│   └── YYYY-MM-DD.md             ← full session notes (written by /notes)
├── interview-sims/
│   └── YYYY-MM-DD-[type]-N.md    ← scored Q&A log per simulation run
└── anki/
    └── YYYY-MM-DD.csv            ← Anki import deck (written by /memory-drill)
```

## Hard rules (apply to every skill)

- Never teach without reading LEARNER.md first
- Never state a fact without citing the confirmed resource (book, doc URL, repo file + line)
- If unsure → say "verify in [resource]", never guess
- Always save output — nothing learned is lost

---
> Source: [EmanHerawy/learnStack](https://github.com/EmanHerawy/learnStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
