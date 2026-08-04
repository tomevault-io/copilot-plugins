## beets

> An in-memory B+tree. This is a learning project: finding and

# beets

An in-memory B+tree. This is a learning project: finding and
fixing bugs myself is the point.

## Bug reports — do not spoil

When you find a bug, flaw, or gap in this codebase (whether asked to review
or noticed in passing):

- Do NOT tell me what the bug is, where it lives, or how to fix it.
- Tell me only THAT a bug/gap exists, and offer to write tests that
  identify it.
- When writing those tests: each test pins one violated contract. Test
  names, doc comments, and assertion messages should state the expected
  behavior (that's how a failure points at the bug) — but must not narrate
  the root cause or the fix.
- Never fix the bug yourself unless I explicitly ask.

Exceptions:

- If I ask a direct question about a specific hypothesis ("was X the
  problem?", "is this a splitting or a getting problem?"), answer it
  honestly and directly — but don't volunteer details of other bugs I
  haven't asked about.
- Simple compile errors, stale test modules, and other things that block
  building or running tests are not "bugs" in this sense. Report and handle
  those plainly.

## perf.md

`perf.md` tracks the perf workstream: the target workload, ideas tried
with their measured impact, and a comparison table (beets vs std
`BTreeMap` vs `sweep_bptree`). When you run benches, update the table
(and its as-of date) with the fresh numbers.

History and decision records:

- The document records what IS, not what was. Outside the "kept" /
  "tried and thrown away" sections, never write was/now narratives —
  no "the old finding is dead", "remeasured, now flipped", run-vs-run
  drift notes, or comparisons to previous runs. When results change,
  replace the old statement with the current fact; history lives in
  git, not in text.
- The "kept" and "tried and thrown away" sections are the exception:
  they document existing code decisions (what was tried, the measured
  impact, why it stayed or went) and keep their A/B numbers even when
  the benches that produced them are deleted.

---
> Source: [prestwich/beets](https://github.com/prestwich/beets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
