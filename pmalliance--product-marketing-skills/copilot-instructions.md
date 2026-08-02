## product-marketing-skills

> This repo contains six Claude skills for evidence-heavy product marketing work.

# The PMM Signal Pack - Operating Instructions

This repo contains six Claude skills for evidence-heavy product marketing work.
Each reads a large body of source material and returns a structured, sourced
read of it.

## Core rules for every skill in this pack

1. **Read the shared context first.** If `.agents/product-marketing-context.md`
   exists, read it before starting so findings are framed against the user's real
   ICP, competitors, and positioning. If it doesn't exist, ask only for the
   task-specific essentials.

2. **Cite everything.** Every finding must point back to the specific source it
   came from - transcript line, review, page, dataset. If a claim can't be traced
   to the material, it does not appear as a finding; it goes in a short "worth
   checking" note instead.

3. **Separate stated from evidenced.** Report what the material literally says
   and what the pattern across the material shows, and keep them distinct. Don't
   smooth a messy finding into a clean one.

4. **State the volume.** Each report opens with how much material was analyzed
   (deals, calls, surfaces, items) and the time window, so the reader can judge
   how much weight the findings carry.

5. **Recommend Fable, but degrade gracefully.** These jobs are built for Claude
   Fable 5's large context. If Fable isn't available, follow the fallback in each
   skill: batch the corpus, produce partial results, merge in a final pass.

## Output conventions

- Save reports to `.agents/` unless the user's workflow requires another
  location.
- Use American English. Use hyphens, not em dashes.
- Lead with a summary, then the detailed tables, then the so-what.

## Relationship to the Product Marketing Skills Pack

This pack is the synthesis counterpart to the production-focused
[Product Marketing Skills Pack](https://github.com/pmalliance/product-marketing-skills).
They share the `.agents/product-marketing-context.md` file. A common flow is to
run an evidence skill here (e.g. `win-loss-reasons`), then feed its output into a
production skill there (e.g. `messaging-positioning`).

---
> Source: [pmalliance/product-marketing-skills](https://github.com/pmalliance/product-marketing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
