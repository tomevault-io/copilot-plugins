## obsedian-research

> > **Read [`AI_INSTRUCTIONS.md`](./AI_INSTRUCTIONS.md) first** — it contains the complete project context, architecture, coding standards, and testing guidelines for this repository.

# CLAUDE.md

> **Read [`AI_INSTRUCTIONS.md`](./AI_INSTRUCTIONS.md) first** — it contains the complete project context, architecture, coding standards, and testing guidelines for this repository.
>
> **DeepWiki:** <https://deepwiki.com/gundestrup/obsedian-research>

## Claude-specific notes

- This is an Obsidian plugin written in TypeScript. The entry point is `main.ts`.
- Always run `npm run lint` and `npm test` before declaring a task complete.
- Use tab indentation (not spaces) to match the project style.
- When linting UI text, proper nouns like "PubMed", "DOI", "PMC", and "NCBI" trigger `obsidianmd/ui/sentence-case`. Add `// eslint-disable-next-line obsidianmd/ui/sentence-case -- <reason>` on the line directly above the violation.
- API functions in `src/api.ts` use dependency injection via `RequestFunction` — never call `requestUrl` directly in those functions.
- Tests import from `src/` modules, not from duplicated test utilities.
- See `AI_INSTRUCTIONS.md` for full architecture, module graph, and build commands.

---
> Source: [gundestrup/obsedian-research](https://github.com/gundestrup/obsedian-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
