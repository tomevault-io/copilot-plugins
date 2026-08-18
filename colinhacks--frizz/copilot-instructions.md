## frizz

> This repo does NOT use pull requests. Ever. NEVER run `gh pr create` (or the `gh api .../pulls` POST

# No pull requests — land on local `main`

This repo does NOT use pull requests. Ever. NEVER run `gh pr create` (or the `gh api .../pulls` POST
equivalent, or the GitHub UI) — not for a typo, not for a big feature, no size threshold, no
exception. This binds EVERY agent that touches this repo: the top-level worker and every sub-agent it
dispatches. If you find yourself about to open a PR, STOP — in this repo that is always the wrong move.

Land finished work on this machine's local `main` instead: work directly on `main`, or do
isolated/messy work in a git worktree on a local branch (`git worktree add <dir> -b <slug>`) and, when
it's done and you're confident, merge it back yourself (`git switch main && git merge <slug>`) and
remove the worktree. Getting the change onto local `main` is your job — never push a branch to stage a
review and never hand back an unmerged branch. Reading GitHub (issues, PRs, CI) is fine; creating or
pushing a PR is not. (`FRIZZ.md` states this in full; it is repeated here because Codex re-reads
`AGENTS.md` fresh every session and sub-agents load it, whereas the frizz worker contract can be frozen
at session start.)

Git hygiene does NOT matter here — the only thing that matters is getting your changes onto local
`main`. Many agents (and the human) work in parallel against the same repo constantly, so `main` moves
under you and the shared working tree is often dirty with someone else's in-progress edits. Do not let
that block you and do not wait it out: commit whatever is in the working tree first (an intermediate
`git add -A && git commit -m "wip: snapshot in-flight work"` is fine — committing another agent's
uncommitted changes to unblock your merge is EXPECTED, not a violation), then merge your branch in and
resolve conflicts favoring a correct build of your own change. A messy history, a WIP commit that mixes
several agents' work, an ugly merge — all fine. Never `git stash` in the shared tree (it corrupts
concurrent agents), never force-discard someone's committed work, and never stall a landing on "the
tree isn't clean." Keep merging into `main`.

## Committing out of a tree other agents are editing

**Default to `git commit -m "…" -- <paths>`.** The pathspec form commits the working-tree content of
exactly those paths on top of `HEAD` through a temp index git seeds for you, so a concurrent `git add`
by another agent cannot ride along and the shared index is left as you found it.

**Do NOT reach for `GIT_INDEX_FILE=/tmp/idx git add … && git commit`.** A temp index path that does not
exist yet starts EMPTY, not as a copy of `HEAD` — `git commit` then writes a tree holding only the
paths you added, recording *every other tracked file* as deleted. The working tree is untouched, so
nothing looks wrong until someone checks that commit out. This has cost real recovery cycles more than
once. If you genuinely need a private index (only to `git apply --cached` one hunk out of a file
another agent is also editing), run `git read-tree HEAD` immediately after setting the path.

Either way, **verify the tree right after committing**: `git ls-tree -r --name-only HEAD | wc -l`
should be roughly what it was before, not collapsed to the size of your change.

`scripts/githooks/pre-commit` backstops this — it refuses any commit that records files as deleted
while they still exist on disk, which a genuine `git rm` cannot trigger because that removes the file
from disk too. It is wired through `core.hooksPath`, which is LOCAL config, so in a fresh clone run:

```sh
git config core.hooksPath scripts/githooks
```

**Keep that value RELATIVE, and re-check it whenever the checkout moves.** Git accepts an absolute `core.hooksPath`, and this machine had one — `/Users/colinmcd94/Documents/projects/fray/scripts/githooks`, left from before the directory was renamed to `.../frizz`. Git does not warn when `core.hooksPath` names a directory that does not exist; it simply runs no hooks. Measured 2026-08-11 with a control pair: pointed at a real dir, a failing hook blocks the commit (exit 1); pointed at a missing dir, the identical commit succeeds (exit 0). So the backstop above was silently OFF for every agent commit between the rename and 2026-08-11 — the one guard against a tree-collapsing commit, gone exactly while nobody could tell. `git config core.hooksPath` must print `scripts/githooks`; if it prints an absolute path, reset it.

# Web UI completion rule — proportionate, not reflexive

Browser QA is for the changes where a browser is what actually settles the question, not for every diff
that happens to touch a `.tsx` file. Judge the change in front of you.

