## llm-wiki-kit

> Personal knowledge base maintained by Claude Code in Obsidian.

# LLM Wiki

Personal knowledge base maintained by Claude Code in Obsidian.

## Structure

```
raw/          # IMMUTABLE — never modify. Human drops sources here.
  ai-tools/   repos/   articles/   projects/   assets/

wiki/         # CLAUDE-OWNED — Claude writes and maintains everything.
  index.md    log.md
  tools/   repos/   concepts/   people/   projects/   comparisons/   insights/

scripts/      # CLI helpers: search.sh, wiki-stats.sh, quick-add.sh
```

## Rules

1. Never modify `raw/` — immutable source of truth
2. Always update `wiki/index.md` and `wiki/log.md` after every operation
3. Use `[[wikilinks]]` for all cross-references — no page is an island
4. YAML frontmatter required on every wiki page
5. Flag contradictions with `> [!warning]` — never silently overwrite
6. Prefer updating existing pages over creating duplicates
7. Kebab-case filenames: `cursor-ai.md`
8. Date time-sensitive claims: "as of YYYY-MM"
9. Cross-reference aggressively after every ingest

## Workflows

**Ingest**: `ingest raw/ai-tools/x.md` → read source → read ONE relevant template → create/update wiki page → cross-reference → update index + log → report

**Query**: question → read index → find/read relevant pages → synthesize with citations → offer to file as insight

**Lint**: `wiki lint` → run checklist → auto-fix safe issues → report contradictions/stale to user → update log

**Batch**: `ingest all new` → find unprocessed raw files → ingest each sequentially

<!-- CUSTOMIZE: Replace the section below with your own context -->
## About the Wiki Owner

I'm a solo AI engineer tracking four knowledge streams:
1. AI tools & products I discover
2. Open source repos worth tracking
3. Articles and social posts (LinkedIn, Twitter, blogs)
4. My own projects — docs, progress, architecture decisions

Write for a senior AI engineer. Be direct, opinionated, technically precise.
<!-- END CUSTOMIZE -->

---
> Source: [AnisurRahmann/llm-wiki-kit](https://github.com/AnisurRahmann/llm-wiki-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
