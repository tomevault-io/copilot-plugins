## builder-log-skill

> Interactive guide for logging engineering work to a personal journal. Use this skill whenever the user wants to record something they built, document a completed project, or reflect on their growth across past work — even when they don't explicitly say "log" or "journal". Trigger phrases include "log what I built today", "record this", "write an entry", "journal this debugging session", "write up the X project", "project snapshot", "project portfolio entry", "growth check", "reflect on the last month", "zoom out on my work", "look back at Q1". The skill has three modes — (1) a short entry about a single work moment, (2) a comprehensive project snapshot for portfolio use, and (3) a flexible growth reflection across a curated set of past files. It walks the user through a conversational Q&A, accepts messy natural-language input, structures it using the bundled templates, and writes the final file to the user's current working directory.


# builder-log

A personal engineering journal skill. Helps an engineer record three kinds of things: short entries about individual work moments, comprehensive project snapshots, and zoom-out growth reflections across past records.

All output is plain markdown written into the user's current working directory. No database, no state beyond the files themselves.

## Core principles

These shape every interaction. If a specific step below conflicts with one of these, trust the principle.

**Accept messy input, structure it yourself.** The user should be able to ramble: *"yeah I spent like 3 days debugging a WebSocket memory leak, turned out we weren't closing subscriber refs."* Turn that into Problem / Root Cause / Solution / Lesson. Don't make them learn the template — that's your job.

**One section at a time.** Don't throw the whole template at the user at once. Ask about one thing, get the answer, move on. The reason people abandon journals is the blank page, not the content.

**Skip is fine, faked content is not.** If a section doesn't apply to this entry, mark it skipped and move on. Forcing every field creates hollow journals that don't get reread.

**End with a written file, not a checklist.** Draft the final markdown from the conversation yourself, show it, accept edits, then write. Never leave the user with a template to fill in — that's what failed them before this skill existed.

**Slug-always.** Every entry and growth check must have a clear topic. If the user can't name what it's about, offer to come back later rather than scaffolding an empty file. A vague slug becomes a file nobody rereads.

**Pitch-gate.** Project snapshots require a one-line pitch. If the user can't articulate it after a couple of tries, the project isn't ready to write up yet — offer a stub or park it. Compression exposes understanding; unforced compression exposes its absence.

## Detect the log root

The skill operates on the user's current working directory. Before starting any mode:

1. Check whether CWD contains any of `entries/`, `projects/`, `growth_checks/`.
2. If at least one exists, CWD is the log root. Proceed.
3. If none exist, ask: *"I don't see a builder-log structure here. Create one in this directory?"*
   - On yes: create all three subdirectories and continue.
   - On no: ask where they want to work, `cd` there conceptually (use absolute paths for all subsequent writes).

Never assume a fixed path like `~/builder-log` or any other hardcoded location. The skill must work for anyone who installs it, wherever they run it from.

## Select a mode

Read the user's trigger phrase first. Most map cleanly:

| User says | Mode |
|-----------|------|
| "log what I did", "record this", "journal this", "write an entry" | entry |
| "write up the X project", "project snapshot", "portfolio entry" | project |
| "growth check", "reflect on the last N weeks", "zoom out", "look back at Q1" | growth check |

If the phrasing is ambiguous, ask once: *"Is this a single moment to log, a whole project to write up, or a zoom-out reflection?"* Then commit to the mode and don't second-guess.

---

## Mode: Entry

Purpose: capture a single work moment before it fades — a problem solved, a decision made, something shipped, something learned.

### Step 1 — Slug gate

Ask: *"What's this entry about? One phrase — the topic, not the whole story."*