**Drive it in Chrome, and carry the evidence in your handoff** (inspected screenshots, console/page-error
check, the optical-review result, explicit browser-cleanup confirmation) when the change is: a new or
restructured surface, layout or interaction flow; anything judged OPTICALLY — spacing, alignment, colour,
truncation, a glyph beside text; behaviour you cannot predict from the code alone (live state, timing,
streaming, restart/recovery, a seam between processes); a fix no test pins where the bug actually lives;
or anything large, cross-cutting, or that you are unsure of.

**Skip it for the small and the certain** — a targeted fix in code you have read, pinned by a test at the
right level, with a blast radius you can name; docs, comments, types, provably mechanical edits; and
anything with no browser surface at all (prove that in its own real runtime). Then say plainly in the
handoff what you verified and how, so the maintainer can weigh the call. Confidence is earned, never
asserted: "it should work" is not confidence, a green unit test over a stubbed seam is not either, and
you never describe something as driven or verified when it was not.

**Load the skill that matches what you need**, and compose them — they were one 300-line skill until
2026-08-08, which meant a backend fix had to read the browser rules and a CSS tweak had to read the
stack rules:

- **`frizz-stack`** — boot a real, fully-isolated disposable Frizz (`scripts/adhoc-stack.mjs`), including
  the multi-project/tenant case, and seed real state through its own RPC surface.
- **`headless-browser`** — take the shot without putting a window on the maintainer's screen
  (`scripts/shot.mjs` is the DEFAULT; Chrome DevTools MCP second, and only because this repo forces it
  headless), which states and widths to capture, browser process hygiene (one owned instance per task;
  never a global close or a broad `pkill`), and how to embed evidence so Frizz renders it inline.
- **`real-subsystem-harness`** — for behavior no browser can reach: the broker socket, a pty, spawn/exec
  paths, migrations, a detached daemon's environment. Real resource, real function, negative control.

# Visual alignment is the implementer's job, not a review someone else does

**Load the `visual-review` skill whenever you place an icon, glyph, emoji, badge, chip, or counter next
to text — and before you declare any new UI correct.** It carries the ink-measurement routine, the
per-glyph offsets it produced, and the instrument bug that makes a naive baseline probe report ~3x the
real error. Two non-negotiables from it:

- **You are the first reviewer of your own screenshot.** Capturing evidence is not reviewing evidence.
  Read the shot back and actively hunt for what is wrong with it — a glyph riding high, mismatched visual
  weight, a collision at a narrow width. Never hand over a screenshot you have not personally critiqued.
  Capture at a scale where the detail is judgeable; a 40px component inside a 1400px shot cannot be
  reviewed, and glancing at it counts for nothing.
- **Icon-beside-text alignment is an INK problem, and every glyph differs.** `items-center` centers a
  glyph's BOX on the flex line; the eye aligns ink. A digit has no descender, an SVG's ink sits wherever
  its path falls inside its viewBox, an emoji ignores your font size — so one shared nudge cannot fix a
  cluster. Measure each glyph, correct each in `em`, then re-measure and confirm the residual is ~0.

**And load the `optical-spacing` skill whenever you set or judge the spacing of a ROW of controls** —
an icon strip, a button rail, a footer of mixed glyphs and pills. The same law sideways: `gap` spaces
boxes, so a bare glyph in a hover square and a bordered pill on one uniform `gap` drew ink distances
from 5.78px to 20.50px on a single strip. It carries the ink-gap instrument (this repo's copy is
`scripts/ink-gaps.mjs`), the negative-margin fix, and the pen-width rule for matching perceived weight
— which no colour token can fix. It is a GLOBAL skill, not one of this repo's, so it needs no entry in
`.agents/skills/`.

## Every UI change gets an OPTICAL SPACING PASS before you call it done — no exceptions, no threshold

Any time you implement or touch any UI, measure the spacing the eye actually reads before you hand it over. This is not conditional on the change looking like an "icon strip", and not something to reach for only when someone complains. It is the last step of implementing UI, the same way running the tests is the last step of implementing logic.

