## anti-ai-slop-scriptwriting

> This file is the standing instruction for any agent working in this repo. Codex

# AGENTS.md — Anti-AI-Slop Script Writing

This file is the standing instruction for any agent working in this repo. Codex
reads it automatically. It exists because Codex does not run this repo's hooks:
in Claude Code a `UserPromptSubmit` hook drops the craft catalog in front of the
model every turn, and a `Stop` hook blocks a script that ships without its two
review receipts. Neither fires here, so what those hooks enforce is written down
below instead. Treat it as binding, not as background.

## Assume your creative-strategy knowledge is thin

The base model knows a lot about writing and very little about this craft. So
before you answer anything touching creative strategy — a script, a hook, a
headline, ad copy, an account read — **open the catalog at
`creative-strategy-context/expertise-routing.md` and load the method docs that
would genuinely help.** Be generous about it. Under-retrieval is the failure
mode: an extra read costs a little, a missing one costs a generic answer.

Then speak those docs' vocabulary. A creative or strategic answer whose sources
name no method doc was almost certainly written from memory — rebuild it rather
than shipping it.

Close every substantive answer with a short Sources list naming what actually
shaped it: the method docs you read, the brand files, the live pulls (what and
when). Casual exchanges and quick factual lookups skip it.

## Creative deliverables have no casual path

Any words a customer will read or hear — a script, a hook, a headline, ad copy,
even a quick one someone asked for offhand — route through the skill in
`.codex/skills/`, not through an inline answer. The skill carries the method, the
output contract, and the two gates. An inline reply carrying customer-facing copy
skipped all three.

## The two gates are not optional

Every script ships with two independent reviews already run, and their verdicts
carried verbatim in the output:

1. **Grounding review** — `skill/reviewers/context-grounding-review.md`. Verifies
   the draft was built from the right docs, brand context, and real data pulls;
   that nothing factual was fabricated; and that the methods were applied, not
   just cited. Returns `grounded` or `bounced`.
2. **Voice review** — `skill/reviewers/creative-voice-review.md`. Runs the
   mechanical linter, then judges by ear against the AI-tells doctrine. Returns
   `ships` or `flagged`.

Both run in their own spawned context, on every script, before the user sees
anything. Rules that hold no matter what:

- **Never offer the review as a choice.** "Want me to run it through the
  reviewer?" is itself the failure. Run it.
- **The reviewer cannot be the writer.** Running `scripts/voice-lint.py` yourself
  and reading the output is not the gate — that is the writer grading their own
  paper, which is the exact thing the gates exist to stop.
- **You never write the verdict blocks.** They are the reviewers' returned text,
  verbatim. A script with no Grounding Review and Voice Review blocks did not
  pass the gates and is not finished.
- **A bounce means regenerate, not annotate.** Re-pull and rewrite the affected
  parts, then re-run the gate.

## Where things are

- `skill/scriptwriting/method.md` — the canonical method, shared by both
  harnesses. `strategy.md` and `processes/` sit beside it.
- `skill/reviewers/` — the two gate definitions, shared by both harnesses.
- `creative-strategy-context/` — the craft docs, catalogued in
  `expertise-routing.md`.
- `scripts/voice-lint.py`, `scripts/grounding-check.py` — plain Python, standard
  library only. The reviewers run these; you do not run them in place of a gate.
- `system/parker-tools.md` — what the Parker MCP can pull. If it is not
  connected, say so plainly rather than inventing numbers.
- `.claude/` and `.codex/` — the thin per-harness skill files and, on the Claude
  side, the hooks.

---
> Source: [jimmyslagle/anti-ai-slop-scriptwriting](https://github.com/jimmyslagle/anti-ai-slop-scriptwriting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
