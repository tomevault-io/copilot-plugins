## html-effectiveness

> use when a user asks for a spec, plan, report, explainer, code review, pr writeup, design exploration, prototype, diagram, slide-like artifact, or custom editor where markdown would be hard to read, compare, share, or interact with. also use when the user says html effectiveness, unreasonable effectiveness of html, make an html file, make an html artifact, make it visual, side by side, interactive, copy/export, or easier to review.


# HTML Effectiveness

## Core rule

Prefer a **single, self-contained HTML artifact** when the user needs to understand, compare, review, tune, or share complex work. HTML is not decoration; it is a readable interface over the answer.

Use Markdown only when the output is short, linear, and will be edited directly as text. Use HTML when the answer benefits from layout, color, SVG, tables, tabs, collapsible sections, code annotations, interactivity, or export buttons.

## Start with an artifact brief

Before writing the artifact, infer or ask only the missing essentials:

1. **Reader:** who will open this and what do they already know?
2. **Decision/action:** what should they decide, review, copy, or do after reading?
3. **Source material:** code, docs, tickets, data, conversation, or assumptions.
4. **Best shape:** comparison grid, annotated diff, explainer, report, diagram, prototype, deck, or editor.
5. **Return path:** if the user edits/tunes inside the page, add `copy as prompt`, `copy JSON`, `copy diff`, or `copy markdown`.

When enough context already exists, proceed without extra questions.

## Pattern picker

Choose the closest source pattern from `references/example-patterns.md` and `references/examples`:

| User need | HTML shape |
|---|---|
| Explore options | side-by-side cards with tradeoffs and recommendation |
| Implementation handoff | milestones, mockups, data flow, key code, risks, open questions |
| Code review | annotated diff, severity tags, file risk map, next steps |
| Explain unfamiliar code | request path/call graph, source snippets, gotchas, key files |
| Design review | token swatches, component matrices, live variants |
| Prototype feel | tiny throwaway interaction with sliders/toggles and copyable parameters |
| Report/status | metric cards, timeline, shipped/slipped/blocked sections, sources |
| Research/learning | TL;DR, diagram, collapsible steps, glossary, FAQ |
| Custom editing | purpose-built mini-editor with validation and export button |

Load `references/html-effectiveness-principles.md` for the full decision criteria and anti-patterns.
Load `references/prompt-recipes.md` when the user asks for reusable prompts.
Load `references/quality-checklist.md` before final delivery of substantial HTML.

## HTML artifact requirements

Create artifacts as portable files unless the environment provides a native HTML/canvas artifact:

- Use one `.html` file with inline CSS, inline JS, and inline SVG where possible.
- Avoid build steps, package managers, frameworks, CDNs, remote fonts, and external assets unless the user explicitly wants them.
- Make the artifact responsive: usable on phone and desktop.
- Optimize scanning: sticky TOC or sidebar for long pages, clear section labels, high-signal headings, and compact cards.
- Use visual encoding deliberately: color = status/severity/category, layout = relationship, motion = feel or state change.
- Keep sample data clearly labeled if any content is fictional or inferred.
- For interactive editors, always include a way to export the user's changes back into text, JSON, diff, or a prompt.
- For reports or explainers, show sources/assumptions in the artifact.
- For code review, preserve code readability and point to exact files/lines when available.
- For implementation plans, make the handoff independently executable by another agent or engineer.

## Recommended workflow

1. **Classify the job.** Decide whether HTML improves readability, comparison, interaction, or sharing.
2. **Select a pattern.** Use the pattern picker and source examples rather than inventing a new structure from scratch.
3. **Sketch the information architecture.** Define sections, navigation, visual elements, and export path.
4. **Generate the artifact.** Write self-contained HTML. Prefer semantic HTML and CSS variables.
5. **Verify the artifact.** For files, run `scripts/check_html_artifact.py path/to/file.html` when tools are available, then fix errors and important warnings.
6. **Deliver with usage notes.** Briefly tell the user what the artifact is for and how to open or use it.

## Helper scripts

- `scripts/new_html_artifact.py` creates a starting `.html` file for common patterns. Use it when a quick scaffold is useful, then customize the result heavily.
- `scripts/check_html_artifact.py` checks for self-contained-file issues and common HTML-effectiveness misses before delivery.

## Common mistakes

- Do not make a pretty brochure when the user needs a decision tool.
- Do not flatten spatial information into paragraphs if a table, diff, diagram, or grid would be clearer.
- Do not build a reusable app when the user asked for a throwaway review/editing surface.
- Do not omit export/copy buttons from editors; without export, the loop back to the agent is broken.
- Do not hide uncertainty. Mark assumptions, fictional data, and unresolved decisions.
- Do not cite the Anthropic examples as proof for the user's facts; use them only as artifact-structure inspiration.

---
> Source: [blnkoff/html-effectiveness](https://github.com/blnkoff/html-effectiveness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
