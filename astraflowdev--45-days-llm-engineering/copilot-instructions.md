## 45-days-llm-engineering

> Guidance for Claude Code (and any AI assistant) working in this repository.

# CLAUDE.md

Guidance for Claude Code (and any AI assistant) working in this repository.

## What this repo is
Course code for **SSAI-101**, Softpro School of AI's **45-day AI Development Summer Training**
(LLM engineering + agentic AI). It is a *teaching* repo: code here is written to be **read and
learned from**, not just to run. Audience = students (pre-final/final-year B.Tech, BCA/MCA,
early-career) with only Python basics as a prerequisite.

- Full curriculum: [`COURSE-PLAN.md`](COURSE-PLAN.md)
- **Career track** (parallel 30-min/day "get hired" talks over slide decks): [`CAREER-PLAN.md`](CAREER-PLAN.md)
- Original syllabus: `docs/course-1-45day-summer-training.md (4).pdf` (text: `docs/_syllabus.txt`)
- Python-week source outline: `dump/python.txt`

## The Career Launchpad track (parallel to the code)
A separate **30-min daily talk** on getting hired / winning freelance work — thesis: **build content +
specialize**. Source of truth: [`CAREER-PLAN.md`](CAREER-PLAN.md) (45-day arc, 5 themes). Each day's
deck lives **inside that day's code folder** as `career-talk/index.html` (+ `README.md` speaker notes).
Decks are self-contained branded HTML (reuse the Day-1 career deck as the shared template); ~10–14
slides ≈ 30 min. Days 1–2 are committed; Days 3–7 are drafted locally. Keep `CAREER-PLAN.md` and the
per-day decks in sync.

**Structure:** 45 days · 3 hrs/day · 4 phases · 3 mini-projects + 1 capstone, all deployed free.

## Repository layout
```
45-Days-LLM-Engineering/
├── README.md                  Master roadmap
├── COURSE-PLAN.md             Full day-by-day plan (source of truth for what each day covers)
├── CLAUDE.md                  This file
├── requirements.txt           Grows as the AI track begins (Day 8+)
├── docs/                      Syllabus & reference
├── Phase-1-Foundations/       Days 1–15
├── Phase-2-RAG-and-Memory/    Days 16–25
├── Phase-3-Agents-and-Tools/  Days 26–37
└── Phase-4-Capstone/          Days 38–45
```

Days 1–7 are a Python power-up; the AI track starts Day 8. Phase day-ranges are fixed in
`COURSE-PLAN.md` — keep all READMEs consistent with it.

## Per-day structure (follow this pattern exactly)
```
Phase-X-Name/Day-NN-Topic-In-Kebab-Case/
├── README.md                  Learning objectives + module index table + how-to-run + today's exercise
├── 01-first-concept/
│   ├── README.md              Teaching notes: explain the idea, tables, gotchas, "➡ Next" link
│   └── concept.py             Runnable, heavily-commented example(s)
├── 02-next-concept/
│   └── ...
└── exercises/
    ├── README.md              Problem statements (link to solutions)
    ├── name.py                Starter/stub for students (with TODOs)
    └── name_solution.py       Worked solution
```

### Conventions
- **Day folders:** `Day-NN-Title-In-Kebab-Case` (zero-padded number, e.g. `Day-01-...`, `Day-12-...`).
- **Module folders:** `NN-concept-name` (zero-padded, kebab-case): `01-variables`, `02-functions`.
- **Python files:** `snake_case.py`. Solutions end in `_solution.py`; student stubs use `# TODO`.
- **Every `.py` is independently runnable** from its own folder: `python file.py`. No hidden setup.
- Each `.py` starts with a docstring saying what it shows and the exact run command.
- READMEs are GitHub-markdown: use tables for comparisons, fenced code blocks, and a
  `➡ Next: [..](../..)` link at the bottom of each module.

## Teaching style
- **Comment the "why," show the "what" by example.** Prefer many small, focused scripts over one
  large file.
- Introduce one concept per module. If a concept needs a forward reference (e.g. f-strings before
  Day 2), note it briefly and move on.
- Use realistic, India-context examples where natural (₹ amounts, local scenarios) — matches the
  cohort.
- Reuse the course's own exercises where they exist (Magic Trick, BMI Calculator, Rock-Paper-
  Scissors, Todo List, etc. — see `dump/python.txt`).
- **Free-tier first, multi-provider.** When the AI track starts: default to Gemini, also show
  Groq / Ollama / Hugging Face. Never write localhost-only or paid-API-only code.

## Running code (Windows environment)
- **Use the real CPython for running scripts and pip:**
  `C:\Users\PC\AppData\Local\Programs\Python\Python312\python.exe`
  ⚠️ The bare `python` on PATH is a hermes venv **without pip** — don't use it for installs.
- Verify new example scripts actually run before considering a day "done".
- Shell is PowerShell; the Bash tool is also available.

## When adding a new day
1. Read that day's row in `COURSE-PLAN.md` for scope.
2. Create `Day-NN-Topic/` with a day `README.md` (objectives + module table).
3. Break the topic into numbered modules, each with a teaching `README.md` + runnable `.py`.
4. Add an `exercises/` folder with at least one stub + solution.
5. Run every script to confirm it works.
6. Add any new dependencies to the root `requirements.txt`.
7. Keep cross-links and phase READMEs consistent.

## Status
- ✅ Repo scaffold, `README.md`, `COURSE-PLAN.md`, phase folders.
- ✅ **Day 1** complete and verified (8 modules + 3 exercises).
- ✅ **Day 2** complete and verified (8 modules + 3 exercises: Strings, f-strings & methods).
- ✅ **Career Launchpad** plan (`CAREER-PLAN.md`) + **Days 1–2 career decks** committed; Days 3–7 decks drafted locally (not yet committed).
- ✅ **Day 2 trainer's guide** (`TRAINERS-GUIDE.md`) — full minute-by-minute session script.
- ✅ **Day 3** complete and verified (7 modules + 3 exercises: Booleans, conditionals & logical operators) + trainer's guide.
- ⏳ Days 4–7 (Python *code*) next; then Day 8+ (AI track).
- ⏳ Career decks Days 8–45 to follow.
- Repo initialized (git).

---
> Source: [astraflowdev/45-Days-LLM-Engineering](https://github.com/astraflowdev/45-Days-LLM-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
