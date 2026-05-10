## patent-prompts

> This repository contains professional patent workflow prompts for AI coding agents. These prompts are designed by a patent attorney for real-world patent work including claims drafting, prosecution, prior art analysis, claim charting, continuation strategy, and portfolio management.

# Patent Prompts — Agent Instructions

This repository contains professional patent workflow prompts for AI coding agents. These prompts are designed by a patent attorney for real-world patent work including claims drafting, prosecution, prior art analysis, claim charting, continuation strategy, and portfolio management.

## Repository Structure

```
pre-filing/          — Prompts for before filing a patent application
  claims-drafting/     Draft claims from an invention disclosure
  detectability-assessment/  Assess infringement detectability
  disclosure-questions/      Generate clarifying questions for a disclosure
  draft-review/              Review a draft for 101 and 112 issues (3 variants)
  prior-art-analysis/        Analyze claims against prior art

prosecution/         — Prompts for patent prosecution
  patent-summarization/  Summarize a patent from its claims
  concept-extraction/    Extract key technical concepts from claims

portfolio/           — Prompts for portfolio management
  continuation-targeted/   Draft continuation claims targeting a product
  continuation-broadened/  Generate broader continuation claims
  continuation-unclaimed/  Find disclosed but unclaimed subject matter
  claim-chart/             Map claims to product features
  categorization/          Categorize patents by technology area
  pruning-analysis/        Evaluate maintain vs. abandon decisions

examples/            — Sample data and pre-filled example prompts
skills/              — SKILL.md files for Claude Code / agent skill installation
```

## How to Use These Prompts

Each prompt lives in a `prompt.md` file within its category directory. The prompts use `{{PLACEHOLDER}}` format for user inputs (e.g., `{{PATENT_CLAIMS}}`, `{{INVENTION_DESCRIPTION}}`).

To use a prompt:
1. Read the relevant `prompt.md` file
2. Extract the prompt section between the `---` delimiters
3. Replace all `{{PLACEHOLDER}}` values with the user's content
4. Execute the completed prompt

Each `prompt.md` file contains a **Placeholders** table documenting what each placeholder expects.

## Important Notes

- These prompts are for **US patent practice** (USPTO, 35 U.S.C., MPEP). Adjust for other jurisdictions.
- Prompts marked with web search capability work best when you can search the internet for prior art or product details.
- All outputs are analytical aids, not final legal work product. Always note this in your responses.
- Use full patent text (complete claims, full specification) for best results — truncated inputs produce lower-quality output.

## Testing

Use the sample data in `examples/sample-patent.md` and `examples/sample-disclosure.md` to test prompts. Pre-filled examples are in `examples/try-*.md`.

---
> Source: [arcprime-ip/patent-prompts](https://github.com/arcprime-ip/patent-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
