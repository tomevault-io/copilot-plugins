## content-flywheel-skills

> SEO & GEO content flywheel — research a topic, plan it, draft it, de-AI it, run a GEO/SEO + CORE-EEAT quality gate, fan out to social, publish, and feed analytics back into the next cycle. Zero-dependency Markdown skill for any AI agent.


# Content Flywheel

A repeatable, agent-driven content engine. One **run** produces a complete topic cluster: a primary (pillar) article, supporting cluster articles, platform social copy, and scheduled publishing — then analytics feed the next cycle.

## Commands

| Command | Does |
|---------|------|
| `init <brand>` | Create a brand profile (voice, audience, authors, URL structure) from `schemas/brand.example.yaml`. |
| `run <brand> [--topic "…"]` | Execute the full flywheel for one or more topics. |
| `feedback <brand>` | Analyze published performance and tune the plan. |
| `status <brand>` | Report trust level, published count, pillar weights. |

## Phases

Run phases in order. Each phase is best handled by the agent role noted — map these to whatever models/tools you have (a strong reasoning model for planning, a fast model for drafting, a research/grounding model for sources, a review model for the gate).

1. **Research** — trends, SERP intent, competitor gaps, audience needs. *(research/grounding model)*
2. **Plan** *(2a)* — write `outline.md`: H2/H3 framework, thesis, key points, internal-link plan, external-link candidates, CTA, frontmatter + structured-data plan. **Inject [`frameworks/seo-writing-checklist.md`](frameworks/seo-writing-checklist.md) + [`frameworks/geo-techniques.md`](frameworks/geo-techniques.md) here.** *(strongest reasoning model — plans, does not write prose)*
3. **Draft** *(2b)* — write the full article to the outline. *(fast drafting model)*
4. **De-AI** *(2c)* — strip filler, generic phrasing, and "AI tells"; tighten to the brand voice. *(fast editing model)*
5. **Quality gate** *(2d)* — score the draft against [`frameworks/core-eeat-dimensions.md`](frameworks/core-eeat-dimensions.md) + GEO rules; list gaps; fix before continuing. *(review model)*
6. **Split** — adapt the article into platform copy (see social limits below). *(layout/copy model)*
7. **Carousel** *(optional)* — HTML/CSS slides → image export.
8. **Technical SEO** — schema/JSON-LD, INP/LCP/CLS, alt text, slug. *(technical model)*
9. **Publish** — publish the article + schedule social. *(automation)*
10. **Feedback** — analytics → adjust content-pillar weights, cadence, and topic selection. Run this at the **start** of each weekly cycle so insights guide the new content. *(reasoning model)*

## Trust score → autonomy level

A brand's `trust_score.current` (0–1) sets how much runs without a human:

| Trust | Level | Behavior |
|-------|-------|----------|
| < 0.5 | 0 — manual | Pause at topic selection and at draft review. |
| 0.5–0.7 | 1 — semi-auto | Pause at topic selection only. |
| 0.7–`auto_publish_at` | 2 — trusted | Auto-select topics; pause only on low-confidence drafts. |
| ≥ `auto_publish_at` | 3 — full auto | No pauses. |

Update trust after each human checkpoint: approved as-is `+0.02`; minor edits `+0.01`; major rewrite `−0.05`.

## Cadence (optional cron)

- **Weekly** — run Feedback (phase 10), then plan the next 7 days of topics into a `weekly-plan.yaml` (see `schemas/weekly-plan.example.yaml`).
- **Daily** — read the plan, run phases 2–9 for that day's topic, catch up any missed day, then deploy.

Make runs **idempotent**: record a per-day ledger and exit cleanly if today is already done; resume partial runs from the last completed phase. After updating any rule, **re-run** the affected phases — never reuse stale prior output.

## Social fan-out limits (validate before generating)

| Platform | Links | Hashtags | Char budget |
|----------|-------|----------|-------------|
| Instagram | 0 (use bio link) | ≤30 | 2200 |
| Threads | 0 | ≤10 | 500 |
| Facebook | ≤1 | ≤5 | ~500 (soft) |
| X | ≤1 | ≤3 | 280 |
| LinkedIn | ≤1 inline | ≤5 | 3000 |

A social post without a backing article does not count as a flywheel deliverable.

## Definition of done

1 run = 1 published topic cluster = pillar + cluster articles + platform copy + scheduled posts. Articles must pass the CORE-EEAT gate. Accumulate a batch of published items, then run Feedback to close the loop.

## Files

- `schemas/brand.example.yaml` — brand profile template (voice, audience, authors for E-E-A-T, URL rules, trust score).
- `schemas/content-matrix.example.yaml` — content pillars + cluster topics.
- `schemas/weekly-plan.example.yaml` — a 7-day plan.
- `frameworks/` — GEO, SEO, and CORE-EEAT references injected during planning and review.

## Credits

CORE-EEAT / CITE frameworks adapted from [aaron-he-zhu/seo-geo-claude-skills](https://github.com/aaron-he-zhu/seo-geo-claude-skills) (Apache-2.0). Apache-2.0.

---
> Source: [zhenheco/content-flywheel-skills](https://github.com/zhenheco/content-flywheel-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
