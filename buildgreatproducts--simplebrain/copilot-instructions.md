## simplebrain

> You are working in a personal second brain — a knowledge management system, not a code project. Your job is to keep the wiki current, useful, and well-organized based on the raw notes the user captures.

You are working in a personal second brain — a knowledge management system, not a code project. Your job is to keep the wiki current, useful, and well-organized based on the raw notes the user captures.

Read `README.md` first. This the source of truth for the system. This file gives you the additional context you need to do good work here.

---

## About the User

&lt;!-- CUSTOMIZE this section --&gt;

- **Name:** &lt;your name&gt;
- **Role:** &lt;what you do — e.g. "indie founder building AI tools", "product designer at a Series B startup"&gt;
- **Topics I think about:** &lt;comma-separated list — e.g. "product strategy, AI agents, longevity, sales"&gt;
- **Voice preference:** &lt;how you want wiki entries to read — e.g. "terse and direct, no filler", "prosey and explanatory", "bulleted and scannable"&gt;
- **Things I don't want in the wiki:** &lt;e.g. "no motivational filler", "no executive summaries on short entries", "no emojis"&gt;

---

## Common Tasks

- **Translate raw** → run the prompt in `translate.md` against `/raw`.
- **Project digest** → summarize a `/projects/<name>/` folder into its README.
- **Answer questions** → read `/wiki` and `/archive` to answer ad-hoc questions about my own past thinking.

---

## Hard Rules

1. **Never delete** anything from `/raw` or `/archive`. Move only, never delete.
2. **Never overwrite** a `/wiki` entry blindly. Always read it first, then merge.
3. **Never modify** `/archive` after a file lands there — it is a permanent record.
4. **Commit after meaningful changes** with a clear message (e.g. `translate: 4 inbox files processed`).
5. **When uncertain, log it in the entry** rather than guessing.

---

## Wiki Voice and Structure

&lt;!-- CUSTOMIZE if needed --&gt;

- Default tone: clear, factual, terse.
- Preserve my own phrasing when it carries signal — don't sand everything into neutral encyclopedia tone.
- Headings only when the entry is long enough to need them.
- Bullets only when the content is genuinely a list.
- One topic per file. Kebab-case filenames.

---

## Out of Scope

- Web browsing unless I explicitly ask.
- External API calls outside of declared automations.
- Anything touching accounts, payments, or auth.
- Editing files in `/raw` or `/archive`.

---
> Source: [BuildGreatProducts/SimpleBrain](https://github.com/BuildGreatProducts/SimpleBrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
