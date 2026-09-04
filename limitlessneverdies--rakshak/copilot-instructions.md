## rakshak

> Instructions for any AI assistant editing this repository. They exist because a

# Working on Rakshak

Instructions for any AI assistant editing this repository. They exist because a
wrong answer here is not a bug report, it is somebody paying Rs 1,70,000 to an
agent who was never licensed.

## The one rule that everything else follows from

**The model transcribes. The code judges.**

A language model may only fill in an `Extraction` (see `lib/types.ts`): a flat,
purely descriptive record of what a document says. Every legal conclusion is
computed by pure functions in `lib/rules.ts`.

Concretely, do not:

- ask a model whether something is legal, fair, excessive, risky or advisable
- let a model produce a verdict, a score, a severity, or a recommendation
- add a rule that calls a model, reads the clock, reads the network, or reads
  `process.env`
- move a legal threshold out of `lib/reference.ts` and into a prompt

`tests/rules.test.ts` greps `lib/rules.ts` for `fetch(`, `Date.now`,
`new Date`, `Math.random`, `process.env` and the model SDKs, and fails if any
appear. If that test is in your way, the design is in your way. Stop and ask.

## Why it is built this way

- **Reproducible.** Two people photographing the same contract must get the same
  verdict. Pure functions guarantee it; a model does not.
- **Defensible.** Every finding cites a specific provision. A worker can hand
  the generated complaint to an officer at the Department of Foreign Employment
  and the reasoning survives being questioned.
- **Correctable.** When the model is wrong, it is wrong about what the paper
  says, which is visible on screen and fixable on the manual-entry form. A
  wrong *opinion* is invisible.

## Rules for changing legal content

1. Every rule in `lib/rules.ts` carries a `legalBasis` string. It must name the
   actual instrument, e.g. the Foreign Employment Act 2064 section 7, or the
   free-visa-free-ticket directive of July 2015. Never paraphrase it into
   something vaguer, and never invent a section number.
2. Do not add a threshold you cannot cite. `WAGE_FLOORS` in
   `lib/reference.ts` ships with every value `null` and `verified: false`, and
   rule R6 therefore *skips* rather than guessing. That is correct. Fill it in
   from the gazetted minimum wage for each destination, or leave it alone.
3. The Rs 10,000 service-charge ceiling is under active policy review. If it
   changes, change `MAX_SERVICE_CHARGE_NPR` and the affected `legalBasis`
   strings together, and update the tests that assert the money figures.
4. `AGENCY_REGISTRY` is a partial snapshot with a `snapshotDate`. Three records
   are invented and marked `fixture: true` with `[DEMO]` in the name. Never
   remove that marker, and never attach a fabricated violation to a real
   company's name.

## Rules for the interface

- Nepali first, English second. Every user-visible string exists in both. If you
  add a string to the `UI` object in `demo/template.html`, add both halves.
- Never ship a screen that says "no problems found" when checks were skipped.
  Show what could not be checked. `Assessment.skipped` exists for this.
- The app never says "this is a scam" or "this is legal". It says which
  provision the document appears to conflict with, and points to the Department
  and to free legal help.
- Contrast: every text colour must clear 4.5:1 against its background. This is
  read outdoors on cracked phone screens. `qa/layout.mjs` enforces it.
- Tap targets: 40px minimum. Also enforced by `qa/layout.mjs`.
- Devanagari conjuncts break when a font with partial coverage wins the
  fallback, so Devanagari faces sit ahead of `system-ui` in `--sans`. Do not
  reorder them. `build-demo.mjs` refuses to build if a U+FFFD appears anywhere.

## Before you say you are done

```bash
npm run verify
```

That runs the engine typecheck, both test suites, the demo build, the layout
audit and the browser screen tests. All five must pass. Do not report success
from reading the code: run it.

## Things that look like improvements and are not

- Replacing the rules engine with "just ask the model, it is smarter now".
  It is, and it still cannot be cross-examined by a labour officer.
- Guessing a wage floor so R6 stops skipping.
- Making the model output the Nepali explanation text. The explanations are
  written once, reviewed, and reused, so the wording that reaches a worker is
  wording a person approved.
- Widening the validator to accept whatever the model sent. If the model's
  answer is not a valid `Extraction`, the manual-entry form is the fallback.

---
> Source: [limitlessneverdies/rakshak](https://github.com/limitlessneverdies/rakshak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
