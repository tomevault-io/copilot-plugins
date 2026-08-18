## marketing-os

> This repo is one marketing skill: an orchestrator (`skills/marketing-os/SKILL.md`)

# Marketing OS — agent entry point

This repo is one marketing skill: an orchestrator (`skills/marketing-os/SKILL.md`)
plus fourteen reference modules under `skills/marketing-os/references/`. Any agent
host (Claude, Codex, Cursor, Gemini CLI, ...) can use it: read the SKILL.md first,
then load ONLY the module the task needs. Do not load all references at once — the routing exists so a
task costs one module, not fourteen.

**Before any module:** read `brand-context.md` if it exists (project root,
`.claude/`, or `.agents/`). It holds the product, ICP, positioning, proof,
voice and constraints. If absent, proceed, say the output is un-contextualised,
and offer to create it from `skills/marketing-os/brand-context.template.md`.

## Routing

| The user wants to... | Load |
|---|---|
| Audit/score a website, landing page, funnel; "why isn't this converting" | `skills/marketing-os/references/audit.md` + `skills/marketing-os/references/audit-rubric.md` |
| Get cited by ChatGPT/Perplexity/AI Overviews; GEO, AEO, llms.txt | `skills/marketing-os/references/geo.md` + `skills/marketing-os/references/geo-engines.md` |
| Write/rewrite headlines, ads, pages; "make this punchier" | `skills/marketing-os/references/copy.md` + `skills/marketing-os/references/copy-frameworks.md` + `skills/marketing-os/references/slop-patterns.md` |
| Hooks for ads/video; "write me 20 hooks"; thumbstop problems | `skills/marketing-os/references/hooks.md` + `skills/marketing-os/references/paid-ads.md` |
| Diagnose paid ads: CPM up, ROAS down, fatigue, what to test next | `skills/marketing-os/references/paid-ads.md` + `skills/marketing-os/references/ads-diagnostics.md` |
| Email sequences, subject lines, deliverability | `skills/marketing-os/references/email.md` + `skills/marketing-os/references/slop-patterns.md` |
| LinkedIn/X posts, content that doesn't read as AI | `skills/marketing-os/references/social.md` + `skills/marketing-os/references/slop-patterns.md` |
| Launch a product/feature, Product Hunt | `skills/marketing-os/references/launch.md` + `skills/marketing-os/references/copy.md` |
| Positioning, category, offer design, pricing | `skills/marketing-os/references/positioning.md` |
| Competitor teardown | `skills/marketing-os/references/competitive.md` + `skills/marketing-os/references/paid-ads.md` |
| App Store / Google Play listing | `skills/marketing-os/references/app-store.md` + `skills/marketing-os/references/store-specs.md` |
| Read performance data, design a test, "did this work?" | `skills/marketing-os/references/analytics.md` |

Given only a URL with no stated task: run the audit module first, then offer
the modules its findings imply (messaging problems → copy, AI-search gaps →
geo, ad-driven traffic → paid-ads).

Every prose deliverable (copy, email, social) runs `skills/marketing-os/references/slop-patterns.md`
before delivery. No exceptions.

## Standards that hold in any host

- Score everything scoreable; state the rubric and that scores are heuristics.
- Ship artifacts (the rewritten headline, the JSON-LD, the full email), not advice.
- End every report with what you couldn't determine.
- Never invent statistics, testimonials, or customer names — write `[NEED: x]`.
- Never touch live campaigns or handle credentials; diagnose and prescribe only.

---
> Source: [Yuzzyuk/marketing-os](https://github.com/Yuzzyuk/marketing-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
