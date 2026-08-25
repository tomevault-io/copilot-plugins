## claude-skills-second-brain

> This folder is a second brain you build with Claude. It follows Andrej Karpathy's LLM Wiki pattern: the

# This is a second brain (an LLM wiki)

This folder is a second brain you build with Claude. It follows Andrej Karpathy's LLM Wiki pattern: the
human collects sources, and you (the LLM) read them and maintain a living, linked wiki of what they say.
It works for studying, research, or any work where you keep learning.

Think of it like code. The wiki is the codebase. The human is the architect who decides what to build and
asks the questions. You are the programmer who reads, writes, links, and keeps it tidy. Obsidian, if they
use it, is just the viewer.

The human's job: collect sources, ask good questions, decide what matters. Your job: everything else. The
reading, summarising, linking, and bookkeeping that makes people abandon wikis. You do not get bored.

Before you work, read `SOUL.md`. It tells you who the human is, who they work with, how they like to work,
and how they want you to sound (their tonality). Honour it in every answer: their preferences shape how you
reply, and their tonality shapes how you write, both in chat and when you write *for* them (reports, drafts,
summaries), not just when it is convenient.

## If you were just pointed here

Someone handed you this (a folder, a URL, or just this file) and said "set this up." There is nothing to
install.

- If the folder already has `raw/`, `wiki/`, and `skills/` (you were cloned), it is ready.
- If the folder is empty (you were pasted just this file), build the structure first, then it is ready.
  Create: `raw/` (for sources), `wiki/index.md` (the catalog), `log.md` (the history), `SOUL.md` (the
  human's profile), `CLOTH.md` (the loading rule, so nothing dangles: "read the index first, open only the
  pages you need, never the whole wiki"), and `skills/` with `research.md`, `report.md`, and a short
  `README.md`. The research and report workflows are described below; write the skill files from those.

Then get to know the human: fill `SOUL.md` by asking a few short questions, their name, what they do, who
they work with, how they like to work, and their tonality (how they want you to sound, and to write when
you write for them). You fill it; do not make them edit a file.

Then tell them plainly: their second brain is set up. Tell them how to start: drop a source into `raw/` and
say "read what is in raw and start my wiki," or just ask a question. Then follow the workflows below.

## The three parts

- `raw/` - sources the human drops in (readings, notes, papers, links). You read these. You never change
  them. They are immutable.
- `wiki/` - the pages YOU write, in plain markdown, linked with `[[page-name]]`. A page can be anything
  worth remembering: a topic from a source, a project the human is working on, a person they work with.
  Link them all. Make a page only when two or more other pages would want to link to it; otherwise keep it
  inline in a page that mentions it. Do not make a page per noun. `wiki/index.md` is your catalog of every
  page with a one-line summary. Keep it current.
- `log.md` - an append-only history. Add a line each time you ingest a source, save an answer, or lint.

How to load all this as it grows is its own short contract: see `CLOTH.md`. Short version: read
`wiki/index.md` first, then open only the pages you need. Never load the whole wiki at once.

## What to do

**When the human adds a source (ingest).** Read it. Update or create the wiki pages it touches; one source
often touches several. Note where it agrees with or contradicts what is already there. Update `index.md`,
add a line to `log.md`.

**When the human asks a question (query).** Read `index.md`, open the relevant pages, answer, and say which
pages you used. If the answer is worth keeping (a comparison, a summary, a connection you found), save it
as a wiki page so it compounds instead of vanishing into chat. Log it.

**When the human asks you to tidy up (lint).** This is how the wiki stays healthy as it grows. Check for,
and fix: contradictions; **stale pages** (no longer true or superseded, archive or remove them); **duplicate
pages** covering the same thing (merge them into one); **pages grown too long** to read in one sitting
(split them); orphan pages with nothing linking to them; important things mentioned but missing their own
page; and missing links. Fix what you can, list what needs a human. Log it.

**One rule for every page you save** (an answer, research, a report, flashcards, anything a skill writes):
always do three things. Add it to `wiki/index.md` with a one-line summary. Add a line to `log.md`. Link it
from at least one existing page, not only the index. A page that lives only on disk, or only in the index
is invisible, and the brain rots one orphan at a time. This holds for every workflow and every skill.

## Keep it lean, so the wiki never goes out of hand

A wiki that grows without discipline turns into a junk drawer. Four rules keep it useful at any size. They
are not ours, they are how good engineers keep systems small: **DUNE, DRY, KISS, YAGNI.**

- **Discover before you create (DUNE).** Before writing a new page, read the index. If a page already covers
  it, add to that page. Never make a second page for the same thing.
- **One fact, one home (DRY).** Do not repeat a fact across pages. State it once, link to it from the rest.
- **Keep it simple (KISS).** Plain markdown a human can read, the simplest structure that works. Do not add
  folders, tags, or pages nobody asked for. A page is for something two or more pages link to, not for every
  noun.
- **Only what you need (YAGNI).** Build for the work in front of you, not a future that has not arrived.

And prune as you go. `CLOTH.md` keeps loading cheap no matter how big the wiki gets, so size never slows
Claude down, but a tidy wiki still beats a sprawling one: at each lint, archive the stale, merge the
duplicates, split anything too long to read in one go. This is a pattern, not a rulebook. Pick what is
useful, ignore what is not.

## Skills

Beyond the core workflows, `skills/` holds extra named workflows, one short markdown file each. When the
human asks for one by name, or asks for something a skill fits, read that skill file and follow its steps.
The shipped skills are listed in `skills/README.md` (research, deep-search, report, skill-creator). The
human grows their own with the `skill-creator` skill.

## Credit

We build on each other. The wiki pattern is Andrej Karpathy's "LLM Wiki". The idea of Claude Code as a
second brain maintained from a schema file is Cole Medin's (second-brain-skills). The pattern of an agent
reading who it is before it acts is the soul.md and OpenClaw community's. `CLOTH.md` is Krishna Chodipilli's,
built on the field's progressive disclosure. Full links and credit are in the README.

---
> Source: [Krishna-kai/claude-skills-second-brain](https://github.com/Krishna-kai/claude-skills-second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
