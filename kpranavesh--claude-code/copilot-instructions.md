## claude-code

> _Auto-loaded by Claude Code when working in this directory._

# CLAUDE.md — Krishna's Workspace

_Auto-loaded by Claude Code when working in this directory._
_Style guide → `memory/guidelines.md` | Personal/business context → `business-ideas/profile.md`_

---

## Who is Krishna

Product Manager, Analytics team (data science, insights, product strategy). New dad (son Eryk, 7 months). South Indian. Travels for work. Time is the constraint, not motivation or money.

**12-month goals:** side income stream · Substack newsletter · grow investment portfolio

Builds fast using Claude Code + Gemini. Can ship a working app in a weekend. Has done it.

---

## How to Respond

- **Direct opinion always.** No hedging. Lead with the recommendation.
- **Pushback:** "That's a bad idea because X." Say it once, then help execute.
- No emojis. Balanced length — not too short, not padded.
- Treat Krishna like a technical peer who moves fast. Don't over-explain.

Full guidelines → `memory/guidelines.md`

---

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- Write the plan to `tasks/todo.md` with checkable items before touching code
- If something goes sideways mid-task: STOP, re-plan, don't keep pushing
- Write detailed specs upfront — Krishna is analytical, ambiguity wastes both our time

### 2. Subagent Strategy
- Use subagents liberally to keep the main context window clean
- Offload research, exploration, and parallel analysis to subagents
- One task per subagent for focused execution
- For complex problems, throw more compute at it via subagents rather than grinding inline

### 3. Self-Improvement Loop
- After ANY correction from Krishna: update `tasks/lessons.md` with the pattern
- Write a rule that prevents the same mistake — not just a note, a rule
- Review `tasks/lessons.md` at the start of sessions involving code or analysis

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Run scripts, check output, demonstrate correctness — don't just assume
- Ask: "Would a senior PM/engineer approve this?" before presenting
- Diff behavior before and after changes when relevant

### 5. Demand Elegance
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a solution feels hacky, implement the elegant one instead
- Skip this for simple obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Point at logs/errors, then resolve.
- Zero context switching required from Krishna
- Find root causes. No temporary fixes. Senior developer standards.

---

## Task Management

For any multi-step task:

1. **Plan First** — write plan to `tasks/todo.md` with checkable items
2. **Verify Plan** — get sign-off before implementation on anything architectural
3. **Track Progress** — mark items complete as you go
4. **Explain Changes** — high-level summary at each meaningful step
5. **Document Results** — add review/outcome section to `tasks/todo.md` when done
6. **Capture Lessons** — update `tasks/lessons.md` after any correction or unexpected outcome

---

## Core Principles

- **Simplicity First** — make every change as simple as possible. Minimal code impact.
- **No Laziness** — find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact** — changes should only touch what's necessary. Avoid introducing bugs.

---

## Tech Stack

- **Language:** Python (primary)
- **AI framework:** LangChain + `langchain-anthropic`
- **Default model:** `claude-sonnet-4-6`
- **Fast/cheap subtasks:** `claude-haiku-4-5-20251001`
- **Heavy reasoning only:** `claude-opus-4-6`
- **Data:** pandas, numpy, yfinance
- **Secrets:** always via `python-dotenv` — never hardcoded
- **Budget:** $50–200/mo for tools/infra

---

## GitHub Push Rules

**NEVER push the following to GitHub — these are private:**
- `business-ideas/profile.md` — contains personal profile, all idea scores, constraints
- `business-ideas/business-plan/` — contains full business plans with competitive intel
- `business-ideas/reports/` — contains scouting reports and analysis
- `memory/` — contains personal context and preferences
- `portfolio/holdings.csv` — contains financial positions

**Safe to push:**
- `skills/*.md` — reusable skill templates (no sensitive content)
- `portfolio/analyzer.py`, `portfolio/deep_dive.py` — scripts only, no data
- `business-ideas/reddit_scout.py` — script only
- `CLAUDE.md` — workspace conventions
- `tasks/todo.md`, `tasks/lessons.md` — task tracking

When asked to "push everything" or "sync to GitHub", always check against this list first. When in doubt, don't push it.

---

## Code Conventions

- Type hints on all new functions
- `main()` entry point with a basic test run
- LCEL for new LangChain chains
- Modular: prompt / chain / tools / memory as separate concerns
- Edit existing files over creating new ones
- Don't touch code you didn't need to change (no unsolicited refactoring, no extra comments)

---

## Workspace Structure

```
Claude Code/
├── CLAUDE.md                          ← this file (auto-loaded)
├── memory/                            ← symlinked to ~/.claude/.../memory/
│   ├── MEMORY.md                      ← always-on context (< 200 lines)
│   ├── guidelines.md                  ← detailed response style guide
│   ├── changelog.md                   ← append-only update log
│   └── snapshots/                     ← monthly MEMORY.md snapshots
├── tasks/
│   ├── todo.md                        ← active task plan (checked off as done)
│   └── lessons.md                     ← self-improvement rules from corrections
├── skills/
│   ├── build-agent.md
│   ├── review-agent.md
│   ├── api-debug.md
│   ├── portfolio-analysis.md
│   └── business-scout.md
├── portfolio/
│   ├── holdings.csv                   ← all 31 positions (~$246K) [private, never push]
│   ├── analyzer.py                    ← python portfolio/analyzer.py --screen
│   └── deep_dive.py                   ← detailed metrics on specific tickers
└── business-ideas/
    ├── profile.md                     ← full personal + business profile [private, never push]
    ├── business-plan/                 ← business plans [private, never push]
    ├── reddit_scout.py                ← 4-agent Reddit + Claude idea scout
    └── reports/                       ← saved markdown reports [private, never push]
```

---

## Portfolio Context

~$246K · 31 positions · Pabrai-style concentrated conviction · Long-term bias (70% "never sell")

Top conviction: BTC, QQQM, VOO, GOOGL, TSM, AMZN, BRK.B
Feb 2026 adds: META + NVDA (add to existing) · LLY + AVGO + MA (new)

Run `python portfolio/analyzer.py --screen` for live data before any stock calls.

---

## Side Hustle Priorities

1. **Substack newsletter** — AI + investing + PM lens. Brand foundation.
2. **Micro-SaaS** — solo-buildable with Claude Code, real MRR within 12 months
3. **Privacy-first parenting apps** — proven gap (built baby food tracker himself)

Scout: `python business-ideas/reddit_scout.py --screen --save`

---

## Key Constraints

- Remote-first · Solo-buildable · Family-compatible (async, not 24/7 support)
- Time-scarce — evenings + weekends only. High-leverage or skip it.
- Budget: $50–200/mo until something makes money

---
> Source: [kpranavesh/Claude-Code](https://github.com/kpranavesh/Claude-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
