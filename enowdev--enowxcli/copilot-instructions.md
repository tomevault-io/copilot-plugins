## enowxcli

> Design for replacing the three hardcoded roles with an orchestrator and a roster of

# Agents

Design for replacing the three hardcoded roles with an orchestrator and a roster of
specialists, and a record of why each decision went the way it did.

**Status: implemented.** `agent_def.rs` holds the definitions, `discovery/
agents.rs` loads them from disk, `routing.rs` holds the rules, and the run
loop in `agent.rs` resolves an agent per turn. What remains is listed under
[Open questions](#open-questions).

> Not to be confused with `/AGENTS.md` at the repo root, which is an
> instruction file for agents working *on this repository*. This document
> describes a feature *of* enowx.

## The shape

One **orchestrator** is the agent the user talks to; **specialists** do the
work. It was called `router` until 2026-09-27, and that name still resolves:
old sessions, `[agent.models] router = …` and a `router.md` agent file all
mean the orchestrator.

The orchestrator answers what it can settle with a few reads (at most three)
and gives everything else to a specialist. It has no editing tools of its own:
if it could do the job it would, and the roster would go unused.

A specialist is one definition used two ways:

```
HANDOFF                                 SUB-AGENT
orchestrator ──hands over──▶ fe         orchestrator ──calls──▶ fe   (branch)
                             │                                  │ briefing only
    same session,            │ answers                          │ works
    full history             │ the user                         │
                             ▼                                  ▼
    fe ──gives back──▶ orchestrator     orchestrator ◀──report── done
    (when its reply ends)               orchestrator keeps leading
```

| | Handoff | Sub-agent |
|---|---|---|
| Context in | Full history, same session | Briefing only, clean start |
| On finish | Answers the user, then the conversation goes back to the orchestrator | Returns a report |
| Session | One, the agent changes | Branches, then rejoins |

Handoff suits one specialist's work: a page, a feature, a bug. The specialist
answers the user itself, and when its reply ends the conversation returns to
the orchestrator, so the next message is routed afresh. A follow-up on the
same work is handed to it again, and it still has the whole conversation, so
it need not re-read what it already read. A sub-agent suits a piece of a
larger plan, or a one-off result: it is the cheaper of the two for that, see
[Cost](#cost).

## Why this is cheaper, not more expensive

The worry with a large roster is token cost. Measured, it is not the problem:

| Roster | Cost per turn |
|---|---|
| 13 agents, one-line descriptions | ~170 tokens |
| 40 agents, verbose descriptions | ~2,100 tokens |

Against the fixed overhead of a turn — orchestrator prompt, specialist prompt, tool
schemas — the whole roster is about **two file reads**. Descriptions are cheap.

What actually fills a window is accumulated tool output, and that is where the
sub-agent mode wins. For a task touching frontend and backend:

| | Tokens held at the end |
|---|---|
| One agent doing everything | ~47,500 |
| Orchestrator with two sub-agents | ~3,200 in the orchestrator |

Each specialist peaks in its own branch and is then discarded. The orchestrator
carries roughly **15× less**. More specialists, used as sub-agents, means less
context held — not more.

Four rules keep it that way:

1. **Descriptions are written to be chosen from, not read.** One line, domain
   keywords. `- fe: frontend, React, CSS, components, accessibility` is 11
   tokens and enough to route on.
2. **A specialist's prompt loads only when it is used.** The orchestrator needs to
   know a specialist exists and what for, not how it thinks.
3. **Tool schemas follow the active agent.** The orchestrator needs read/glob/grep
   plus handoff/delegate; a specialist's ten-tool schema never reaches it.
4. **Prefer sub-agent for substantial work.** Handoff is comfortable but its
   context accumulates; delegation discards it.

## The roster

Two groups. The split is by what the work *is about* (domain) versus what kind
of work it *is* (cross-cutting), because those are the two questions an orchestrator
can actually answer from a request.

### Domain — which part of the stack

| Agent | For |
|---|---|
| `fe` | Frontend: React/Vue/Svelte, CSS, components, accessibility, bundlers |
| `motion` | Motion: entrances, scroll reveals, transitions, animated drawings, product demos, loaders |
| `be` | Backend: APIs, services, business logic, auth |
| `db` | Schema, migrations, queries, indexing, data modelling |
| `devops` | CI/CD, containers, deploy, infrastructure, observability |
| `mobile` | iOS/Android, React Native, Flutter, native APIs |
| `systems` | Low-level: memory, concurrency, FFI, binary formats |

### Cross-cutting — applies to any domain

| Agent | For |
|---|---|
| `librarian` | Gathers material: reads widely, returns excerpts with paths and line numbers. Does not conclude. |
| `research` | Answers questions: maps the ground and reports findings with conclusions. |
| `review` | Reads diffs and code, reports defects, never edits |
| `test` | Writes and fixes tests, reproduces reported failures |
| `docs` | READMEs, guides, changelogs, API docs, architecture notes, comments |
| `security` | Auth, secrets, injection, dependency risk, infrastructure and privacy audits |
| `perf` | Profiling, hot paths, benchmarks, load tests |
| `general` | Fallback for anything outside the above |

Plus one that is never routed to: `compactor`, invoked by the loop rather than
chosen by the orchestrator. See [Compaction](#compaction). It is not routable
(`AgentDef::is_routable`), so it appears in no roster, is refused as a
delegation target, and `/agent` will not switch to it.

Each specialist's prompt says how its domain is done well and what "done"
means there: `fe` checks every width from a 360px phone up, `motion` reads
the page's timeline of movement and its reduced-motion pass, `db` keeps
migrations reversible and tries them on a scratch database, `devops` runs the
build rather than trusting a file that looks right, `test` makes a new test
fail for the stated reason before making it pass.

### Work that does not look generated

Left to its defaults a model builds the same thing every time: a dark page
with a blue-to-purple gradient hero over three identical cards, emoji for
icons, invented numbers and testimonials, one file holding everything, a
comment above every line, and copy that "unlocks" and "empowers". The prompts
and three built-in skills are there to stop that (2026-09-28).

| Skill | For |
|---|---|
| `ui` | Direction, layout, spacing, type, colour, icons, components, states, responsive, accessibility, content |
| `ui-layout` | The measures: container, grid, spacing rhythm, type scale; section compositions; the phone layout; checks for a generated-looking layout |
| `ui-audit` | Finding the marks of generated work in an existing interface: what to search the code for, how to judge and rank each finding, the report, then the fix |
| `ui-page-*` | One per kind of page (landing, local business, portfolio, docs, dashboard, list and detail, settings, form, sign-in): the skeleton to start from |
| `ui-stack-*` | One per stack (plain HTML and CSS, Tailwind, React, Next, shadcn/ui, Vue, Svelte): its idioms, the generated version of it, and how to look at the result |
| `ui-part-*` | One per part (header, which sticks by default, navigation, back to top, sidebar, page header, footer, hero, sections, cards, social proof, pricing, FAQ, CTA, buttons, links, forms, choices, tables, lists, dialogs, drawers, menus, tooltips, notifications, badges, loading, tabs, breadcrumbs, pagination, search, images, avatars, charts, empty states, steps, dates, uploads, command palette, chat, media): what it is for, how to build it, the generated version to avoid |
| `motion*` | Motion: what moves and why, timing, scroll reveals, interface states, animated drawings, demos, cost, comfort, the audit, each stack |
| `frontend*` | The engineering behind an interface: architecture, state, data, forms, accessibility, performance, testing, SEO, security, errors |
| `backend*` | A server: the API, errors, auth, data, jobs, integrations, security, running it, tests, caching, real-time, files, search, GraphQL, and one per stack (Next.js, Node, Python, Go, Rust, Laravel, Java, .NET, Rails) |
| `database*` | Schemas, migrations, queries, indexes, transactions, Postgres, MySQL, SQLite, MongoDB, operations |
| `devops*` | CI, containers, deploys, platforms, Kubernetes, infrastructure as code, observability, security, networking |
| `mobile*` | Platform UX, React Native, Flutter, iOS, Android, offline data and sync, releases, performance, testing |
| `systems*` | Rust, C and C++, concurrency, memory, FFI, binary formats, the operating system |
| `testing*` | Unit, integration and end-to-end tests, reproducing bugs, flaky tests, test data, tests in CI |
| `docs*` | READMEs, guides, API references, changelogs, comments and commit messages, architecture notes, docs sites |
| `security*` | The audit method, web weaknesses, auth, secrets, the supply chain, infrastructure, cryptography, privacy |
| `performance*` | The method, profiling, servers, memory, benchmarks, load tests |
| `review*`, `research*`, `librarian`, `orchestration` | How to review a change and the checklists by kind; how to answer with evidence, read a codebase, search the web and choose a library; how to gather excerpts; how to run a large task across specialists |
| `code` | Reading the codebase first, structure, names, types, errors, dependencies, frontend specifics, comments, hygiene |
| `writing` | Specific over generic, words to drop, sentences, interface copy, errors and empty states, docs, voice |
| `brainstorm` | When to agree a design with the user before building (only when the user asks for it, or a large new product's purpose cannot be told; never the default) and when not to; how: look first, ask everything open in one `ask` session (always the theme, for a new interface, and which plan documents to write), offer approaches, confirm, write the chosen documents with `plan_write`, hand the agreed design over. Parts: `brainstorm-prd`, `-design`, `-architecture`, `-erd`, `-api`, `-plan`, one template each |

They ship inside the binary (`crates/enowx-core/skills/`), so every install
has them, and are listed with the scope `built-in`. A project or user skill of
the same name replaces one, so a team with its own design system writes its
own `ui`.

A built-in skill goes only to the agents that carry it (`AgentDef::skills`):
it is listed in their prompt and nowhere else, and `skill_read` refuses it to
any other agent that names it anyway. `skill_read` is offered only to an agent
with a skill to read. In the prompt a family is listed by its root, with its
description, and its parts are named on one line under it (`its parts:
ui-layout, ui-part-hero, ...`): the root says which part holds what, and the
reviewer's two hundred skills stay a few thousand characters.

| Agent | Carries |
|---|---|
| `fe` | every `ui*`, `motion*` and `frontend*` skill, `code`, `writing`, `i18n` |
| `motion` | every `motion*` skill, `ui`, `ui-themes`, every `ui-stack-*`, `code` |
| `mobile` | every `mobile*`, `ui*` and `motion*` skill, `code`, `i18n` |
| `be` | every `backend*` and `database*` skill, `code`, `i18n` |
| `db` | every `database*` skill, `code` |
| `devops` | every `devops*` skill, `code` |
| `systems` | every `systems*` skill, `performance-profiling`, `performance-benchmarks`, `performance-memory`, `code` |
| `test` | every `testing*` skill, `backend-testing`, `frontend-testing`, `mobile-testing`, `code` |
| `docs` | every `docs*` skill, `writing` |
| `security` | every `security*` skill, `backend-security`, `backend-auth`, `frontend-security`, `devops-security` |
| `perf` | every `performance*` skill, `frontend-performance`, `database-queries`, `database-indexes`, `backend-caching`, `systems-memory`, `code` |
| `review` | every `review*` skill and every family above |
| `research` | every `research*` skill |
| `librarian` | `librarian` |
| `general` | `code`, `writing`, `i18n`, and the root skill of each family |
| `orchestrator` | `brainstorm` and its parts, `orchestration` |
| `compactor` | none |

Every specialist carries the family for its work, a root skill (the
principles, the defaults and which part skill holds what) and its parts, and
its prompt names which to read for which job. The reviewer carries them all,
since it judges every kind of change. Skills found on disk (the project's and the
user's) are still offered to every agent: nothing says which agent they are
for. An agent file names the built-ins it carries with `skills: ui, code`;
without the field it carries none. `/skills` shows who carries each built-in.

The prompt carries the essentials, read on every call; the skill carries the
depth, read when the work needs it. `fe` is the first specialist rewritten
this way. It reads the project before building (framework, styling, component
library, icon set, tokens, `DESIGN.md`), sets a direction in one line when
there is none rather than falling back to the generated look, keeps one icon
set imported per icon, builds components for named concepts and repeated
markup on tokens, gives every data view empty, loading and error states, uses
real content or labelled placeholders, and reports the direction, stack and
icon set it chose. It reads `ui` and `ui-layout` before designing a page, the
`ui-page-*` skill for the kind of page and the `ui-part-*` skill for each part
it builds (only those), `writing` before a page's copy, and `code` before a new
component or module. An existing interface is audited first with `ui-audit`,
and `review` reports the same marks. The guidance is split one skill per part
so building a hero reads the hero's rules, not thirty components' worth.
The list of built-in skills is generated from `crates/enowx-core/skills/`, and a
test fails when a directory there is not compiled in.

`motion` is built the same way. Its prompt holds the essentials: read what
each part of the page says and let the movement act it out, set the dial
(feedback only, entrances and transitions, or choreography) in `DESIGN.md`'s
`Motion:` line, move only transform and opacity from one set of tokens,
reveal once through one IntersectionObserver, keep the final state as the
base style so no JavaScript, print and reduced motion all show the whole
page, stop every loop off screen, and add no library for what CSS does. Its
skills hold the depth: `motion` (principles and a worked case),
`motion-timing`, `motion-reveal`, `motion-interface`, `motion-drawings`,
`motion-demo`, `motion-performance`, `motion-comfort`, `motion-audit` and
`motion-stacks`. The orchestrator gives it animation work, and in a build it
runs after the `fe` parts whose markup it animates, on its own files.

### Tools for interfaces

Reading the CSS is not looking at the page, and a rule in a prompt is not a
check. The interface agents (`fe`, `motion`, `mobile`) have four tools of their own,
and `review` has the two that only look:

- `ui_check` searches the interface files for the marks of generated work:
  invented figures, dead links and handlers, removed focus outlines, images
  without alt, default gradients, glow, glass, buzzwords, emoji, colours
  outside the tokens, several icon sets. Each finding has a priority, a
  reason and a fix. The harness runs it once more when the agent says it is
  done, on the interface files the turn changed, and sends a high or medium
  finding back to the agent before the turn ends.
- `preview` opens the page in headless Chrome (or Chromium, Edge, Brave;
  `ENX_CHROME` names another) at 360, 768 and 1440px and measures what a
  person would see: horizontal overflow and the element causing it, text
  below AA contrast, links to nowhere, controls without a name, touch
  targets under 44px on a phone, console errors, the number of h1s, a
  missing viewport tag. It also scrolls a long page halfway down: a top bar
  that has left the screen (a sticky bar a wrapper's overflow stops from
  sticking counts) and, past three screens, no visible way back to the top
  are both reported. It saves a screenshot of each width. An HTML file is
  opened by its path; an application by its dev server's url, with the
  command that starts it, which the tool runs, waits for and stops. It talks
  to Chrome over the DevTools protocol, so nothing is installed with it.
  With `motion`, it also watches how the page moves at 1440px, since an
  agent reads text and cannot watch frames: hooks installed before the
  page's scripts record every animation as it starts, then the report gives
  a timeline of what moves on load (when, how long, which element, what it
  animates and how far), what starts as the page is scrolled through,
  grouped by the block each reveal belongs to, loops that never stop and
  whether they keep running scrolled away, animated layout and heavy paint,
  `transition: all`, styles rewritten by a script many times a second, long
  frames, layout shift, text still invisible after scrolling through the
  whole page, scroll and wheel listeners, and the page again with reduced
  motion: what still moves, loops or stays hidden, and smooth scrolling left
  on. Frames of the first seconds and of the first reveal are saved for the
  user.
- `icon` finds icons by meaning in one Iconify set and returns their exact
  SVG, so no icon is drawn from memory.
- `multi_edit` makes several changes to one file in one call, all or none.

The direction a design settles on (the look, the palette, the type, the
theme) goes into `DESIGN.md` at the project's root, and every later change
reads it first, so the tenth screen matches the first.

### Measuring them

A change to these prompts and skills is measured, not judged by one run:

```sh
cargo run -q -p enowx-core --example fe_eval                  # every case
cargo run -q -p enowx-core --example fe_eval -- --case bakery --compare target/fe-eval/<earlier>.json
cargo run -q -p enowx-core --example fe_eval -- --seeds       # the scoring alone, free
```

It runs five cases on the configured model, each in a fresh folder with the
built-in agents and skills only: a bakery's one-page site, a support team's
dashboard, a photographer's portfolio, a generated page to fix, and a
settings page added to a clinic app that already has a `DESIGN.md`. Every
question is answered with its first option. What each case leaves is
scored out of 100 with `ui_check` and `preview` (a high finding costs 8, a
dead link 4, overflow at a width 6, a missing page 40, no `DESIGN.md` 5),
beside the steps, the cost, the skills read and whether the gate sent the
agent back. The scores go to `target/fe-eval/` as JSON, and `--compare`
shows each case's change from an earlier run. It spends what the runs cost.

The shared rule used to allow one skill per task, which stopped agents loading
six skills before any work. It now reads: the skill the agent's instructions
name for the work, or the one that applies, never every skill listed.

`librarian` and `research` are both read-only and are easy to confuse. The
difference is the output: librarian returns **raw material, filtered** —
quotes, paths, line numbers — while research returns **an answer**. Librarian
is the natural sub-agent: read twenty files, hand back the ten relevant
excerpts, discard the rest. Their one-line descriptions, which are what the
orchestrator chooses from, now say so: librarian gathers material *for another
agent*, research *answers a question with evidence*.

The legacy `/role` menu (Orchestrator, Writer, Researcher) was removed from
the interface on 2026-09-27: agents replaced roles, and a role named
"Orchestrator" beside an agent of the same name meant two different things.
Sessions saved with a role still open, mapped to an agent.

### Why not one axis

An earlier draft used domain only (8 agents). Checked against fifteen real
requests, eight had no home: review, tests, docs, dependency upgrades,
security audits, changelogs. They are genuinely cross-domain — "review this
PR" is neither frontend nor backend.

A three-axis catalogue (domain × activity × concern, 17 agents) covered
everything but made routing ambiguous: "fix the bug in the API" is equally
`debug` and `be`, and the orchestrator has no basis to choose.

Two groups with a stated tie-break avoids both:

> **When a request is clearly one domain, use the domain agent. Use a
> cross-cutting agent when the work spans domains or the domain does not
> matter.**

So "review the API changes" → `be` with a review instruction. "Review this PR"
(touching several areas) → `review`.

## Agent definitions

Agents are files, not Rust. This is the load-bearing decision: a roster that
needs a recompile to extend will not be extended. Skills and MCP servers are
already discovered from disk; agents follow the same path.

```
.agents/agents/fe.md          # project
~/.enx/agents/fe.md           # user
```

```markdown
---
name: fe
description: frontend, React, CSS, components, accessibility
tools: read, write, edit, glob, grep, bash, todo
tier: balanced          # optional; default is the active model
---

You are a frontend specialist. …
```

| Field | Meaning |
|---|---|
| `name` | How the orchestrator addresses it |
| `description` | One line. This is what the orchestrator sees; write it to be chosen from |
| `tools` | Tool surface. Filtered twice, as roles are today: never advertised, and refused if called anyway |
| `tier` | Optional model tier, see below |
| `skills` | Built-in skills it carries (`ui`, `code`, `writing`); none when absent |

Project definitions override user ones by name, matching how skills already
resolve.

## Model tiers

A specialist should not always run on the same model as the orchestrator. The orchestrator
mostly classifies; a specialist may be doing the hard part.

Agents name a **tier**, not a model, so a definition survives changing
provider:

```toml
[agent.tiers]
cheap    = "cbc/deepseek-v4.1-flash"
balanced = "cbc/claude-sonnet-4.5"
strong   = "cbc/claude-opus-5"
```

Resolution order, first match wins:

1. User override in config (`[agent.models] fe = "…"`)
2. Tier chosen by the orchestrator for this particular delegation
3. Tier declared in the agent file
4. The active model

The orchestrator picking a tier per call is the point: the same specialist can run
cheap for a small fix and strong for a redesign. Guidance for that judgement
belongs in the orchestrator's prompt and is still to be written — without it the
orchestrator will reach for `strong` every time.

Unmapped tiers fall back to the active model rather than failing; a missing
tier table must not make delegation stop working.

One agent is given a model of its own from the command line, and an empty
value gives it back to its tier. The model is named with its provider, so it
can be on any connected provider, not only the one in use:

```sh
enowx config set agent.models.fe "openrouter/anthropic/claude-sonnet-4.5"
enowx config set agent.models.fe ""
```

An agent whose model is on a provider with no key runs on the model in use,
and the transcript says so.

`/agent` shows the model beside each agent that does not run on the active
one. To choose a model for `fe`, run the interface suite once per candidate
(`fe_eval --model <id>`, see "Measuring them") and compare.

## Control

```toml
[agent]
auto_switch = true    # switch without asking
```

- `true` — the orchestrator hands over or delegates on its own.
- `false` — a prompt appears first: *"hand over to fe?"*

Either way the user can force a switch (`/agent fe`), and forcing overrides
whatever the orchestrator had decided. A specialist picked this way keeps the
conversation until the user picks again: only a handover the orchestrator
made is given back. A pick made before the first message goes with that
message (`RunRequest::agent`), so the session it creates starts with the
pick.

## What was built

| Piece | Where |
|---|---|
| Agent definitions and the shipped roster | `agent_def.rs` |
| Loading `.agents/agents/*.md` over the built-ins | `discovery/agents.rs` |
| Who may switch to whom, contracts, scheduling | `routing.rs` |
| `session.agent`, switch history, branch sessions, usage roll-up | `session.rs` |
| Tier table, per-agent overrides, `model_for` | `config.rs` |
| Per-turn agent resolution, handoff, delegation | `agent.rs` |
| Roster, switch markers, active agent | `enowx-tui` |
| `/agent` | `app/actions.rs` |

The old `Role` enum still exists: sessions written before this resolve their
agent from it, so an existing session file still opens.

## Failure and fallback

A model call fails in four ways, and today all of them take the same path: ten
retries against the same model, then give up.

| Kind | Example | Retry helps? | Another model helps? |
|---|---|---|---|
| Transport | connection reset, timeout | Yes | No |
| Capacity | 429, 503, overloaded | Yes | Yes |
| Capability | context overflow, no tool support | **No** | **Yes** |
| Permanent | 401, unknown model | No | No |

Retrying capacity failures is close to free — the server rejects before
processing, so a refused request is generally not billed; what is spent is
time. The exception is a stream that dies *after* the response started: that
work was done and charged, so it is retried twice rather than ten times.

Capability failures are the ones retrying cannot fix. Those need a different
model, which is what the tier ladder is for.

### The ladder

1. Retry the chosen model, per the table above.
2. Exhausted — try another model in the same tier, if the config lists one.
3. Still failing — drop one tier (`strong` → `balanced` → `cheap`) and say so
   in the transcript: *"strong unavailable, continuing with balanced"*.
4. Bottom of the ladder — fail, naming every model tried.

The notice at step 3 is not optional. A cheaper model may not be up to the
task, and a silent downgrade produces worse work with no indication why.

## Compaction

Compaction is the largest lever on token cost — larger than the agent
architecture. Measured over a thirty-turn conversation:

| | Cumulative input |
|---|---|
| No compaction | 1,395,000 |
| Compact aggressively | 363,000 (26%) |
| + sub-agent architecture | 279,000 (20%) |

The reason is that every turn resends the whole conversation, so cost grows
with the square of the length. Nothing else moves that number as much.

### What it does today

1. Keep the last `keep_last` turns verbatim (default 4), fold the rest.
2. Flatten the folded turns to `USER: …` / `ASSISTANT: …` text.
3. Ask the model for a Goal / Decisions / Files / Open note.
4. Replace the folded turns with one `[COMPACTED SUMMARY]` message.

The session file on disk is untouched — only what is sent to the model
shrinks, so a compacted session still resumes with its full history.

### The problem

The summariser only sees user and assistant prose. Tool calls and their
results are dropped by a catch-all arm before summarisation, and an assistant
turn whose content is empty because it carries `tool_calls` is dropped with
them.

For a working session — twelve reads and six commands — that leaves the
summariser looking at about **7% of what happened**. It is then asked for
"Files: which files were created, edited, or read", information that was
discarded before it arrived. That is why compaction feels like it loses the
thread: not that it folds too much, but that it folds away the part that
carried the work.

### The fix

Include tool calls, summarised rather than verbatim: the tool, its argument,
and a one-line outcome.

```
USER: fix auth
ASSISTANT: let me look
  → read src/auth.rs (240 lines)
  → edit src/auth.rs (+12/-4)
  → bash cargo test → exit 0, 41 passed
ASSISTANT: done, the token check was inverted
```

This costs about **360 tokens** per session for the tool lines, and closes the
89% gap. Full file contents and command output stay out — the point is what
was touched and what came back, not the bytes.

### Compaction as an agent

The summariser becomes a `compactor` agent rather than a hardcoded prompt. It
is a natural fit for the roster: its own prompt, its own tier — `cheap`, since
the job is condensing rather than reasoning — and improvable by editing a file
instead of recompiling.

It is not routed to like the others; the loop invokes it directly, on the
threshold or on `/compact`.

### Trigger

Percentage of the window, as today, plus the existing manual `/compact`.

Worth knowing: a percentage trigger scales with the window, so on a 1M-context
model `auto_compact_at = 0.85` fires at 850,000 tokens — in practice never,
and the conversation pays the full quadratic cost. A large window is for
holding one large file, not a reason to drag 800k of history through every
turn. Users on large-context models should set this far lower.

## Delegation rules

### Depth

One level: only the orchestrator delegates. A specialist that could delegate turns a
task into a tree — three levels at a fanout of three is forty agents and
~600,000 tokens, which no task justifies.

One exception: **a specialist may call `librarian`.** It is read-only, so it
cannot corrupt anything, and it exists precisely to keep a specialist's
context clean — read twenty files, hand back ten excerpts. Forbidding it would
push that reading into the specialist's own window, which is the cost the
architecture is trying to avoid.

So: orchestrator → specialist → librarian, and no other second hop.

### When a sub-agent fails

Five ways it ends badly, and they are not equivalent:

| Failure | State afterwards |
|---|---|
| Wrong specialist chosen | Nothing changed — safe |
| Model exhausted every tier | Nothing changed — safe |
| Specialist declines the task | Nothing changed — safe |
| **Ran out of steps** | **Partial: files already changed** |
| **Interrupted** | **Partial: files may be half-written** |

The first three are simply "pick differently". The last two are the dangerous
ones: work is half done, and an orchestrator that treats them as "try again" will
have a second specialist build on a state it knows nothing about.

So a partial failure is retried **once**, with the same specialist, and the
briefing carries what already changed:

```
Previous attempt ran out of steps. Already changed:
  src/auth.rs   edited (+12/-4)
  tests/auth.rs created
Continue from there; do not redo work that is done.
```

Once. If it fails again the orchestrator stops and reports to the user, naming the
files left in a partial state. Retrying further only stacks up more half-done
edits, and each attempt makes the state harder to reason about.

### Cost

A branch session keeps its own usage, so without rolling it up the sidebar
reports the orchestrator's spend alone: 3,000 tokens shown against 87,000 actually
spent, off by **29×**. That is not an incomplete number, it is a wrong one.

Branch usage rolls up into the parent. The total is what the sidebar shows;
a per-agent breakdown is available for anyone who wants to see where it went.

### Parallelism

Sub-agents may run in parallel, but each one declares a **contract** up front:

```
delegate(agent="fe", task="…", writes=["src/ui/**"], reads=["src/api/types.rs"])
```

- **`writes`** — paths or globs it intends to change.
- **`reads`** — paths it expects to read. Overlapping reads are fine.

Overlapping *writes* are the only conflict. Those delegations are serialised —
the second waits for the first — while everything else still runs together:

```
fe    → src/ui/**        ┐
be    → src/api/**       ├ in parallel
docs  → README.md        ┘

test  → src/ui/**        ← overlaps fe, runs after it
```

Nothing is refused. An orchestrator that had its delegation rejected would have to
re-plan, and it has no better information than the scheduler does.

Declaring intent beats locking: a conflict is known before any work starts,
rather than discovered halfway through when one agent has already written.

The contract is **enforced**, not advisory. A write outside the declared paths
is refused the way an out-of-workspace path already is, and the agent is told
why so it can widen its contract or leave the file alone. An advisory contract
is worth nothing while other agents are running against it — the scheduler
serialised on the strength of that declaration, and a write outside it
reintroduces exactly the conflict the declaration ruled out.

A refused write does not kill the delegation. The specialist keeps running and
can ask for a wider contract; only the one call fails.

## A sub-agent is not a participant

The user talks to one agent. A sub-agent is work that agent chose to hand
out: there is no way to send it a message, no way to steer it, and it takes
its instructions from the calling agent alone.

That has to hold for the keys too. Ctrl+C means "stop what I asked for",
which is the conversation — and cancelling a sub-agent from outside left it
dead mid-task while the caller was handed a `NO REPORT` that could not be
told apart from one that had genuinely given up. So a delegation runs on its
own cancellation token, not a child of the caller's: stopping the turn stops
the turn, and the sub-agent already under way finishes and reports. The
cancelled turn makes no further model call, and starts no new delegation.

While a sub-agent's transcript is open, Ctrl+C and Esc both mean "leave this
window", and sending a message returns to the conversation first. Nothing
typed there can reach the sub-agent, which is the point.

What still ends a sub-agent: its own step limit, and its own errors.

## A delegation owes a report

The caller sees nothing a sub-agent does — its events are dropped on purpose,
because interleaving two agents' tool calls in one transcript is unreadable.
So its final message is the entire interface between them, and everything the
caller decides next rests on it.

Left to itself that message is whatever the sub-agent happened to say when it
ran out of work. Real ones, from sessions in this repo:

```
[delegation to `fe` finished]
Now rewriting `js/main.js` with the corrected logic.

[delegation to `review` finished]
Let me check the touch-target claim rigorously — …
```

Both are narration from the middle of a task, passed up as results. A third
sent the caller a question it had no way to answer.

So the task carries the shape of the report with it:

```
DONE:     what you achieved, or what you could not
CHANGED:  every file you created or edited, or `none`
VERIFIED: how you checked it (the project's build or tests, or reading the result back) and what you found, or `not verified`
NEXT:     what the caller must know to carry on, or `nothing`
```

And when a sub-agent does not write one — it ran out of steps, or stopped
mid-sentence — the report is assembled from what the branch actually did,
opening with `NO REPORT` so the caller cannot mistake it for a result. The
files come from the tool calls rather than the prose, so the list is what
happened rather than what the model said happened.

`VERIFIED` is the line that matters most. A sub-agent's report is a claim; a
caller that treats "built the page" as "the page works" is building on
something nobody checked.

The report is one line per field, and the sub-agent stops after the task it
was given. The interface shows it under the delegation's row as a label
column and a value column. The parent session records each delegation's
branch (`Session::delegations`), so a resumed session can still open the work
behind a report; `routing::report_message` and `parse_report_message` are the
one place the report message's format is written and read.

## Asking the user

Added on 2026-09-28. Until then no agent ever asked the user anything: the
shared rules told every agent to choose a default rather than stop to ask,
and there was no way to ask. Now the agent holding the user's conversation
(the orchestrator, or a specialist it handed the conversation to) has an
`ask` tool (`ask.rs`).

- **When.** Before something that cannot be undone, or when a choice changes
  what gets built and neither the request nor the project settles it.
  Otherwise the agent still chooses and says what it chose.
- **Shape.** One question, or a few related ones (at most five), each with
  up to five options, the recommended one first. The model never adds an
  "Other" option: the interface always offers one.
- **Answer.** The turn waits on it; the answer comes back as the tool's
  result, question by question: the options chosen, an answer in the user's
  own words, and any note the user wrote on an option.
- **Who cannot ask.** A delegated sub-agent: the user cannot see it, so its
  questions go in its report for the agent that called it. A host where no
  one can answer (the HTTP server, the headless example by default) does not
  offer the tool (`Agent::asking_user` turns it on), so no run waits on a
  question nobody will see. The headless example turns it on and answers
  every question with its first option.
- **Stopping.** Stopping the turn is the way out of a question; the agent is
  told the user stopped instead of answering.

## Handoff carries a summary, not the raw history

A handoff keeps the session, so the obvious question is whether the previous
specialist's prompt should be compacted away. Measured, it should not matter:
after five handoffs the retired prompts are about 6% of the context. The
accumulated *work* is the rest.

The real reason to summarise is correctness, not size. A file read before it
was edited is now **wrong**: the frontend specialist read `src/auth.rs`, edited
it, and handed over — the backend specialist inherits the pre-edit text and can
reason from code that no longer exists.

So a handoff passes on a summary:

```
User asked: …
Already decided: JWT rather than sessions
Files touched:
  src/auth.rs    edited   ← re-read if you need it
  tests/auth.rs  created
Last command: cargo test → 41 passed
```

What carries: the user's side of the conversation, decisions already made, and
which files are in what state. What does not: file contents, which may be
stale, and the retired specialist's prompt.

The `compactor` agent writes it. The job is the same one it already does —
condense without losing the thread — so there is one place to improve rather
than two.

The risk to watch is summarising too hard: a specialist that loses the thread
re-reads everything, which spends the tokens that were saved plus another
round of tool calls. Naming the files rather than dropping them is what keeps
that in check — the specialist knows what exists and can fetch what it needs.


## The orchestrator's prompt

The orchestrator is the piece most likely to decide whether this works. Every
judgement the design defers — which specialist, which tier, handoff or
delegate, how to split parallel work — lands here. A vague prompt produces an
orchestrator that picks `general` and `strong` for everything, and the roster is
decoration.

It needs to answer five questions, in order.

### 0. Answer, or route

Added on 2026-09-27. Before, every question went to a sub-agent, even "where
is X defined?", which cost a whole delegation for one grep.

```
Conversation (a greeting, thanks, working out what the user means) needs no
tools. A question you can settle with a few looks, such as where something
is defined, what a function does, what changed lately, which versions are
installed or whether the tests pass, you answer directly: at most five
reads, searches or commands, then the answer with paths and line numbers or
what the command showed. A question that needs more than that goes to
`research`. Anything that changes files goes to a specialist.
```

On 2026-09-28 the orchestrator's tools grew from `read`, `glob` and `grep`
to those plus `bash`, `fetch` and `todo`. With only reads it could not say
what changed (`git status`, `git diff`), which version was installed, or
whether the tests passed, so each of those cost a delegation; it could not
read a link the user pasted; and a plan of several delegations had nowhere
to show its progress. It still has no tool that edits.

`bash` in an agent without `write`, `edit` or `multi_edit` (the orchestrator
and `review`) is for looking and checking. The harness refuses, before
running it, a command that would change something
(`tools/shell_guard.rs`): a redirect to a file, `tee`, `rm`, `mv`, `cp`,
`mkdir`, `sed -i`, `find -delete`, a git command that changes the tree, the
index or a remote, a package install, a formatter's `--write` or `--fix`, a
download to a file, an inline script that writes, a dev server that never
returns. The refusal says why and tells the agent to route the change. It
is a guard against habit, not a sandbox.

### 1. Which specialist

```
Pick the agent whose description matches the work, not the words.
"The login page is broken" is `fe` if the page renders wrong and `be` if
the request fails — read enough to tell which, then choose.

When the work is clearly one part of the stack, use the domain agent.
Use a cross-cutting agent when the work spans domains or the domain does
not matter: `review` for a PR touching several areas, `docs` for a
changelog, `security` for an audit.

`general` is for work that fits nothing above. Reaching for it often means
the roster is missing an agent — say so rather than quietly absorbing the
task.
```

### 2. Handoff or delegate

The first version defaulted to delegation. In practice a request like "build
me a page" is followed by "make the header smaller", "now the footer", and
each follow-up started a fresh sub-agent that read everything again. So the
second version preferred handoff for work the user would iterate on, and the
specialist kept the conversation until the user moved on.

That left the conversation with whichever agent spoke last (changed on
2026-09-28): the user's next message, about anything, went to the specialist.
Now a handoff lends the conversation for one reply. The specialist answers,
the conversation goes back to the orchestrator (`↳ fe → orchestrator ·
finished`), and the next message is routed afresh. A follow-up on the same
work is handed over again, to a specialist that still has the whole
conversation. Giving it back calls no model; only the holder changes.

```
Hand off when the request is one specialist's work: building a page or a
feature, a design, a bug to fix. The specialist works in this conversation,
with everything said so far, and answers the user; when its turn ends the
conversation comes back to you. A follow-up on the same work goes to it
again: it still has the whole conversation, so nothing it learned is lost.

Delegate when the work is one piece of a larger plan you are coordinating,
or a one-off whose result you report back: a review, an investigation, a
single fix. The specialist starts clean, returns a report, and its context
is discarded.
```

`Session::lent_by_orchestrator` holds the rule. The conversation goes back
when the orchestrator made the last handover, never when the user picked the
specialist with `/agent`, and never inside a delegated branch. It goes back
when the reply ends, and also when the reply failed or was stopped. A session
left with a specialist anyway (the process was killed, or the session was
saved before this rule) goes back when the next message arrives.

The agent taking over is told so. It sees the previous agent's messages as
its own, and continuing them, `fe` once answered "I've passed this to `fe`"
and stopped with the page unbuilt. So on the turn after a handoff its system
prompt ends with a `HANDED TO YOU` note: who handed it over and why, that the
user's last request is its to handle now, and not to repeat or describe the
handoff.

A specialist holding the conversation still gets a `handoff` tool that can
name only the orchestrator, for a request that turns out not to be its work;
the orchestrator then answers in the same reply. One working in a delegated
branch gets none, since it reports back instead.
`Delegation::may_hand_off_to` and `routing::HandOff` hold that rule.

### 3. Which tier

Without guidance an orchestrator picks `strong` every time, because nothing punishes
it for doing so. The prompt has to make the cost legible:

```
cheap     mechanical work with a clear specification: rename, format,
          apply a stated pattern, gather files
balanced  normal implementation: a feature with known shape, a bug with a
          known cause, tests for existing code
strong    work requiring judgement: unclear cause, design decisions,
          unfamiliar code, anything where a wrong answer is expensive

Start one tier lower than feels right. A `balanced` attempt that fails
costs less than a `strong` attempt that was never needed, and the ladder
promotes on failure anyway.
```

That last line matters: the fallback ladder already goes *down* on failure,
so starting low and being wrong is recoverable. Starting high and being wrong
is simply paid for.

### 4. How to split parallel work

```
Two delegations may run together when neither writes what the other writes.
Declare `writes` honestly — too narrow and the specialist is refused
mid-task; too wide and it blocks work that could have run alongside.

Prefer splitting by area, not by activity: `fe` on the components and `be`
on the endpoints can run together; "implement" and "test" on the same files
cannot.
```

### What the orchestrator must not do

```
Do not implement. Your tools are for answering quick questions, choosing a
specialist and checking a report, not for doing the work: past five looks,
a question is `research` and a change is a specialist's.

Do not chain delegations to build a result yourself. Delegate the task, not
each step of it — the specialist plans its own steps.

Do not present a specialist's work as your own. Report what came back.
```

Each of these is a failure seen in routing agents elsewhere: reading until the
task is done, decomposing into tool-call-sized fragments, and laundering a
specialist's output. They are cheap to forbid and expensive to discover.

### Reading, briefing, and after

Added once real sessions showed the orchestrator reading its way through a project
before delegating a portfolio page: eighteen calls in one session, four in
another, for a request that named its kind of work.

```
Most requests name their kind of work: "build a portfolio page" is `fe`,
"this query is slow" is `db`. Route those straight away, without reading
anything. Read only when the request leaves the specialist genuinely open,
and then one or two small reads at most. The specialist reads the files it
needs itself.

Say what the user wants and any constraint they stated, in a few lines. Do
not plan the specialist's steps or invent requirements the user did not give.

Answer the user from the report. Do not re-read the specialist's files to
check its work unless the report leaves something the user asked about
unclear.
```

A later run stalled the other way: asked for "a simple portfolio", the
orchestrator delegated nothing and sent the user three questions (whose content,
which stack, where) as prose at the end of its reply, after reading a story
into the workspace's folder name. That became a rule never to stop and ask,
and then no agent ever asked anything, including before building a whole new
project on guesses.

Since 2026-09-28 the two are split. Work whose shape is open is brainstormed
first, through the `ask` tool and the `brainstorm` skill; small details in
work whose shape is settled are still chosen, not asked:

```
BRAINSTORM FIRST WHEN THE SHAPE IS OPEN
A new project, a new feature or page, a redesign: work two reasonable
specialists would build differently. Before routing it, read the
`brainstorm` skill and agree the design with the user through `ask`, one
question at a time, then hand the agreed design over as the brief. Not for a
fix, a small change with a clear result, a question, work the user already
specified, or when they say to just build it.
```

Since 2026-10-04 that rule is narrower. The orchestrator brainstormed nearly
every request, because any new page or feature counted as "work two
specialists would build differently", and it asked which checks to run even
without a brainstorm, so even a small, clear project began with questions and
the lead looked unable to decide. The orchestrator now decides by default:

```
DECIDE BY DEFAULT
You lead, so decide. A request to build something goes to work now: fill what
it leaves open from the request, the code and common convention, ... and put
your assumptions in one line in the brief and in your answer. ... Scale the
ceremony to the work: a small project, one page or one feature gets no
brainstorm and no plan documents.

BRAINSTORM ONLY WHEN ASKED, OR WHEN A LARGE PRODUCT IS UNREADABLE
Brainstorm when the user asks for it (to brainstorm, plan, or settle the
design before building), or when the work is large (a new product across
several areas) and what it is for or who uses it cannot be told from the
request or the workspace. ...

ASK ONLY WHAT IS THE USER'S
Ask only for what is theirs to decide and you cannot infer: taste that changes
the result and nothing in the request or the project hints at, spending money
or a paid service, anything destructive or that cannot be undone, credentials
or accounts. Once, in one `ask`, a few options with the one you recommend
first. Everything else you decide and say what you chose.
```

The check wave (`review`, `test` where there is logic, `security` where it
matters) now runs without a question, and plan documents are written only
when the user asks for them or chooses them in a brainstorm. Inside a
brainstorm, the stack and architecture are decided, not asked.

When a brainstorm does run, the skill asks what is still open in a fixed order
(what it is for and who uses it, the first version's scope, how it should
feel, the constraints only the user can name),
three to six questions and never more than eight, then offers two or three
approaches, confirms the design in a few lines ("Build it like this?"), and
hands it over with every decision the user made. The earlier rule survives
for details:

```
Small details left open in work whose shape is settled are not a reason to
stop: choose the plainest thing that does the job, with placeholder content
marked as such, say so in the brief, and route it. A question goes through
`ask`, with options, never as prose at the end of a reply; without `ask`
there is no one to ask, so choose and say what you chose.
```

## Effort

Every model call re-sends the whole context, so a step spent on busywork
costs as much as one spent on the task. One orchestrator session in this repo sent
2.26M input tokens. The specialists showed the same habits in every session
read: `bash ls`, `find`, `cat` and `sed -n` where glob and read fit; a
checklist for a two-file page with each item ticked in its own call; six
skills loaded before any work; throwaway Python to check a stylesheet.

Every agent's prompt now carries the rules against those, next to the rules
about facts and paths:

- Match effort to the task: a small task is look, write, check once, report.
- Use glob, grep and read for files; `bash` builds, runs and tests.
- Request independent reads and searches together, in one step.
- Create a file whole with one `write`; change it with `edit`.
- Verify with the project's own build, tests or linter, or by reading the
  result. No throwaway scripts. (The report once asked for "what you ran"
  under VERIFIED, which pushed sub-agents to run something: a real run
  started two HTTP servers and a Python HTML checker for a static page.
  Reading the result back is a check too, and the contract now says so.)
- Leave nothing running: no servers or background processes that outlive
  the command.
- `todo` only for four or more steps: set it once, tick finished steps
  together. `todo done` takes several items for that reason.
- Read a skill only when the task needs it, and only that one.
- When a detail is open and a sensible default exists, choose it and say
  what was chosen, rather than stopping to ask.
- Stop when the request is met.

The tool descriptions for `bash`, `todo` and `skill_read` say the same thing
where the model reads them.

## Open questions

- **Orchestrator prompt wording.** The shape is settled above, but the exact text
  will need iterating against real sessions — routing quality is not something
  a first draft gets right.
- **Contract granularity.** `writes: ["src/ui/**"]` is easy to declare and
  easy to check. But a specialist that needs one file outside its area has to
  widen the whole glob. Is per-file amendment worth the complexity, or is
  re-delegating cleaner?

---
> Source: [enowdev/enowxcli](https://github.com/enowdev/enowxcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
