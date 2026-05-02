## mindpalace

> - `raw/` is immutable and is the source of truth.

# Knowledge base rules

## Sources and precedence

- `raw/` is immutable and is the source of truth.
- `wiki/` is compiled knowledge maintained by Codex.
- Documents under `references/` are background inspiration only.
- If `references/llm-wiki.md` exists, use it only for high-level workflow inspiration, not as factual source material for the user's projects or wiki content.
- Local repository conventions in this `AGENTS.md` and explicit user instructions take priority over external reference documents.
- Do not import extra complexity from reference documents unless the user explicitly asks for it, especially additional tooling, search infrastructure, MCP, skills, hooks, or nested agent schemas.
- Do not assume Obsidian or any Obsidian plugin is available; emulate Obsidian-like knowledge connections using repository-native Markdown conventions only.

## Wiki maintenance

- Write `wiki/` primarily in Korean, but keep proper nouns and necessary technical terms in their original language when helpful; avoid writing pages in excessively English-only or Korean-only style.
- On every ingest, update `wiki/index.md` and append one entry to `wiki/log.md`.
- If an ingest involves changes to 2 or more `raw/` sources, use subagents for the ingest work.
- In multi-source ingest, assign each subagent an isolated subset of `raw/` paths and have it return only the reusable synthesis, supporting evidence, and citation candidates for those paths.
- In multi-source ingest, keep the main agent focused on integrating subagent outputs into `wiki/`, resolving overlaps, and updating `wiki/index.md` and `wiki/log.md`; inspect `raw/` directly only when subagent output is conflicting or insufficient.
- Prefer updating existing topic pages over creating new ones.
- Create a new topic page only when a concept recurs across sources or the user explicitly asks.
- Keep topic pages interlinked in an Obsidian-like way using standard Markdown links so the repository remains readable on GitHub and without Obsidian-specific tooling.
- When a page title or file path changes, update affected inbound and outbound links in the touched `wiki/` pages, plus `wiki/index.md` when needed.
- Answer from `wiki/` first; inspect `raw/` only when evidence is missing.
- Persist only reusable syntheses, not every chat answer.
- If a `raw/` source is deleted and a `wiki/` page depends only on that source, delete the corresponding `wiki/` page and update `wiki/index.md` and `wiki/log.md` in the same change.
- If a `wiki/` page is supported by multiple `raw/` sources, remove the deleted source from its citations and keep the page only if enough remaining evidence still supports the synthesis.
- If a `raw/` source was only moved or renamed, update the recorded source path instead of treating it as a deletion.

## Topic linking

- Treat links as navigation and synthesis aids, not as evidence transfer. A link between pages does not by itself justify copying claims, definitions, or conclusions across pages.
- Use relative Markdown links for page, section, and same-page references, for example `[DoLa](./dola.md)`, `[DoLa - 방법](./dola.md#방법)`, and `[방법](#방법)`.
- Add a `## Related` section near the end of a topic page when there are meaningful neighboring topics. Keep it concise with 2-5 high-signal links, and include a short phrase explaining each relation.
- Prefer precise links to headings or narrow sections when only part of another page is relevant; avoid broad page-level links when the relation is scoped.
- Add links only for substantial relationships such as shared mechanism, contrast, dependency, attack/defense relation, evaluation connection, or recurring comparison in the sources.
- When adding a meaningful relation from page A to page B, add the reciprocal relation to page B only if the reverse connection is genuinely useful and not redundant.
- To avoid context contamination, do not preload or summarize all linked pages. Start from the page directly relevant to the task, and expand only to directly linked pages when the user asks for comparison, dependency tracing, or broader synthesis.
- When expanding beyond the current page, prefer section-level extraction and quoted evidence over whole-page carryover.

## Repository workflow

- Manage the entire project with Git and GitHub, and treat repository state as part of the maintained artifact.
- After making intentional changes, review `git status` before finishing.
- Commit intentional repository changes before ending the task unless the user explicitly asks to keep them uncommitted.
- If the current branch tracks a remote, push committed changes unless the user explicitly asks to keep the work local.
- Keep diffs small and list changed files at the end.
- In the final response, report what Git/GitHub actions were taken.

---
> Source: [jemin7709/MindPalace](https://github.com/jemin7709/MindPalace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
