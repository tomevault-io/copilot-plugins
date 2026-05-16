## surface-aeo-geo-engine

> this is an AEO/GEO content engine that runs a full pipeline from question generation to blog publishing to visibility monitoring. the goal is to help a company turn weekly founder/operator recordings into articles that are useful to humans and easy for ChatGPT, Gemini, Perplexity, Google AI Overviews, and Copilot to cite.

# Claude Code Instructions

this is an AEO/GEO content engine that runs a full pipeline from question generation to blog publishing to visibility monitoring. the goal is to help a company turn weekly founder/operator recordings into articles that are useful to humans and easy for ChatGPT, Gemini, Perplexity, Google AI Overviews, and Copilot to cite.

Follow the SKILL.md instructions in each agent folder.

## agent order

run agents in order unless the user explicitly asks for a single step:

1. `agents/01-question-generator/SKILL.md`
2. `agents/02-transcript-extractor/SKILL.md`
3. `agents/03-angle-spinner/SKILL.md`
4. `agents/04-aeo-geo-optimizer/SKILL.md`
5. `agents/05-blog-generator/SKILL.md`
6. `agents/06-blog-optimizer/SKILL.md`
7. `agents/07-cms-publisher/SKILL.md`
8. `agents/08-visibility-monitor/SKILL.md`
9. `agents/09-content-refresher/SKILL.md`

## user config

user configuration lives in `templates/`:

- `templates/icp-definition-template.md` describes the buyer, pains, objections, triggers, and buying committee.
- `templates/brand-voice-template.md` describes tone, point of view, banned phrases, examples, and proof style.
- `templates/content-calendar-template.md` holds planned topics, owners, publication dates, and status.
- `templates/competitor-tracking.md` tracks competing pages, positioning, and citation gaps.
- `templates/keyword-research.md` tracks query clusters, search intent, and ai-answer opportunities.

## operating rules

- use answer-first structure whenever writing content.
- do not publish vague thought leadership. every piece needs a clear buyer question, concrete answer, proof, and next action.
- cite credible sources where claims depend on outside data.
- include original examples, first-party observations, numbers, and strong opinions when available.
- avoid ai-sounding filler like "in today's landscape", "dive deep", "game-changer", "unlock", "harness", and "it's worth noting".
- preserve the user's actual voice over generic polish.
- when publishing, ask for confirmation before sending content to a live cms.

---
> Source: [trysurface/surface-aeo-geo-engine](https://github.com/trysurface/surface-aeo-geo-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
