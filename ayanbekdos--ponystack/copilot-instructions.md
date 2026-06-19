## ponystack

> The membrane between an expensive brain and cheap hands.

# ponystack - agent rules

The membrane between an expensive brain and cheap hands.

**Boil the lake to understand it. Don't pour it into your code.**
Think expensive, write cheap.

Work has phases. Let the right discipline drive each one:

- **understand / plan / design / research** - boil the lake. Think wide and
  deep: decompose, research, weigh trade-offs. Do not write production code yet.
  Spend reasoning freely; understanding is the cheap lake.
- **implement** - the lazy senior takes over. Climb the ladder, stop at the
  first rung that holds: 1) does it need to exist (YAGNI) 2) stdlib 3) native
  platform feature 4) already-installed dependency 5) one line 6) the minimum
  that works. No unrequested abstractions, no speculative scaffolding, fewest
  files, shortest working diff. Mark deliberate shortcuts with a `ponystack:`
  comment naming the upgrade path.
- **review** - run both critics: correctness/security/perf bugs, AND a
  delete-list of everything that does not earn its lines, ending
  `net: -N lines possible.`

Never simplify away: trust-boundary input validation, error handling that
prevents data loss, security, accessibility, anything explicitly requested.
Non-trivial logic leaves ONE runnable check behind. Trivial one-liners need no
test.

(Yes, this applies to agents working on the ponystack repo itself. Especially
to them.)

---
> Source: [AyanbekDos/ponystack](https://github.com/AyanbekDos/ponystack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
