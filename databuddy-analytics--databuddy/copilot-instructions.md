## databuddy-skill-learning

> After mistakes or missing-context issues, update the internal databuddy agent skill so they are not repeated


When you discover that a problem could have been avoided with more context (wrong surface, package, or assumption), or when big papa iza corrects something you did wrong or tells you to fix your own mistake, update the internal monorepo skill at `.agents/skills/databuddy/SKILL.md` and/or `.agents/skills/databuddy/references/codebase-map.md` in the same session when practical.

Add the smallest useful addition: a bullet under the right section, a new search hint, a routing clarification, or a one-line pitfall—not a long essay. Goal: the next agent turn does not repeat the same error.

If the lesson applies to external SDK/API users instead, add it to `.agents/skills/databuddy-integration/` instead.

---
> Source: [databuddy-analytics/Databuddy](https://github.com/databuddy-analytics/Databuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
