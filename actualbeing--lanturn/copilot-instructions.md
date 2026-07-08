## lanturn

> A positive news publication. Real events, verified sources, actionable CTAs. No fabrication, no corporate bias.

# Lanturn Weekly

A positive news publication. Real events, verified sources, actionable CTAs. No fabrication, no corporate bias.

## Project Structure

- Hugo static site deployed on Cloudflare Pages
- Content in `content/posts/` as markdown files
- Published automatically when pushed to `main` branch

## Editorial Format

**Monday Digest** (`YYYY-MM-DD-digest.md`): 3 stories, 400-700 words each. Each story has:
- Factual reporting with verified sources
- Context that helps readers understand *why* it matters to their daily lives
- A concrete, positive call to action

**Thursday Glow** (`YYYY-MM-DD-glow.md`): 800-1200 word deep dive expanding the strongest CTA from Monday's digest.

## Writing Standards

- Every claim must be traceable to a named source with a URL
- Use relatable comparisons for large numbers (not "1,162 terawatt-hours" — instead "enough to power 108 million homes" or "equivalent to 300 Hoover Dams")
- Avoid jargon-heavy phrasing. If "The EIA projects developers" could be misread, rewrite it
- Substantial depth — 5 sentences is worthless. Readers should actually learn something
- Warm, conversational, but not condescending. Like explaining something to a smart friend who hasn't been following the news
- Positive framing without being naive — acknowledge catches and challenges honestly
- No links to lead-gen or mildly predatory aggregator sites (e.g., EnergySage). Prefer official sources, government sites, or "search for X near you" guidance
- No corporate PR dressed as news

## Editorial Preferences (learned from selections)

- Prefers hard data over anecdote (EIA federal data, peer-reviewed research)
- Values stories that show AI positively ("People need some help trusting AI")
- Drawn to practical wallet-impact stories (childcare costs, energy bills)
- Cross-partisan appeal — shared stakes, not tribal takes
- Merges related stories into richer narratives (e.g., AI magnets + rare earth recycling = one supply chain story)

## Frontmatter Format

```yaml
---
title: "Lanturn Weekly: [Tagline]"  # or "Lanturn Glow: [Title]"
date: YYYY-MM-DDT03:00:00-06:00     # Always 3 AM CST
description: "50-70 words for digest, 30-50 for glow"
tags: ["weekly-digest", "topic1", "topic2"]  # or ["glow", "topic"]
sources:
  - url: "https://..."
    publication: "Source Name"
---
```

## Workflow Commands

Use the `./lanturn` script:
- `./lanturn research` — Search for news, produce candidate stories
- `./lanturn draft` — Draft articles from approved candidates
- `./lanturn publish` — Commit and push to deploy

## Confidence Flags

When drafting, mark each story:
- 🟢 All facts verified against named sources
- 🟡 Some details from single source only — needs cross-reference
- 🔴 Contains claims that couldn't be independently verified — flag for review

---
> Source: [actualbeing/lanturn](https://github.com/actualbeing/lanturn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
