## catalyst-integrity-desk

> `CLAUDE.md` sits beside this file and is addressed to a different tool. Where the

# AGENTS.md — for IBM Bob

`CLAUDE.md` sits beside this file and is addressed to a different tool. Where the
two disagree about who builds what, this file governs your work.

**You are the primary development tool on this project.** The build prompts in
`docs/BOB_PROMPTS.md` are addressed to you. Claude verifies your output, runs the
kill gates, and writes docs; it does not execute those prompts.

Read `HANDOFF.md` first, then `docs/SPEC.md`. Read `docs/FINDINGS.md` before
touching any extraction code: it records eight failure modes found empirically,
and every one of them produces a wrong number rather than an error.

## What this is

Every biotech thesis rests on a date. The company has cash into X, the trial
reads out before X, therefore they are funded to the catalyst. The left side
comes from SEC filings. The right side comes from ClinicalTrials.gov, where the
sponsor sets it, can revise it whenever it likes, and nothing reconciles that
revision against the thesis that depended on the old date.

This project treats that as a contract and audits it.

## The rule everything follows

**Python computes. Granite judges prose. Humans decide.**

No model-produced number reaches the user. `_fabricated()` in the ported
`granite.py` enforces it and is ported byte for byte:

- Do **not** tighten it to ban all digits. That rejects correct output quoting a
  figure from the belief's own claim text, which is quotation, not invention.
- Do **not** loosen it. The rule is "a number absent from the input".

## Credentials

Granite needs `WATSONX_API_KEY`, `WATSONX_PROJECT_ID` and `WATSONX_URL`. They
live in `.env` at the repo root, which is gitignored and which you cannot read,
because "Respect .gitignore" is on. You do not need to read it. You need to load
it, in the same shell command as whatever you are running:

```bash
set -a; . ./.env; set +a; python3 -m pytest tests/
```

A shell that has not sourced `.env` has no credentials, so `GraniteClassifier`
cannot authenticate and anything requiring live Granite fails. That failure is
correct behaviour, not a bug to work around: silently falling back to the stub
is how a stub judgment ends up in a demo that claims Granite made it.

Never print the values, never paste them into a message, and never write them
into a file that is not `.env`.

## Do not rewrite the engine

`engine/runway.py`, `engine/ctgov_history.py` and `engine/gap.py` predate you and
are verified against live SEC and ClinicalTrials.gov data. They are the reason
this project is credible. Run all three before and after any change:

```bash
python3 -m engine.runway
python3 -m engine.ctgov_history
python3 -m engine.gap
```

If a change of yours needs one of them modified, stop and say so rather than
editing it. That has been the right call every time it has come up.

## Four invariants

**Every displayed number traces to a named XBRL tag or a specific registry
version.** `Runway.provenance` exists for this. A figure that cannot name its
source does not get displayed.

**Unreliable rows are shown, never ranked, never silently dropped.** A screen
that hides its hard cases is worse than one that shows them. See
`Runway.reliable`. Arrowhead computes to a 1,116 month runway in this engine
because of a partnership upfront; the answer is to flag it, not to hide it.

**A lapsed completion date is never a catalyst.** You cannot run out of money
before an event that already happened. The binding catalyst is the nearest
registered primary completion still in the future. Lapsed dates are retained and
surfaced as date-integrity signals, which is the entire point of the project.

**Burn is a band, not a point.** One quarter containing a partnership upfront is
unrepresentative, and a single number would hide that.

## Language

Say **registered primary-completion expectation**, never "readout date". They
differ systematically by two to four months and the gap is always optimistic.

Say "we could not find", never "nobody has". A commercial screener and a
published revision dataset both exist and are cited in `docs/DEMO.md`. The claim
is the monitor, not a novel dataset and not a finding.

## Voice and commits

Comments say why, not what. No em dashes. Plain language, no filler. Commit as
kristenharim with plain messages, and never add AI attribution, `Co-Authored-By`,
or generated-with footers to any commit or file.

## Never touch

`~/projects/deliberate-risk-desk` is a separate working project and the fallback
demo for this submission. Phase 2 copies four files out of it, listed in
`docs/PORT.md`. Read those. Change nothing there, ever.

## Log what you build

Add a row to `docs/BOB_LOG.md` for each task, naming the tool. The README's Bob
section is written from that file rather than from memory, so a line that says
"by hand" must never be described later as yours.

**You are IBM Bob. Write `IBM Bob` in the Tool column, and nothing else.** Do not
copy the value from the row above and do not name the model you believe you are.
This has already gone wrong twice: you wrote `Claude Code` for your own work on
the redline loop and again on the console, which understated your contribution on
the one requirement the submission cannot lose. In the Transcript column name the
export file if one exists yet, otherwise write `pending export`.

---
> Source: [kristenharim/catalyst-integrity-desk](https://github.com/kristenharim/catalyst-integrity-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
