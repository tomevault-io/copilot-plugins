## portuguese-legal-rag-bot

> Citation-graph RAG over Portuguese law: vector search finds an entry article →

# Portuguese Legal RAG Bot

Citation-graph RAG over Portuguese law: vector search finds an entry article →
outbound graph traversal expands along typed citation edges → an LLM generates a
cited, grounded answer. See `README.md` (how to run) for orientation.

Full design rationale lives in the project wiki (see the WikiJS section below):
[Architecture](projects/portuguese-legal-rag/architecture),
[UI decisions](projects/portuguese-legal-rag/ui-design-decisions),
[Auth decisions](projects/portuguese-legal-rag/auth-design-decisions).

## Key architecture — load-bearing (do not break)

These are invariants the code depends on. Several source files reference this
section (the canonical in-repo spec).

1. **All article IDs come from `ids.make_article_id`. Nowhere else.** The canonical
   ID (e.g. `CP:152`) is the shared join key between Qdrant and Neo4j.
2. **The LLM extracts citations; `ids.py` canonicalizes them.** The model proposes
   a code + article number; the ID is built deterministically against
   `CODE_REGISTRY`, so the join key never drifts even though extraction is
   probabilistic.
3. **Stubs live in Neo4j only; never embedded in Qdrant.** Two-pass ingestion
   `MERGE`s citation targets — a not-yet-ingested target becomes a stub
   (`resolved=false`, no text); ingesting it later fills the text and flips
   `resolved=true`, so ingestion order doesn't matter. A stub is a *traversal*
   target, never a *retrieval* target.
4. **Every answer is grounded in freshly retrieved articles with citations;**
   memory sets context and tone, never serves as a source of truth.

### Citation taxonomy (the extraction spec)

Two orthogonal dimensions. **Edge TYPE** (`EdgeType` in `ingestion/models.py`) —
the legal force of a reference; wrong label still points at the right article (low
stakes): `REFERS_TO` (default), `WITHOUT_PREJUDICE_TO`, `EXCEPT_AS_PROVIDED`,
`PENALIZED_UNDER` (offence→sanction, key cross-KB edge), `APPLIES`,
`FOR_PURPOSES_OF`, `AMENDED_BY`/`REVOKED_BY`.

**Target FORM** (HIGH STAKES — `ids.normalize_article_number` must canonicalize):
single/abbreviated (`art. 152.º`), suffix (`5.º-A` — never drop the `-A`),
drill-down (`n.º 2 do artigo 7.º` — n.º/alínea go on the *edge*, not the node id),
range (`artigos 12.º a 15.º` → expand), list, cross-code (trailing `do Código
Penal` selects the KB), bare (current document's code), relative (`o artigo
anterior` → flagged `needs_llm` for the residue pass).

## WikiJS project pages — keep them in sync

Project documentation lives in WikiJS, maintained via the **WikiJS MCP**
(`foxcore:WikiJS`):

| Page | ID | Path | Update cadence |
|---|---|---|---|
| Project overview | `139` | `projects/portuguese-legal-rag` | **Live** — keep current |
| Architecture & rationale | `140` | `projects/portuguese-legal-rag/architecture` | Design history — rarely |
| UI design decisions | `141` | `projects/portuguese-legal-rag/ui-design-decisions` | Design history — rarely |
| Auth design decisions | `142` | `projects/portuguese-legal-rag/auth-design-decisions` | Design history — rarely |

**Keep the overview (139) current.** Update it whenever something *meaningful*
changes — the **architecture, stack, auth/security model, ingested data sources,
major features, or implemented/remaining status**. Do **not** update for trivial
changes (typos, small refactors, dependency bumps, internal renames, test-only
tweaks).

The three design-history pages (140–142) are snapshots of past design reviews;
touch them only if a decision they record is reversed (then add a note, don't
rewrite history).

**How to update:**
1. `get_page(139)` to read current content.
2. Edit only the sections that changed; keep structure and tone.
3. Bump the `Last reviewed:` date in the maintenance note at the top.
4. `update_page(page_id=139, content=…)` with the full revised Markdown.

If a page is missing (deleted), recreate it at the same path with `create_page`
and update its ID in the table above.

Do this proactively at the end of any change that meets the bar above — no need to
ask first; make the edit and mention it in your summary.

---
> Source: [Raposo06/Portuguese-Legal-RAG-Bot](https://github.com/Raposo06/Portuguese-Legal-RAG-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
