## open-bfme-1

> Several agents push to `origin/master` continuously. Keep each change small,

# Contributing

Several agents push to `origin/master` continuously. Keep each change small,
verified, and easy to rebase. `docs/matching.md` covers byte matching;
`docs/structural.md` covers manual RE. Prose trimmed from these docs lives in
git history — one `git show` away.

## Work selection

An explicit user request or assigned lane overrides the queue. Otherwise:

1. `git pull --rebase origin master`
2. `python3 tools/check_csv.py` — repair ledger errors before other work
3. `python3 tools/list_naked_candidates.py Code` — serves a byte-true dump from
   `Code/gen_asm/`, boundary already proven. **Converting dumps to real C++ is
   the default work.**
4. `python3 tools/next_work.py` for identity/structural work; its output
   explains its tiers.

A tier reporting zero candidates is exhausted, not broken. Regenerate:
`tools/drift_classify.py`, `tools/anchor_unclaimed.py`, `./build.sh`.

Finish or revert each body before starting the next.

## Work the file, not the row

`next_work.py` lists every other queued candidate in the same source file.
**That file is your unit of work** — drain it before asking for another
candidate. Measured land rate: 19.5% for solo candidates, 46.5% with ten or
more siblings landed together. The layout, offsets, and callee pins from the
first body are what the next body needs; a shared header edit costs a full
gate, so edit every dependent body first and pay it once.

## Convert, verify, commit, push — per body

1. Make the smallest source and ledger change for one function.
2. `./build.sh <file-or-symbol>`. If a command returns a process or session ID,
   poll it to completion; never launch a duplicate long build.
3. Stage explicit paths only: `git add <specific-paths>`. Never `git add .` or
   `git add -A`. Check every new ledger source is tracked.
4. Commit normally. **Never bypass hooks.**
5. `git pull --rebase origin master`, `git push`, then pull --rebase again. On
   rejection: rebase, recheck the ledger, retry the push, final pull.

Header, vendored-reference, and shared-shim edits — and a manually resolved
merge — trigger the long full gate in the hook; poll it, don't relaunch. Never
filter a gate through a pipeline that hides its exit status.

Before pushing, `python3 tools/progress.py origin/master` prints what your
session added. `+0.00 pp` is the common outcome — do not stop there; take
another body from the file you were served. If the file is exhausted, say so
in your final message with the figure and the reason.

## Anti-lift policy

Clean C++ is preferred. MASM or inline asm only for a proven codegen blocker
(compiler machinery, x87 shape, SEH). Lifting a dump into a
`__declspec(naked)`/`__emit` .cpp is **not** a conversion — an `__emit` spray
of retail bytes byte-matches by construction, scores +0, and deletes the body
the next converter needed. The naked body must be **gone**, replaced by real
C++. `tools/conversion_gate.py` enforces this in both the commit and push
hooks; the push hook scans your whole outgoing range, so a blocked push may
name a historical lift, not your commit — rebase past it or revert the lift.
Never `--no-verify`.

Ghidra boundaries, xrefs, and vtables are identity evidence; decompiled C is
not byte-match proof. After several failed shapes or ~30 minutes without byte
progress, revert and take a fresh candidate — never leave a nonmatching
reconstruction in the tree. Record the verdict:
`python3 tools/re_log.py record <symbol> <rva> <size> <status> <evidence>`
(never hand-edit `reverse/re_attempts.log`); cite the real boundary and
include `t=<minutes>` and your model.

## Placement and integrity

- Game source under `Code/`; MASM dumps in `Code/masm_dumps/`; scratch
  untracked under `build/`.
- Prefer TU-scoped shims over shared-header edits.
- Progress = `matched` `reverse/functions.csv` rows backed by real source and
  byte verification. Markers and prose are not progress.
- **Landing a `reverse/symbols.csv` pin?** It is an ADDITIVE candidate list —
  the resolver keeps the first pinned address that reproduces retail, so a pin
  naming the *wrong* function still byte-matches and a green gate proves
  nothing about it. `tools/pin_consistency.py --symbol <name>` before you pin,
  `--check` (1.6s) after; both hooks and the full gate run it.
  `reverse/pin_consistency_baseline.csv` is the known-bad backlog and may only
  shrink — never add a line to get green. See `docs/lessons.md`, "A
  `pinharvest` row in symbols.csv is a candidate, not an address".
- No fallback paths; they conceal mismatches.
- Never load `reverse/functions.csv`, `ghidra_functions.csv`, or `exports.csv`
  wholesale; use `rg` or narrow Python filters.
- Preserve unrelated dirty-tree work; revert only the failed attempt.

## Generated claims

`gen-*` rows (`Code/gen_small/`, `Code/gen_asm/`) are byte-true placeholders,
not progress. When you recover a real identity, write the clean C++ in its
proper `Code/` source and repoint the row:
`tools/add_match.py <real-name> <rva> <size> <source> --replace-rva <rva>` for
`gen_asm` dumps (`--replace-existing` when the name is unchanged, e.g.
`gen_small`). `check_csv` rejects a gen-* row sharing a range with a
real-name row — the placeholder yields.

**Never edit a file under `Code/gen_asm/`** — repoint the ledger row and leave
the orphaned `PROC`; that is what keeps converters conflict-free there.
`Code/gen_small/uw_gen_*.cpp` is owned end to end by
`python3 tools/gen_uw.py land` — never hand-edit or append, and never infer a
funclet's `parent=` from adjacency; a guessed parent is invented identity.

After landing a batch, sweep your own new rows: one body per address. A
duplicate range among them is an over-claim to retract, not an ICF alias.

## Vendored third-party claims

`vendored=<lib>-<ver>` rows carry the upstream's real identities, never
`gen-` prefixes; the file's header comment names the exact release. Library
sources live at their official BFME paths. Pristine C TUs compile against
`reference/shims/gamespy/`, never a real Platform SDK.

---
> Source: [Open-BFME/Open-BFME-1](https://github.com/Open-BFME/Open-BFME-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