- **Two marks on one line ARE a row.** A label and its chevron. A gerund and its elapsed clock. A badge beside a title. If you can name two things sitting side by side, the skill applies — that is the reading that keeps getting missed, because "row of controls" sounds like a toolbar and a disclosure row does not.
- **A `gap` is not a distance.** It spaces BOXES, and small glyphs are mostly empty box. Lucide's chevron paints 4.67 of its 14 box px, a third of dead space per side — so `gap-1` beside its label and `gap-2` before its clock drew **9.06px and 13.00px** of ink where the CSS claimed 4 and 8, and the chevron floated equidistant between the two instead of reading as the label's handle (maintainer 2026-08-05: *"Fix the fucking optical spacing on that chevron"*). Nothing in the CSS looks wrong; you only see it by measuring the pixels.
- **Run the instrument, both axes.** `scripts/ink-gaps.mjs` for horizontal ink gaps, the `visual-review` ink routine for vertical ink-vs-cap-band. One caveat that will bite: once a negative margin makes boxes overlap, `ink-gaps.mjs` unions the NEIGHBOUR's ink into the mark and reports nonsense — switch to geometry then (an SVG child's `getBoundingClientRect` IS its ink box; canvas `actualBoundingBox*` gives a string's ink edges).
- **Then look at it, cropped, at dsf 6–8.** The numbers say whether the gap is what you set; only the picture says whether what you set is right. Both failures happened here in one sitting: shipping 9.06px because the CSS said 4, then over-collapsing to 4.4px because the trim was correct and the target was not.
- **THIS APP RENDERS IN TWO FONTS, and a hand-fitted constant is only ever right in one.** The prose/UI font is a user setting — `html[data-font="sans"|"mono"]`, applied in `index.html` before first paint — so every glyph placed beside text ships against two different cap heights. A fixture page that does not set `data-font` silently renders the MONO default: a chevron measured there at a 0.00px residual rode visibly high in the maintainer's sans window (2026-08-05: *"this is awful"*). Set `data-font` in any fixture you measure on, and check both values.
- **Prefer a correction the BROWSER computes to one you fit by hand.** `1cap` is the resolved font's cap height, so `self-baseline` + `translate-y-[calc(0.5em_-_0.5cap)]` puts a symmetric 1em glyph's ink exactly on the cap band **in any font, at any size**, with nothing to re-measure when the setting flips or the type scale moves. Reach for a measured `em` constant only where no derivable reference exists — and say which font you measured. (`align-self: baseline` needs a shared baseline to align against: if the row is `items-center`, the glyph has nothing to align to and silently lands ~1px off. Make the row `items-baseline`.)
- **Put the corrections in ONE place with the readings in the comment,** not per call site. Three chevrons in one column each placed by hand had drifted into two vertical offsets, two tones, and a horizontal rhythm nobody had ever measured. They are measurements, not taste — the next reader has to know to re-measure rather than re-guess.

