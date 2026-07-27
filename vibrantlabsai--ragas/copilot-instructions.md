## ragas

> When writing or editing documentation, categorise each page as **one** of the four Diátaxis modes and follow its specific guidance. *Do not mix modes in a single page.*

# Diátaxis Documentation Guidelines

When writing or editing documentation, categorise each page as **one** of the four Diátaxis modes and follow its specific guidance. *Do not mix modes in a single page.*

## 1. Tutorials  🧑‍🏫  (`docs/getstarted/`, `docs/experimental/tutorials/`)
• Purpose: provide a structured **learning experience** – “Can you teach me to…?”  
• Form: narrative lesson that leads the reader from zero to a working result.  
• Include: context, motivation, complete working example.  
• Avoid: deep technical detail; troubleshooting; exhaustive options.

## 2. How-to Guides  🍳  (`docs/howtos/`)
• Purpose: help the user **achieve a specific goal** – “How do I…?”  
• Form: concise series of **step-by-step instructions** focused on the user’s project.  
• Write from the **user’s perspective**, not the tool’s operations.  
• Link to reference and explanation pages for background; keep prose minimal.
• Add any code run outputs as expandable click blocks. Readers should be able to understand the guide without running the code.

## 3. Reference  📑  (`docs/references/`)
• Purpose: provide **neutral, complete, accurate description** of APIs, commands, options – “What is…?”  
• Maintain consistent patterns (parameter tables, return types, examples).  
• Avoid instruction or opinion; instead *link* to how-to or explanation pages.  
• Examples are welcome if they illustrate usage without drifting into tutorial style.

## 4. Explanation  💡  (`docs/concepts/`, `docs/experimental/core_concepts/`)
• Purpose: **clarify concepts and rationale** – “Why…?”  
• Form: discursive article that illuminates design decisions, theory, background.  
• May link out to tutorials, how-tos, and reference, but does not instruct step-by-step.

### Keep the Borders Sharp
• Do **not** let content blur between modes (e.g., no instructions inside reference; no lengthy theory in how-tos).  
• If a page starts serving two modes, split it.

### Filing & Navigation
• Place the file in the folder matching its mode (above).  
• Update `mkdocs.yml` `nav:` under the corresponding section.

### Incremental Improvement Cycle (per Diátaxis)
Choose → Assess → Decide → Do. Focus on small, atomic upgrades rather than grand rewrites.

### Writing Style
• Use second-person ("you") and active voice.
• Ensure code blocks are **copy-pasteable** and include necessary context (imports, environment).
• Prefer short sentences; use Markdown admonitions (`!!! note`, `!!! warning`) sparingly for important side-information.
• Use `??? "Click to expand"` collapsible admonitions to contain outputs, long prompts, verbose logs, or any content that would clutter the main article flow. This keeps the primary content scannable while preserving detailed information for readers who need it.
• **Always add a blank line after text ending with a colon before starting a list.** This ensures proper Markdown rendering in MkDocs. Without the blank line, list items may render as continuation text instead of a proper bulleted/numbered list.

### Cross-linking Between Modes
• End tutorials with pointers to relevant how-to guides for further exploration.
• How-to guides should include links to reference/API pages for deeper details.
• Explanations can reference tutorials and how-tos to illustrate concepts in action.

### Page Metadata & Prerequisites
• Start each page with a one-sentence purpose statement and a brief list of prerequisites (libraries, data, environment variables).
• Highlight any external services/configuration required before the reader begins.

### Keep Pages Atomic
• One page = one task, concept, or API surface. If content grows, **split** rather than creating a mega-guide.

---
> Source: [vibrantlabsai/ragas](https://github.com/vibrantlabsai/ragas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
