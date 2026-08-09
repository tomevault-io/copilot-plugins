## lean-interact

> This file tells the assistant how to work in `lean-interact`. It is not background reading;

# CLAUDE.md — operating instructions for this repository

This file tells the assistant how to work in `lean-interact`. It is not background reading;
it is the procedure. Follow it literally.

The short version: **the user states mathematics, you produce compiled Lean, fast, and you
never assert a Mathlib name you have not verified.** Everything below serves those three
things.

---

## 0. Start of every session

Before responding to the first mathematical request, read, in this order:

1. **`MEMORY.md`** — durable facts and preferences. Non-negotiable context. It tells you who
   the user is, what the environment guarantees, and which rules have been established. If
   its template sections are still unfilled, assume the default this project was built for: a
   working mathematician who knows the mathematics and is learning Lean and Mathlib. Then ask
   the user to fill those sections in, once, and record the answers.
2. **`vault/Formalization Style.md`** — how Lean is written here: naming, statement shape,
   tactic style, `ℕ` vs `ℤ` vs `ZMod n`. Entries marked PROVISIONAL are the project's guess,
   not the user's stated wish; follow them, but never defend them. If the user objects, the
   user is right and the note gets updated.

Then skim the top entry of `JOURNAL.md` for where the last session stopped.

Do not skip this because the request looks simple. The whole point of those files is that
they contain decisions you would otherwise re-litigate or silently contradict.

If a live session is not already running, start it: `tools/session.sh` (see `--help`).

Five skills package the recurring procedures, and you should use them rather than
improvising: **`install`** (bring a fresh clone to a working state — the precondition for all
the others), **`formalize`** (prose in, theorem out), **`formalize-from-magma`** (computation
in, theorem out), **`mathlib-lookup`** (find and verify a name), **`lean-session`** (bring up
or repair the live views).

---

## 1. The formalization loop

The user types a sentence of ordinary mathematics. Your job is the following sequence, every
time.

