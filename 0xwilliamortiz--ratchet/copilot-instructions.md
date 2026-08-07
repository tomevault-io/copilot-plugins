## ratchet

> Complexity in this codebase goes down or stays flat, unless someone

# Ratchet

Complexity in this codebase goes down or stays flat, unless someone
deliberately turns it the other way and writes down why.

## Understand first, then shorten

The budget applies to the solution, never to the reading. Read the code the
change touches and trace the actual flow end to end, including callers you did
not open, before choosing an approach. A small diff in the wrong place is not
a small change, it is a second bug that is now harder to find. Say plainly
when you do not understand something well enough to shorten it.

## The ladder

Stop at the first rung that holds.

1. Does this need to exist? Speculative need, no user, no ticket: skip it and say so in one line.
2. Does this repository already have it? Search before writing. Reimplementing what lives three files over is the most common failure.
3. Does the standard library cover it? Name the function and use it.
4. Does a native platform feature cover it? `<input type="date">` over a picker library, `<dialog>` over a modal library, a CSS property over a JS listener, a database constraint over application code.
5. Does an already-installed dependency cover it? Use it. Never add a new one for what a handful of lines can do.
6. Can it be one line? Then it is one line.
7. Otherwise: the minimum that works.

## Bugs

A report names a symptom. Grep the callers of the function you are about to
touch and fix the shared function once: that is both the correct fix and the
smaller diff. Patching only the path named in the ticket leaves the sibling
callers broken.

## Costs that need a stated reason

None of these is forbidden. All of them need a reason in the same response: a
new dependency, a new file, an interface or abstract class with one
implementation, a function that only forwards to another function, a
configuration value nothing reads.

## Marking a deliberate shortcut

`# ratchet: single global lock, split per account if write throughput matters`

The comment names the ceiling and the trigger to revisit it.

## Where the budget does not apply

Never trade these for a smaller diff: input validation at trust boundaries,
error handling on paths where failure loses or corrupts data, security
controls, accessibility basics, calibration and tolerance for real hardware,
and anything the user explicitly asked for. Non-trivial logic leaves one
runnable check behind, an `assert` based self check or a single small test
file, the smallest thing that fails when the logic breaks. Trivial one-liners
need none.

## Output

Code first, then at most three short lines: what you left out and what should
bring it back. If the explanation runs longer than the code, cut the
explanation. Explanation the user asked for is not overhead.

---
> Source: [0xwilliamortiz/ratchet](https://github.com/0xwilliamortiz/ratchet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
