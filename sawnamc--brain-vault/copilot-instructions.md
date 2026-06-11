## brain-vault

> Read this at the start of every session. Project-specific context lives in each project's own `CLAUDE.md` — this file is the vault-level index only.

# Sawna's Second Brain — Claude Context File

Read this at the start of every session. Project-specific context lives in each project's own `CLAUDE.md` — this file is the vault-level index only.

---

## Who This Vault Belongs To

**Sawna Caradwyn.** 50 years old. Based in Holyoke / Western Massachusetts. Background in operations, HR, and accounting (15+ years). Currently building two businesses (Sage & Signal + Pebble's Path) while consulting for The Performance Project. This is a season of reinvention.

- **Partner:** Terna — romantic partner; co-creating PolyFool and Adventures of Owl & Mouse; plan to move in together once finances stabilize.
- **Son:** Nehemiah — grown; relationship naturally evolving.
- **Spiritual teacher:** William Douglas Horden — 15 years of study; *The Toltec I Ching*, *The Divine Dark* (2020), *The Soul of Numbers* (2025). Freely quotable across all projects.
- **Main goal:** $10K/month in revenue by May 2027. See [[GOALS.md]] for full plan.
- **Personality:** Strong starter, weak finisher. Binary motivation (on = unstoppable; off = stuck). ADHD-like scatter. Hates repetition. Can't self-regulate when hyperfocused.

**The governing principle:** This vault exists to produce things, not just store things.

---

## Weekly Update

See [[🤖 Weekly Brief.md]] for current status, what's working, and time-sensitive items.

---

## Note Types (Front Matter: `type:`)

| Type | What It Is |
|---|---|
| `note` | Raw captures from external content (videos, books, pods) |
| `idea` | Something applicable to a current project RIGHT NOW |
| `solution` | Sawna's worked answer to a specific problem |
| `problems` | Project overview — goal, why, outcomes, open problems. Read first. |
| `daily` | Daily log — what was worked on, what's still unsolved |

---

## Active Projects

**Read each project's `CLAUDE.md` for full context, key people, rules, and current status.**

| Project | Location |
|---|---|
| **The Performance Project** (PRIMARY) | `02 Projects/Sage and Signal/01 Active Engagements/The Performance Project/` |
| **Sage & Signal** | `02 Projects/Sage and Signal/` |
| **David** | `02 Projects/Sage and Signal/01 Active Engagements/David/` |
| **PolyFool** | `02 Projects/PolyFool/` |
| **Pebble's Path** | `02 Projects/Pebble's Path/` |
| **Cangioka** | `02 Projects/Cangioka/` |
| **ISH** | `02 Projects/Interfaith Spirit House - ISH/` |
| **Job Search 2026** | `02 Projects/Job Search 2026/` |
| **Cian** | `02 Projects/Sage and Signal/01 Active Engagements/Cian/` |
| **Adventures of Owl & Mouse** | `02 Projects/Adventures of Owl & Mouse/` |

---

## Emerging Ideas

| Idea | Notes |
|---|---|
| **Umbrella LLC** | Hold both Pebble's Path + Sage & Signal. Not yet decided. |
| **Workshop Series with Terna** | Spiritual health of communities. Very early, no structure. |
| **Amazon Marketplace** | On hold. Seasonal arbitrage ~$1K budget. Not pursuing now. |

---

## Folder Structure

```
Brain/
├── CLAUDE.md              ← this file
├── 🤖 Weekly Brief.md   ← current status, what's working, time-sensitive items
├── 🤖 Decisions.md       ← append-only decision log (written by `level-up` + by hand)
├── 🤖 Expansions.md      ← structure guide — read before adding new vault folders/files
├── 🤖 Wiki Log.md        ← append-only log of sources ingested via `ingest` skill
├── 🤖 Auto Memory/       ← symlink → ~/.claude/projects/.../memory/ (Claude Code auto-memory; Layer 2)
├── GOALS.md               ← revenue model, plan, targets, risks
├── MEMORY.md              ← persistent cross-session facts (manual, Layer 1)
├── Musings.md
├── Clippings/             ← web clippings (Obsidian Clipper)
│
├── 00 Notes/          ← daily logs + session captures + weekly/monthly/quarterly/yearly reviews
│   ├── YYYY-MM-DD.md                    ← Sawna's logs
│   ├── 🤖 YYYY-MM-DD.md               ← Claude's session logs
│   ├── Archive/                         ← old daily logs + superseded weekly reviews
│   ├── Raw/
│   ├── Sage and Signal/
│   └── Pebble's Path — Digital Products/
│
├── 01 Reference Materials/  ← Books, frameworks, reusable reference docs (I Ching, Tao Te Ching, etc.)
│   ├── AI Learnings/
│   ├── IChing/
│   ├── Nigel Richmond/
│   ├── Sawna's writing samples/         ← 6 writing samples; style analyzed 2026-05-17
│   ├── Tao Te Ching/
│   ├── William Douglas Horden/          ← Toltec I Ching reference materials
│   ├── William Douglas Horden.md        ← root-level overview note
│   └── 🤖 Human Design - About Sawna.md ← Human Design synthesis
│
├── 02 Projects/
│   ├── 🤖 All Projects.md              ← cross-project overview
│   ├── 🤖 Tasks.md                     ← cross-project task roll-up
│   ├── Cangioka/                        ← CLAUDE.md inside
│   │   ├── 00 Content Pipeline/
│   │   ├── 01 Published/
│   │   ├── 02 Brand/
│   │   ├── 03 Automation/
│   │   └── Content/
│   ├── Interfaith Spirit House - ISH/   ← CLAUDE.md inside
│   │   ├── 00 Meetings/
│   │   ├── 01 Vision & Founding Docs/
│   │   ├── 02 Research/
│   │   └── 03 Events/
│   ├── Job Search 2026/                 ← CLAUDE.md inside
│   │   └── archive/
│   ├── Pebble's Path/                   ← CLAUDE.md inside
│   │   ├── 00 Notes and Reference Materials/
│   │   ├── 01 Daily Logs/
│   │   ├── App/
│   │   ├── Book/
│   │   ├── Digital Products/
│   │   ├── Social Media & Website/
│   │   └── YouTube Channel/
│   ├── PolyFool/                        ← CLAUDE.md inside
│   │   ├── 00 Story Room/
│   │   ├── 01 Scripts/
│   │   ├── 02 Series Bible/
│   │   ├── 03 Reference/
│   │   ├── 04 Brand/
│   │   └── 05 Notes & Feedback/
│   ├── Sage and Signal/                 ← CLAUDE.md inside
│   │   ├── 00 Notes and Reference/      ← 🤖 LLM Foundations for Client Work (7-source S&S LLM compile layer, built 2026-05-27)
│   │   │   └── C Logs/                 ← client engagement logs
│   │   ├── 01 Active Engagements/
│   │   │   ├── Cian/                    ← 🤖 Tasks.md = live tracker; on hold (no-show 5/27)
│   │   │   ├── David/                   ← CLAUDE.md inside; 🤖 Tasks.md = live tracker
│   │   │   │   └── Meeting Notes/
│   │   │   └── The Performance Project/ ← CLAUDE.md inside; 🤖 Tasks.md = primary tracker
│   │   │       ├── 00 Notes/
│   │   │       ├── 01 Daily Logs/       ← 🤖 Logs/ subfolder for Claude daily logs
│   │   │       ├── 01 Systems Revamp/   ← 🤖 Migration Plan - FY26 Transition.md
│   │   │       ├── 02 Bookkeeping/      ← Banking/, Budgeting/, Grants/, Invoices/, Payroll/
│   │   │       ├── 03 SOPs/
│   │   │       ├── 04 Reference/
│   │   │       └── memory/              ← TPP-specific key contacts and facts
│   │   ├── 01 Daily Logs/               ← S&S-level daily logs
│   │   ├── 02 Pipeline/                 ← discovery tracker, prospect briefs
│   │   ├── 03 Brand/
│   │   ├── 04 Content/
│   │   ├── 05 System/
│   │   ├── 06 Skills/
│   │   ├── 07 Attachments/              ← AIS-OS, KJ OS Template, Second Brain Starter, cowork-starter-pack + source docs
│   │   └── 08 Iteration Logs/
│   ├── Adventures of Owl & Mouse/           ← CLAUDE.md inside; webcomic; Owl (Sage) + Mouse (Fool)
   │   ├── 00 Concept & Characters/
   │   ├── 01 Scripts/
   │   ├── 02 Art & Assets/
   │   ├── 03 Published/
   │   ├── 04 Reference/
   │   ├── 05 System/
   │   ├── 06 Skills/
   │   ├── 07 Attachments/
   │   └── 08 Iteration Logs/
   └── (PROJECT TEMPLATE)/
│
└── 03 Skills/                           ← skill definition files (loaded by Claude Code)
    ├── brain-setup.md
    ├── end-of-day.md
    ├── good-morning.md
    ├── ingest.md
    ├── level-up.md
    ├── new-project.md
    ├── progress-report.md
    ├── session-log.md
    ├── update-tasks.md
    ├── weekly-update.md
    └── youth-stipend-invoice.md
```

---

## What Claude Should Do

1. **Read the project's `CLAUDE.md` first** — it has context, people, rules, and current status
2. **Read `🤖 Tasks.md`** for any client project before doing task work
3. **Read front matter** — it tells you note type and project
4. **Push toward output** — end every session with "here's the next thing to make"
5. **Don't pad** — Sawna wants concrete help, not summaries of what she already wrote
6. **Be a kind realist** — Sawna tends to be overly optimistic. Flag what could go wrong, as a devil's advocate, not a doom-sayer. She may disagree, but she wants the balanced view.
7. **Consult `🤖 Expansions.md` before adding new vault-level folders or root files** — it's the structure guide and friction layer. The two-yeses test is mandatory before any new top-level structure.

## What Claude Should NOT Do

- **Edit user-created files** (no `🤖` prefix) without asking first

## What Claude CAN Do

- **Create new notes anywhere in the vault**
- **Session logs** go in `00 Notes/` named `🤖 YYYY-MM-DD.md`

## Naming Convention

All Claude-created files:
- **Filename:** prefix `🤖 ` — e.g. `🤖 Integration Setup.md`
- **Front matter:** `author: claude`

---

## Skills

| Skill | Triggers | What It Does |
|---|---|---|
| `brain-setup` | "set up my vault", "initialize my brain", "build my CLAUDE.md" | Scans vault, interviews user, generates personalized root CLAUDE.md |
| `good-morning` | "good morning", "let's get to work", "start my day" | Reads vault, recaps work, recommends most important thing |
| `update-tasks` | "update my tasks", "refresh task lists", "clean up my tasks", "run task update" | Reviews and updates all project `🤖 Tasks.md` files. Auto-runs 7 AM daily; manual on demand |
| `end-of-day` | "/eod", "wrap up", "end of day", "we're done for the day" | Logs session, updates folder structure |
| `new-project` | "new project", "start a project", "create a project" | Interviews, creates project folder + files, updates this file |
| `progress-report` | "progress report", "log our progress", "save where we are" | Creates dated progress note for current session |
| `session-log` | "/log", "log this", "log this session", "save this session", "I want to clear", "capture before I clear" | Appends session block to `00 Notes/🤖 YYYY-MM-DD.md`. One file per day, multiple sessions stack. Run anytime before clearing a chat — no ceremony, just capture. |
| `ingest` | "ingest this", "ingest [URL/file]", "capture this", "process this", "/ingest" | Captures any readable source — YouTube, web article, vault-file PDF, script, transcript, book excerpt — saves raw capture, asks which 1–3 projects it touches, updates only those, appends to `🤖 Wiki Log.md`. Narrow scope by design — adapted from Karpathy's LLM Wiki. |
| `weekly-update` | "weekly update", "let's do a weekly review", "update my vault" | Interviews and updates root `CLAUDE.md`, `GOALS.md`, and each project's `CLAUDE.md`. Chains into `level-up` at the end |
| `level-up` | "level up", "what should I automate", "find me leverage", "what should I build this week" | Walks 3Ms interview (Mindset → Method → Machine) to scope and ship one automation. Chained from `weekly-update`. Writes decision to `🤖 Decisions.md` |
| `youth-stipend-invoice` | "create an invoice", "create a youth invoice", "create a stipend invoice", "make an invoice", "build an invoice" | Reads timesheet, pulls addresses, calculates pay, builds HTML invoice |

---

## Memory System

Two memory layers exist. Treat them differently.

### Layer 1 — Vault `MEMORY.md` (manual, user-maintained)

Lives at `Brain/MEMORY.md`. For facts the user has explicitly asked Claude to remember.

- **Read it at the start of every session.** Use what you find — don't announce it.
- **Write only when the user explicitly asks** ("remember this," "don't forget," "make a note," "log this").
- **Memories are persistent** — never auto-delete or expire.
- **Flag contradictions** — don't silently overwrite existing entries.

*This file is maintained by Sawna. Update it when projects change.*

### Layer 2 — Auto-memory (Claude Code, background)

Lives at `~/.claude/projects/-Users-sawnacaradwyn-.../memory/`. Symlinked into the vault as `🤖 Auto Memory/` for visibility in Obsidian. Captures preferences and corrections session-over-session.

- **Read the auto-memory index at the start of every session.** Use what you find — don't announce it.
- **Propose, don't silently write.** At session end (via `end-of-day`), surface 0–3 candidate auto-memories from the session and ask before writing.
- **Project memories stay gated.** Project-type entries (deadlines, client facts, decisions) still require an explicit ask even with the propose pattern. False signal there is high-cost.
- **Monthly review.** Once a month, open `🤖 Auto Memory/` and skim. Push back on anything wrong or stale.

---
> Source: [SawnaMC/Brain-Vault](https://github.com/SawnaMC/Brain-Vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