**Step 1 — Restate the sentence precisely.**
Before any Lean, say back what you understood, in mathematics, with every quantifier,
hypothesis and side condition made explicit. Ordinary prose leaves things implicit ("let `p`
be prime" often silently means odd; "`a/b`" often silently means `b` divides `a`). Surface
those choices, and when it matters, ask. A formalization of the wrong statement is worse than
none, because it looks like progress.

**Step 2 — Write the informal record.**
Put the user's sentence and its LaTeX form into `Scratch/Current.md`. This is what lets
anyone later check the Lean against the intent, and it is what the live views display beside
the code.

**Step 3 — Find and verify every name you will need.** See section 2. Do this *before*
writing the proof, not after the error.

**Step 4 — Write the Lean.**
Statement and proof into `Scratch/Current.lean`. Import **`NtLean.Preamble`** (section 3).
Remember `autoImplicit = false` and `relaxedAutoImplicit = false`: **every binder must be
declared explicitly**, type variables included. A missing binder is a hard error here, not a
convenience.

**Step 5 — Let the watcher compile.**
Saving is enough: `tools/live.py` polls the mtime, runs Lean through the daemon, and writes
`.live/status.json`. Read the result from that file rather than running Lean yourself while
the watcher is up. To check a fragment without disturbing the live file, use
`tools/leanserver.py snippet` — it goes through the warm daemon; `tools/leanlib.py`'s
`lean_snippet` always takes the slow one-shot path.

**Step 6 — Iterate to zero.**
Keep going until `state` is `"ok"`: **zero errors and zero sorries**. A `sorry` is a
legitimate intermediate step, and a legitimate way to show the user exactly where the
difficulty sits, but it is never an endpoint and never something left behind quietly. If a
`sorry` survives to the end of a turn, say so in the first sentence of your reply.

**Step 7 — Promote.**
Move the result into the right module under `NtLean/`, add the import to `NtLean.lean` if the
module is new, and run `lake build` to confirm it integrates. Promotion is the only moment
`lake build` is used. **After it, restart the daemon** (section 3) — otherwise the next check
of `Scratch/Current.lean` will not see the new declaration.

**Step 8 — Write the vault note.**
One note per promoted result, from the template in `vault/Templates/`. It carries the
informal sentence, the LaTeX, the final Lean, the lemmas used, and `[[wikilinks]]` to related
notes. Navigation between notes is a goal of the vault, so the links matter; a dead one is
invisible in Obsidian until someone clicks it, which is why `tools/check_names.py` also
checks them.

**Step 9 — Append to the journal.**
One bullet in the top section of `JOURNAL.md`: what was proved, in which module, and why it
mattered. Create a section for today's date if there is none. Failed attempts get a bullet
too, with the reason — that is usually the most valuable line in the entry.

### Turn length, and deferred bookkeeping

This is a live, conversational instrument. **A turn should take seconds.** Steps 1 to 6 are
the turn: show the compiled Lean and the short teaching note, then stop and let the user
react.

Steps 7 to 9 — promotion, the vault note, the journal bullet, index updates, commits — are
**bookkeeping, and are deferred by default**. Do not perform them mid-flow. Keep a short
running list of what is outstanding, mention it in one line ("deferred: promote
`foo`, vault note, journal"), and flush it when the user pauses, changes topic, or asks.

When thoroughness and responsiveness conflict here, responsiveness wins and the thorough part
is deferred rather than dropped. Offer choices instead of performing them, and do not batch
several results into one turn unless asked.

### When the proof is the interesting part

For a proof the user wants to build rather than receive, use **`tools/step.py`**: an
interactive one-tactic-at-a-time driver over `Scratch/Current.lean`.

- `tools/step.py init "<statement>"` opens a statement, with a trailing `sorry` as the cursor.
- `tools/step.py step "<tactic>"` applies one tactic and prints the **new goal state**. If the
  tactic fails, the file is restored and Lean's error is printed, so a failed attempt costs
  nothing and leaves no mess. If it closes the goal, the `sorry` is removed.
- `goal`, `undo`, `show` do what they say; history survives across invocations.

This is the mode to use when the user says "induct on `n`" and wants to see what that leaves
to prove, rather than being handed a finished proof.

---

## 2. Absolute rule: never write an unverified Mathlib name

**Do not write a Mathlib lemma, theorem, definition or instance name into a proof, into
prose, or into a suggestion, until you have verified that it exists.**

Find candidates with `tools/mlq.py` — free text, `--name`, `--about`, `--namespace`,
`--concept`; run `--help` for the full interface. Then verify with one of:

- `tools/leanserver.py verify NAME...` — asks the compiler and prints the **real elaborated
  types**, warm, in well under a second (17 names in ~0.6 s). This is the one to reach for.
- `tools/mlq.py --verify NAME...` — the same check from the search tool. It swallows every
  following argument, so put it last on the command line.
- A `#check NAME` line in `Scratch/Current.lean` — checked by the same Lean and Mathlib that
  will elaborate the proof, and therefore the final authority when anything disagrees.

**The false-negative caveat, which matters.** Verification elaborates against
`NtLean.Preamble`, so a genuine Mathlib name in a module the preamble does not import is
reported ABSENT. Before you tell the user a name does not exist, or record an absence in the
vault, in `book/`, or in `tools/concept_aliases.json`, re-check with `verify --full`
(`tools/mlq.py --verify-full`), which elaborates against all of Mathlib: slow, roughly two
minutes in a separate process, but authoritative.

**Why this is absolute rather than a best practice.** While this repository was built, **353
Mathlib names were written from memory and 27 of them did not exist** — roughly one in ten —
along with 18 stale module paths. A plausible invented name reads as authoritative, costs a
compile cycle to disprove, and quietly teaches the user something false; and if the user's
stated pain point is that they do not remember Mathlib names, they cannot catch it. "I need
to look this one up" is always an acceptable answer. Guessing is not. Hedged guessing ("I
think it might be…") is not either, because the hedge evaporates the moment the name is
repeated back.

The same rule covers lemma **signatures**: do not describe the exact hypotheses or the
argument order of a Mathlib lemma from memory. Check them; `verify` prints the real type.

**The standing invariant.** Every Mathlib name cited anywhere in `NtLean/`, `vault/` or
`book/` must either exist, or carry the marker `(absent at Mathlib v4.28.0)` on the same line,
so a reader meets the warning where the name is used. `python3 tools/check_names.py` enforces
this and exits non-zero otherwise. Run it before committing documentation and after any
Mathlib bump.

---

## 3. Which Lean command to run

- **Imports: `import NtLean.Preamble`, never `import Mathlib`** in a working file. Measured:
  `import Mathlib` costs **~111 s per check** (5.4 GB of oleans against 16 GB of RAM, so they
  never stay in the page cache) versus **~24 s** one-shot, **~0.25 s** warm, for the preamble.
  This single fact is why the project is usable at all.
- **Need a module the preamble lacks?** Add the import to `NtLean/Preamble.lean`, run
  `lake build NtLean.Preamble`, then `python3 tools/leanserver.py restart`.
- **Read a check with `--status`, never the raw JSON.**
  `python3 tools/leanlib.py Scratch/Current.lean --status` prints one line, about 80
  bytes on a clean file plus one line per error, and still flags a drifted sidecar. The
  default JSON embeds the whole source in its `code` field and runs to roughly 26 kB.
  It is the single largest avoidable output in this repository; everything else here
  already reports in tens of bytes.
- **Feedback loop: the warm daemon.** `tools/leanlib.py::lean_check_file` routes through
  `tools/leanserver.py`, which keeps one `lake env lean --server` process alive with the
  environment already elaborated and pushes each edit as an LSP `didChange`. Measured
  **~0.25 s** per edit. If the daemon is down it falls back to a one-shot `lake env lean`
  (tens of seconds): correct, but not live. Start it with `tools/session.sh`, or
  `python3 tools/leanserver.py check Scratch/Current.lean`.
- **First check of a file is slow, once.** Lean runs **one worker per open document**, so the
  first check of each file pays the import load (**~40 s**). `tools/session.sh` pre-warms
  `Scratch/Current.lean` for exactly this reason. A slow first check is expected and should
  be explained rather than debugged; a slow *second* one is a bug.
- **Restart the daemon after any `lake build` that touches a module `Scratch/Current.lean`
  imports.** The daemon loads imports once at worker startup, so without a restart it reports
  `unknown identifier` for a lemma that is plainly there. This has cost real time; do not
  rediscover it.
- **`lake build` is for promotion only.** Reserve it for the moment a result moves into
  `NtLean/` and genuinely has to become part of the library.
- **Never** start a Mathlib build from source. If the olean cache misses, stop and say so;
  the matched pin exists to prevent exactly that, and `lake exe cache get` is the repair.

---

## 4. Teaching contract

After every result, without being asked, give:

1. **The Mathlib lemmas used, and why.** Not just the names — say what each one does and why
   it was the right tool for that step. Include the naming logic when it is instructive
   (`Nat.ModEq.add_right` reads as "ModEq, adding on the right"), because the naming grammar
   is how the user will find the next lemma without you.
2. **Exactly one alternative tactic worth knowing.** One. Show how the same step could have
   gone with `omega`, or `decide`, or `simp [...]`, or `exact?`, or `gcongr`, and say when
   that alternative would be the better choice. A survey of five tactics teaches nothing; one,
   with the conditions under which you would reach for it, sticks.

Keep both short. Unless `MEMORY.md` says otherwise, spend the words on Lean and Mathlib and
none on explaining the mathematics — the default assumption is that the user knows it.

---

## 5. Recording preferences

Whenever the user expresses a preference — about naming, statement style, tactic choice,
verbosity, workflow, anything, even in passing, even as a complaint — record it immediately,
in both places:

- **`vault/Formalization Style.md`** — the full version, when it is about how Lean code looks
  or how statements are phrased. If it confirms or overrules something marked PROVISIONAL
  there, edit that default in place and mark it confirmed, with the date.
- **`MEMORY.md`**, in the append-only "Preferences learned" section — a dated bullet in the
  documented format. **Append only**: never edit or delete an existing bullet; if a preference
  changes, add a new bullet saying what it supersedes.

Do this in the same turn the preference is expressed, not "later". A preference that is not
written down will be violated next session, and the user will have to say it again — which is
the specific failure this whole file exists to prevent.

---

## 6. Magma is optional, and remote

Magma is commercial and usually lives on a departmental server, so `tools/magma_run.sh`
drives it over SSH. It is configured by copying `config.example.sh` to `config.sh`
(gitignored) and setting `LEAN_MAGMA_HOST` and `LEAN_MAGMA_BIN`.

- **Check before you rely on it.** With nothing configured, `tools/magma_run.sh` prints an
  actionable message; relay that rather than working around it. **The Lean side of this
  project needs none of it** — never block a formalization on Magma, and never suggest
  installing Magma locally.
- **Always go through `tools/magma_run.sh`** (see `--help`). It handles the two things that
  break naive invocations: the binary is often on the `PATH` of a **login** shell only (hence
  `ssh $LEAN_MAGMA_HOST 'bash -lc "..."'`), and Magma prints a startup banner ending in a row
  of asterisks plus a session-log path, which must be stripped before the real output.
- **What it is for:** computing the examples and tables that motivate or refute a claim —
  orders, residue counts, Legendre symbols, small-case searches. Testing a claim at `n = 12`
  before spending an hour proving it is an excellent trade, and finding out that it *fails* at
  `n = 12` is better still. Keep the cleaned output in the vault note as the evidence behind
  the statement.

---

## 7. Repository hygiene

- **Never commit** `.lake/`, `.live/`, `book/_build/`, `_build/`, `__pycache__/`, `config.sh`,
  `.claude/settings.local.json`, or any `.olean` / `.ilean` file. `.gitignore` covers these;
  do not add exceptions.
- **`lake update` is denied** in `.claude/settings.json`, deliberately: it rewrites the
  committed `lake-manifest.json` pin and a bad resolution there is what causes a from-source
  Mathlib build. Show the user the command and let them run it.
- **Do commit** `lake-manifest.json` (it pins the exact Mathlib revision and is what makes the
  build reproducible) and `.vscode/settings.json` (project configuration).
- **Nothing machine-specific, ever**, in a tracked file: no hostnames, no usernames, no
  absolute paths under a home directory. Machine-specific settings go in `config.sh`, which is
  gitignored, and are documented as placeholders in `config.example.sh`.
- All runtime tooling is **Python 3 standard library only**. No pip installs, no third-party
  imports, not even a small one. Shell is **bash**, and must stay macOS/BSD compatible: no
  `readlink -f`, no GNU-only flags, no `sed -i` without a backup suffix.
- Every script in `tools/` is executable, has a `--help`, and carries a one-line usage comment
  at the top. Match that when adding one.
- No emoji, in code, documentation or output.

---

## 8. Style of your replies

- Restate before you formalize. Never formalize a sentence you have silently reinterpreted.
- Show the Lean, not a description of the Lean.
- Report failure plainly: which line, which error, what you think it means, what you will try
  next. A failed elaboration is information, not an embarrassment to smooth over.
- When you do not know a Mathlib name, say you are looking it up — and look it up.
- Prefer small, clear, commented code to clever code, in Lean and in Python alike. This is a
  research instrument that the user reads and modifies.
- Write for a working mathematician who is learning Lean. Explain the formalization, not the
  mathematics.

---
> Source: [nasqret/lean-interact](https://github.com/nasqret/lean-interact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
