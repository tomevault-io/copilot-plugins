## cognitive-surrender

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A small documentation set about **cognitive surrender** — the behavior, identified by Shaw & Nave (Wharton, 2026), where users adopt AI output without verification and treat it as their own reasoning. The repo delivers three things:

1. A summary of the *You Are Not So Smart* podcast episode covering the research.
2. A pointer to the underlying working paper on SSRN: https://ssrn.com/abstract=6097646
3. Claude system prompts designed to counteract the mechanisms of cognitive surrender (sycophancy, confidence inflation, zero friction, ownership confusion, bypassed deliberation).

There is no code, no build system, and no test suite. The entire deliverable is markdown.

## Document structure and reading order

The numeric filename prefixes enforce a deliberate sequence — diagnosis → principles → practice:

- [01-podcast-summary-cognitive-surrender.md](01-podcast-summary-cognitive-surrender.md) — the phenomenon, the Cognitive Reflection Test experiment, and the Tri-System Theory framing. This is the source-of-truth summary; downstream docs reuse its vocabulary (supernormal releasers, agentic pareidolia, System 3).
- [02-prompt-design-principles.md](02-prompt-design-principles.md) — extracts six countermeasures (kill sycophancy, force calibrated uncertainty, add friction, preserve attribution, invert the default on agreement, refuse to decide for the user). Each principle ties back to a specific mechanism named in **01**.
- [03-ready-to-use-prompts.md](03-ready-to-use-prompts.md) — operationalizes the principles as copy-pasteable prompts: one *general* prompt for Claude's global settings, plus four *project-specific* prompts (writing, coding, research, decisions) that are designed to **stack on top of** the general prompt, not replace it.

When editing one file, check whether the change propagates. Adding a new principle to **02** usually implies a corresponding prompt fragment in **03**; changing terminology in **01** usually needs to flow downstream.

## Editorial voice — non-negotiable

The documents argue against AI sycophancy and ownership confusion. They must sound like they believe it. Preserve:

- **No filler praise or hedging.** No "great question," "it's worth noting," "certainly." The general prompt in **03** explicitly bans this; the prose follows the same rule.
- **Direct attribution.** When a claim comes from Nave, Shaw, McRaney, or Kahneman, name them. Don't launder cited ideas into the ambient voice.
- **Calibrated language.** Nave is described as "cautiously pessimistic" — not "worried" or "alarmed." Match that level of precision rather than smoothing it out.
- **Short paragraphs, concrete examples.** The Dwight/Michael *Office* scene and the beaver/gull examples exist because abstract claims about AI don't land. Keep that pattern for new material.

If an edit would fit comfortably in a generic AI-risk blog post, it probably violates the voice.

## Prompt composability

The prompts in **03** are engineered as a layered system:

- The **general prompt** is tuned to be usable for everyday lookups — it corrects for sycophancy and uncertainty without being obstructive.
- The **project prompts** (writing, coding, research, decisions) intentionally go further on the axis most relevant to that domain — e.g., the research prompt enables Socratic mode, which would be annoying as a global default.

When adding or editing a prompt, don't duplicate directives that the general prompt already covers — the project prompts assume the general prompt is already loaded. If you change a directive in the general prompt, check that none of the project prompts contradict it.

---
> Source: [mmarseglia/cognitive-surrender](https://github.com/mmarseglia/cognitive-surrender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