Do not ship "it renders" and wait to be told it looks wrong. If the pattern exists in a real product
(GitHub, Linear, this app's own components), measure the real one and mirror it instead of designing
from taste.

# Copy capitalization: sentence case, never title case

All user-visible copy uses SENTENCE case — capitalize only the first word and any proper nouns. This
covers button and menu labels, headings, section titles, toasts, and thread titles. Never Title-Case
Every Word (write "Confirm snooze", "Mark as done", "Fix queue focus" — not "Confirm Snooze", "Mark As
Done", "Fix Queue Focus"). Acronyms (PR, CI, API) keep their established casing. When an agent titles a
thread, the same rule applies.

**"Frizz" is a proper noun — always capitalize it in prose.** The product is Frizz; write "Frizz
dispatches a worker", never "frizz dispatches a worker". Lowercase survives only in literal
identifiers, where it is part of the name: `npx frizz`, `FRIZZ.md`, `.frizz/`, `~/.frizz/`,
`frizz-<slug>` session names, the `frizz`/`frizz-update` CLIs, and the `frizz:*` skill and sub-agent profile names.

# There is NO tmux. Agents are detached broker daemons

Every agent that touches this repo eventually tells the maintainer that stopping Frizz is safe "because the agents are in tmux," and every one of them is wrong. They are reading it off the codebase: `tmux_name` is a session column, `FRIZZ_LAUNCH_TMUX_SOCKET` is a launch variable, and dozens of comments in `router.ts`, `tailer.ts` and `dispatch.ts` still narrate a tmux pane. **None of it runs.** There is no `tmux.ts`, nothing imports one, and dispatch never execs `tmux` — check it in ten seconds with `ls packages/server/src/tmux*` and `grep -rn '"tmux"' --include='*.ts' packages/server/src`.

What actually happens: a Claude thread is `claude_runtime="broker"`, and `claude-broker-host.ts` forks a daemon with `detached: true, stdio: "ignore"` into its **own process group**. That single fact is the answer to most operational questions — Ctrl-C on the server signals the launcher's group and cannot reach the daemon, so a running turn survives a stop; its events queue in a 20,000-frame backlog rather than dropping while nothing is attached, and the bridge replays them on reconnect. Sub-agents live inside that daemon's SDK session, so they ride along too.

**So: never tell the operator anything about tmux, and never repeat "the agents are in tmux" as a reason a restart is safe.** It is safe, for a different reason, and saying the wrong one has cost real trust here. The surviving tmux names are legacy spellings of "the identity string for this thread" — kept because the column is load-bearing on disk. Treat any present-tense tmux comment as stale, and prefer `git log -S` over believing it.

# Board nomenclature: "active" means SPINNING, and nothing else

The sidebar's row groups have names the maintainer uses precisely (2026-08-05: *"when I say active, I'm only referring to the things that are currently spinning; the things beneath that, I would refer to as rested, or just items in the queue"*). Use them in code, comments, copy and when reporting back. (The quote says "beneath" because Rested sat below Active when it was said; on 2026-08-08 the two bands SWAPPED position — the cue moved up under the prompt box — and the vocabulary did not change with them.)

Top to bottom:

- **Rested** — the CUE: the top band, directly under the prompt box; the same set as **"the queue"** / "items in the queue", one row per card, each with a right-justified rest time.
- **Active** — the rows below that rule, in practice the ones currently spinning. Never carries a queue card, and never a rest time — the rule is drawn on the CARD, so a thread the server excused from the queue while it rests lands here too (see `ARCHITECTURE.md`).
- **Held** — the dimmed, labeled park band (a `human:` gate, a future `timer:`, a wall-clock snooze, an auto-resumed limit pause).
- **Done** — the collapsed archived section.

The trap is that `groups.ts` `sectionOf` returns `"active"` for Active AND Rested rows alike — that key names the `<section>` holding both bands, not the maintainer's word. `partitionActive` splits it (`.running` = Active, `.rested` = Rested), and `inActiveBand` is the one predicate that means Active exactly. Full detail, including why the archived key is `"inactive"` while its label reads Done, in `ARCHITECTURE.md` § Board nomenclature.

# "Shipped" means merged into the primary branch

Never describe a created, opened, or pushed PR as "shipped." An open PR is implemented, pushed,
ready for review, or awaiting merge. Use "shipped" only after the change has actually been merged
into the repository's primary branch. This applies to progress updates, final handoffs, and signal-card
bullets.

# Project-local skills and tools are shared across agents

Any project-local skill or tool lives in ONE agent-neutral copy that every agent configuration
discovers — never a per-agent fork. Skills live canonically in `.agents/skills/<name>/` (Codex
discovers this path natively); `.claude/skills/<name>` is a relative symlink into that canonical copy
so Claude Code discovers the identical content. When adding a skill, create it under
`.agents/skills/` and add the symlink; verified end-to-end 2026-07-21 with `adhoc-cdp` (Claude lists
it through the symlink, `codex exec` resolves it at `.agents/skills/<name>/SKILL.md`) — that skill has
since been split into `frizz-stack` / `headless-browser` / `real-subsystem-harness`, which is also the
shape to copy: **one skill, one job**, cross-linked, rather than a single file every caller must skim. Shared
tooling scripts follow the same rule: one copy in an agent-neutral location (e.g. `scripts/`),
referenced from skills — never duplicated into agent-specific config trees.

**A GLOBAL skill takes the identical shape one level up**: canonical in `~/.agents/skills/<name>/`,
with relative symlinks at `~/.claude/skills/<name>` and `~/.codex/skills/<name>` (both `../../.agents/…`
— `~/.codex/skills` is two levels deep, not one, and getting that wrong yields a dangling link that
still `ls`es fine). A global skill must BUNDLE any script it needs beside its `SKILL.md`; it cannot
reach into a repo's `scripts/`. And a skill belongs in exactly one scope — never a project copy AND a
global copy of the same name, or they drift and the agent lists it twice.

# Use Nub for the Node toolchain

Prefer Nub over direct `node`, `npm`, `npx`, `pnpm`, `yarn`, `tsx`, or `ts-node` commands. Run
JavaScript and TypeScript files with `nub <file>`, package scripts with `nub run <script>`, installed
CLIs with `nubx <tool>`, tests with `nub --test`, and installs with `nub install`. Nub transpiles
TypeScript but does not typecheck it, so keep `tsc --noEmit` and project typecheck gates separate.

# Agent completion invariant

Once spawned, an agent runs to its terminal return. Do not interrupt or cut off an active agent to
reduce churn, reclaim slots or quota, redirect work, respond to a user steer, contain live-server
instability, or hurry completion. Deliver new direction through the agent's message/follow-up path,
then reconcile obsolete or conflicting results after it returns. Mid-turn interruption can leave
partially applied edits, tests, and owned processes behind, making the resulting repository state
unsound. Isolate or restart only the affected unstable service; never stop a writer to stabilize it.
If an agent appears hung or continuing would be dangerous, use the interactive question path to ask the
user. The sole exception is an explicit user instruction that names the interruption.

---
> Source: [colinhacks/frizz](https://github.com/colinhacks/frizz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