- If they give a clear topic (e.g., *"WebSocket memory leak"*, *"Postgres migration decision"*): slugify (see [Slug rules](#slug-rules)) and continue.
- If they hesitate or offer vague fluff (*"just stuff I did"*): push back gently. *"Try to name one concrete thing — a problem, a decision, a moment you'd want to remember. If nothing sticks out, this might not be ready to write yet."*
- If they still can't name it: offer to skip, and suggest coming back when something concrete surfaces. Don't scaffold empty entries.

### Step 2 — Filename and collision check

Compute `entries/YYYY-MM-DD-<slug>.md` using today's date.

If a file with that exact name already exists, ask whether this is a continuation of that entry or a separate one. On separate, append `-part-2` or a short differentiator.

### Step 3 — Triage

Ask: *"Quick note or longer reflection?"*

- **Quick**: walk only the sections that clearly matter (usually What I Built, or Problems Solved + Learned). Two or three questions, a couple of minutes total.
- **Longer**: walk the full set of template sections, skipping gracefully where nothing applies.

### Step 4 — Conversational walkthrough

Read `assets/entry_template.md` to get the current section names and structure. Then ask one section at a time, conversationally. Suggested prompts:

- *What did you build or ship?*
- *Any calls you had to make — between approaches, libraries, designs?*
- *Hardest thing you worked through? Walk me through it — what broke, what was wrong, what you did, what you'd do differently.*
- *Any numbers worth remembering? Latency, cost, users, errors — before and after.*
- *What didn't you figure out? What are you still unsure about?* — important to ask explicitly; people self-censor on this section because of ego.
- *Anything new you learned — a tool, a pattern, a concept?*

Accept rambling answers and structure them into template shape. For Problems Solved specifically, draft the Problem / Root Cause / Solution / Lesson quadrants yourself from whatever the user said and let them correct.

### Step 5 — Draft, review, write

Assemble the final markdown using the structure of `assets/entry_template.md`. Show it to the user: *"Here's the draft — anything to change?"* Apply edits, then write to `entries/YYYY-MM-DD-<slug>.md`.

---

## Mode: Project snapshot

Purpose: a portfolio-grade writeup of a completed project. Intended for interview prep, portfolio records, or documenting the arc of a finished piece of work.

### Step 1 — Project identification and resumption check

Ask for the project name. Slugify.

Check whether `projects/<slug>/snapshot.md` already exists.

- **If it exists**: read it. Determine which sections are filled vs. empty (placeholder text, `[...]`, TODOs). Tell the user what you see: *"Looks like you've completed Framing and Technical. Pick up at Outcomes?"* Resume from the right phase. Never re-ask questions they've already answered.
- **If new**: continue to Step 2.

### Step 2 — Pitch gate

Ask: *"Give me the one-line pitch — what would you say about this project at dinner, in one sentence?"*

- If they nail it: continue.
- If they struggle: offer help iterating. *"Try again — what was the thing? Who used it? What did it change?"*
- If they still can't after a couple of tries: offer to save a stub file with the metadata they have and exit. *"Sounds like this project isn't fully ready to write up yet. Want me to save a stub so you can come back, or park it?"*

The pitch gate isn't a formality. If the user can't compress the project to a sentence, they don't own the story yet, and the snapshot will be weaker for it. Better to stop here than produce a foggy writeup.

### Step 3 — Scope check and folder creation

Ask about timeline, team size, and rough scale. If the signals point to small (solo, <2 weeks, scrappy/experimental), suggest the Lite Variant from the template: *"This sounds like Lite-variant territory — shorter form with just the essentials. Want that, or the full snapshot?"* User can override.

Create the project folder: `projects/<slug>/snapshot.md` and `projects/<slug>/artifacts/`.

### Step 4 — Phased walkthrough

Read `assets/project_snapshot_template.md`. The full variant has three phases; walk one at a time, drafting markdown after each so the user sees progress.

**Phase 1 — Framing**: metadata (company, role, timeline, team size), one-line pitch, Problem & Context, Role & Ownership.

**Phase 2 — Technical**: Architecture overview, Key Technical Choices table, Tech Stack split (*what I built directly* vs *what the system used but isn't mine*), What I Considered But Rejected.

*For the Tech Stack split, push for honesty.* It matters: *"What did you actually write yourself? And what did the system use that wasn't yours — managed services, teammate-owned code, vendor APIs?"* This is where people accidentally overclaim, and it's also what interviewers and portfolio readers probe.

**Phase 3 — Outcomes**: Challenges & Solutions, Results (quantitative metrics and qualitative impact), What I Would Do Differently, Artifacts checklist.

### Step 5 — Artifacts pass

At the end, walk the Evidence & Artifacts checklist. For each item:
- Ask whether they have it (screenshot, diagram, link, commit history).
- If yes and it's a file on disk, offer to save it to `projects/<slug>/artifacts/`.
- If no, leave it as `[ ]` so they know what's missing.

Don't block completion on artifacts — the snapshot is useful without them. Just make gaps visible.

### Lite Variant

If chosen, skip the phased walkthrough. Use the Lite block at the bottom of the template: pitch, role, stack (mine), 2–3 results, artifacts, one-line retrospective. One conversational pass, write `projects/<slug>/snapshot.md` in a single go.

---

## Mode: Growth check

Purpose: zoom out across multiple entries or projects to see patterns — what's shifting in trajectory, what's stuck, what to do next. User-triggered, no fixed cadence. The killer feature is surfacing the previous growth check, so each reflection isn't a silo.

### Step 1 — Frame the reflection

Ask: *"What prompted this? And what are you looking back at — a time span, a specific project, or a theme across multiple things?"*

Don't skip this. The prompt-source shapes the whole reflection: end-of-project vs. interview prep vs. a stuck-feeling are very different conversations.

### Step 2 — Define scope

One of:
- **Time span**: *"last 6 weeks"*, *"since March 1"*, *"since my last growth check"*.
- **Specific project**: a project slug.
- **Theme**: a topic or skill (e.g., *"debugging work"*, *"backend architecture"*, *"design decisions"*).

### Step 3 — Auto-detect and curate

Based on scope:
- Glob `entries/*.md` and filter by the date in the filename.
- Glob `projects/*/snapshot.md` and check metadata for timeline overlap.
- Read the title line and first short paragraph (or the pitch, for projects) to get a one-line summary of each candidate.

Present the list:
```
Found 12 entries and 1 project in this scope:
  entries/2026-03-15-postgres-migration.md — "Decided to shard metadata by tenant..."
  entries/2026-03-22-rate-limiter-design.md — "Picked token bucket over sliding window..."
  ...
  projects/payment-rewrite/snapshot.md — "Rewrote the payments pipeline for 10x throughput..."

Which are relevant? (include all / exclude some / add others I missed)
```

Let the user curate. They may prune, add files you missed, or include files outside the original scope.

### Step 4 — Surface the previous growth check

Glob `growth_checks/*.md` and find the most recent. If one exists:
- Read its *"Still stuck on"* and *"What I want next"* sections.
- Show them to the user: *"Last time you said you were stuck on X and wanted to do Y. Want to track those into this one?"*

This is what makes a series of growth checks more valuable than one. Without this step, each reflection is independent, and avoidance patterns go unnoticed.

If no previous growth check exists, skip this step silently.

### Step 5 — Conversational reflection

Read `assets/growth_check_template.md`. Walk through its sections, drawing on the curated files for specifics:

- **Trajectory ratings** — ask, but tell the user they can skip. Some reflections are qualitative only; don't force numbers.
- **The throughline** — *"What connects these files? A skill you kept exercising, a kind of problem you kept hitting, a theme?"* This is the core question; spend real time on it. Ratings can't answer this — it's the reason for curating a set rather than looking at files individually.
- **Hardest thing** — *"Which specific moment in this period was the hardest?"* Name a file.
- **Still stuck on** — carry forward from the previous growth check, and/or introduce new stuck threads.
- **What I want next** — one to three concrete intentions. Specific enough to grade next time.
- **Evidence highlights** — pick three files that best represent this period.
- **Compared to last reflection** — only fill if a previous growth check exists. Be honest: did last time's stuck thread resolve? Did you follow through?

### Step 6 — Draft, review, write

Ask for a slug to title the reflection (e.g., *"q1-reflection"*, *"pre-interview-review"*, *"end-of-payments-project"*). Assemble into the template shape. Write to `growth_checks/YYYY-MM-DD-<slug>.md`.

---

## Templates and file writing

**Template source**: always read from `assets/entry_template.md`, `assets/project_snapshot_template.md`, and `assets/growth_check_template.md` in this skill's own directory. They're the structural source of truth for what sections to include and in what order.

**Why read at runtime** (instead of hardcoding section names in this SKILL.md): the templates will evolve. If their section order or names change, the skill should pick that up automatically without needing a new release. Don't copy template structure into this file.

**File writes**: use absolute paths. Use today's date unless the user explicitly says otherwise (*"log this for yesterday"*, *"the reflection is for last month"*). Use `Write` for new files and `Edit` for resumed project snapshots.

## Slug rules

- Lowercase.
- Kebab-case (hyphens between words).
- Strip punctuation except hyphens.
- Drop articles (*a*, *an*, *the*) if that makes the slug cleaner without losing meaning.
- Target 20–40 characters. If longer, ask the user to shorten.
- Examples:
  - *"WebSocket memory leak"* → `websocket-memory-leak`
  - *"Q1 growth reflection"* → `q1-reflection`
  - *"How I finally fixed the rate limiter"* → `rate-limiter-fix`

## End-of-session

After writing a file, report the absolute path in a one-liner: *"Written to /Users/you/your-log/entries/2026-04-19-websocket-memory-leak.md"*. Don't re-summarize the content — the user just reviewed it.

---
> Source: [C0ldSmi1e/builder-log-skill](https://github.com/C0ldSmi1e/builder-log-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
