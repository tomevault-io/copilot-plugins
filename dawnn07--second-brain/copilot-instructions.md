## second-brain

> Use when building or maintaining a personal LLM-powered knowledge base. Triggers: ingesting sources into a wiki, querying wiki knowledge, 'add to wiki', 'what do I know about', or any mention of 'LLM wiki' or 'Karpathy wiki'.


# Karpathy LLM Wiki

Build and maintain a personal knowledge base using LLMs. You manage two directories: `raw/` (immutable source material) and `wiki/` (compiled knowledge articles). Sources go into raw/, you compile them into wiki articles, and the wiki compounds over time.

Core ideas from Karpathy:

- "The LLM writes and maintains the wiki; the human reads and asks questions."
- "The wiki is a persistent, compounding artifact."

## Architecture

Three layers, all under the user's project root:

**raw/** — Immutable source material. You read, never modify. Organized by topic subdirectories (e.g., `raw/machine-learning/`).

**wiki/** — Compiled knowledge articles. You have full ownership. Organized by topic subdirectories, one level only: `wiki/<topic>/<article>.md`. Contains two special files:

- `wiki/index.md` — Global index. One row per article, grouped by topic, with link + summary + Updated date.
- `wiki/log.md` — Append-only operation log.

**SKILL.md** (this file) — Schema layer. Defines structure and workflow rules.

Templates live in `references/` relative to this file. Read them when you need the exact format for raw files, articles, archive pages, or the index.

### Initialization

Triggers only on the first Ingest. Check whether `raw/` and `wiki/` exist. Create only what is missing; never overwrite existing files:

- `raw/` directory (with `.gitkeep`)
- `wiki/` directory (with `.gitkeep`)
- `wiki/index.md` — heading `# Knowledge Base Index`, empty body
- `wiki/log.md` — heading `# Wiki Log`, empty body

If Query cannot find the wiki structure, tell the user: "Run an ingest first to initialize the wiki." Do not auto-create.

---

## First-run bootstrap

Before the first invocation of any helper under `tools/` in a session (`tools/search.py`, `tools/graph.py`, `tools/dashboard.py`, `tools/watcher.py`), verify the Python deps are installed.

### Check

Run this once per session and remember the result:

```bash
python -c "import rank_bm25, networkx, textual, watchdog" 2>&1
```

- **Exit 0** — deps are installed. Proceed.
- **`ModuleNotFoundError`** — deps are missing. Go to Install.

### Install

1. Locate the skill root. If the working directory is a vault (contains `raw/` + `wiki/`), the skill lives elsewhere — typical locations:
   - `~/.claude/skills/second-brain/`
   - `~/.config/agent-skills/second-brain/`
   - wherever `npx skills add dawnn07/second-brain` dropped it

2. Tell the user exactly once:

   > This skill needs one-time Python setup (`pip install -e .` in the skill directory). This installs `rank-bm25`, `networkx`, `textual`, `watchdog`, and optional semantic-search libraries. Proceed?

3. On confirmation, run from the skill root:

   ```bash
   pip install -e .
   ```

   If `pip` is not available, fall back to:

   ```bash
   python -m pip install -e .
   ```

4. Re-run the check. If it still fails, report the error verbatim and stop — do not attempt further fallbacks (virtualenv creation, conda, etc.); that's the user's call.

### When to skip

- The user explicitly says "don't install anything" — fall back to direct `wiki/index.md` reads for Query, and tell the user graph/dashboard/watcher are unavailable until they install.
- The wiki has fewer than ~30 articles — Query can read `index.md` directly without search. Offer to skip install until the vault grows.

### What not to do

- Do not re-prompt for install within the same session. Cache the decision.
- Do not auto-install without asking. Always get consent first.
- Do not fall back to `pip install --user` or `sudo pip install`.

---

## Ingest

Fetch a source into raw/, then compile it into wiki/. Always both steps, no exceptions.

### Fetch (raw/)

1. Get the source content. For local files, use the Read tool. For URLs, use WebFetch. If neither can reach the source, ask the user to paste it directly.

2. Pick a topic directory. Check existing `raw/` subdirectories first; reuse one if the topic is close enough. Create a new subdirectory only for genuinely distinct topics.

3. Save as `raw/<topic>/YYYY-MM-DD-descriptive-slug.md`.
   - Read `references/raw-template.md` for the exact format.
   - Slug from source title, kebab-case, max 60 characters.
   - Published date unknown → omit the date prefix from the file name (e.g., `descriptive-slug.md`). The metadata Published field still appears; set it to `Unknown`.
   - If a file with the same name already exists, append a numeric suffix (e.g., `descriptive-slug-2.md`).
   - Include metadata header: source URL or local file reference, collected date, published date.
   - Preserve original text. Clean formatting noise. Do not rewrite opinions.

### Compile (wiki/)

Read `references/article-template.md` for the exact article format.

Determine where the new content belongs:

- **Same core thesis as existing article** → Merge into that article. Add the new source to Sources and Raw fields. Update affected sections.
- **New concept** → Create a new article in the most relevant topic directory. Name the file after the concept, not the raw file.
- **Spans multiple topics** → Place in the most relevant directory. Add See Also cross-references to related articles elsewhere.

These are not mutually exclusive. A single source may warrant merging into one article while also creating a separate article for a distinct concept it introduces. In all cases, check for factual conflicts: if the new source contradicts existing content, annotate the disagreement with source attribution. When merging, note the conflict within the merged article. When the conflicting content lives in separate articles, note it in both and cross-link them.

### Relationships

When compiling or updating an article, record typed edges to other articles in the `Relationships` frontmatter field. Use the narrowest type that fits:

- `supports` — the article reinforces a claim made by the target.
- `contradicts` — the article argues against the target. Also note the disagreement in the body under a `## Conflicts` heading.
- `supersedes` — the article replaces the target with newer/better information. The target is not deleted; it stays as history.
- `caused-by` — the target is a causal ancestor (event → event, decision → outcome).
- `related-to` — catch-all. Prefer a stronger type when one fits.

Rules:

- Edges are directional. If A supersedes B, add `supersedes:[B](...)` on A. Do not add the inverse on B.
- `contradicts` is symmetric: add it on both sides, so each article surfaces the conflict.
- When adding a new article, scan the target topic directory and `wiki/index.md` for candidates before writing the field. An article with zero edges is suspicious — recheck the index.
- Omit the field entirely when there are genuinely no edges; do not write `Relationships:` with an empty value.
- `Relationships` is machine-readable (used by `tools/graph.py`). `See Also` is human-readable navigation. A pair may appear in both — that is fine.

### Cascade Updates

After the primary article, check for ripple effects:

1. Scan articles in the same topic directory for content affected by the new source.
2. Scan `wiki/index.md` entries in other topics for articles covering related concepts.
3. Update every article whose content is materially affected. Each updated file gets its Updated date refreshed.

Archive pages are never cascade-updated (they are point-in-time snapshots).

### Post-Ingest

Update `wiki/index.md`: add or update entries for every touched article. Read `references/index-template.md` for the exact format. When adding a new topic section, include a one-line description. The Updated date reflects when the article's knowledge content last changed, not the file system timestamp.

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] ingest | <primary article title>
- Updated: <cascade-updated article title>
- Updated: <another cascade-updated article title>
```

Omit `- Updated:` lines when no cascade updates occur.

---

## Query

Search the wiki and answer questions. Examples of triggers:

- "What do I know about X?"
- "Summarize everything related to Y"
- "Compare A and B based on my wiki"

### Steps

1. **Decide whether to use helpers.** For wikis with fewer than ~50 articles, read `wiki/index.md` directly and skip helpers. For larger wikis, or when the question is about relationships ("what contradicts X", "what does Y supersede"), call the Python helpers below.

2. **Search** (when a question is about content similarity):

   ```bash
   python -m tools.search query "<question>" --top 5
   ```

   Parse the JSON and open the top article paths. If the index is missing or stale, rebuild first with `python -m tools.search index`. Pass `--no-embeddings` for a BM25-only run when the embeddings model is not installed.

3. **Graph** (when a question is about relationships):

   ```bash
   python -m tools.graph query "<topic>/<article>.md" --type contradicts --direction both
   ```

   Allowed `--type` values mirror the article schema: `supports`, `contradicts`, `supersedes`, `caused-by`, `related-to`. Omit `--type` to see every edge. Use `--direction in` to find what points AT a node.

4. **Read and synthesize.** Read the articles returned by search/graph (or from the index, for small wikis). Prefer wiki content over your own training knowledge. Cite sources with markdown links: `[Article Title](wiki/topic/article.md)` (project-root-relative paths for in-conversation citations; within wiki/ files, use paths relative to the current file).

5. **Output the answer in the conversation. Do not write files unless asked.**

### Archiving

When the user explicitly asks to archive or save the answer to the wiki:

1. Write the answer as a new wiki page. Read `references/archive-template.md` for the exact format. When converting conversation citations to the archive page, rewrite project-root-relative paths (e.g., `wiki/topic/article.md`) to file-relative paths (e.g., `../topic/article.md` or `article.md` for same-directory).
   - Sources: markdown links to the wiki articles cited in the answer.
   - No Raw field (content does not come from raw/).
   - File name reflects the query topic, e.g., `transformer-architectures-overview.md`.
   - Place in the most relevant topic directory.
2. Always create a new page. Never merge into existing articles (archive content is a synthesized answer, not raw material).
3. Update `wiki/index.md`. Prefix the Summary with `[Archived]`.
4. Append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] query | Archived: <page title>
   ```

---

## Conventions

- Standard markdown with relative links throughout.
- wiki/ supports one level of topic subdirectories only. No deeper nesting.
- Today's date for log entries, Collected dates, and Archived dates. Updated dates reflect when the article's knowledge content last changed. Published dates come from the source (use `Unknown` when unavailable).
- Inside wiki/ files, all markdown links use paths relative to the current file. In conversation output, use project-root-relative paths (e.g., `wiki/topic/article.md`).
- Ingest updates both `wiki/index.md` and `wiki/log.md`. Archive (from Query) updates both. Plain queries do not write any files.

---

## Lint

Quality checks on the wiki. Two categories with different authority levels.

### Deterministic Checks (auto-fix)

Fix these automatically:

**Index consistency** — compare `wiki/index.md` against actual wiki/ files (excluding index.md and log.md):

- File exists but missing from index → add entry with `(no summary)` placeholder. For Updated, use the article's metadata Updated date if present; otherwise fall back to file's last modified date.
- Index entry points to nonexistent file → mark as `[MISSING]` in the index. Do not delete the entry; let the user decide.

**Internal links** — for every markdown link in wiki/ article files (body text and Sources metadata), excluding Raw field links (validated by Raw references below) and excluding index.md/log.md (handled above):

- Target does not exist → search wiki/ for a file with the same name elsewhere.
  - Exactly one match → fix the path.
  - Zero or multiple matches → report to the user.

**Raw references** — every link in a Raw field must point to an existing raw/ file:

- Target does not exist → search raw/ for a file with the same name elsewhere.
  - Exactly one match → fix the path.
  - Zero or multiple matches → report to the user.

**See Also** — within each topic directory:

- Add obviously missing cross-references between related articles.
- Remove links to deleted files.

### Heuristic Checks (report only)

These rely on your judgment. Report findings without auto-fixing:

- Factual contradictions across articles
- Outdated claims superseded by newer sources
- Missing conflict annotations where sources disagree
- Orphan pages with no inbound links from other wiki articles
- Missing cross-topic references
- Concepts frequently mentioned but lacking a dedicated page
- Archive pages whose cited source articles have been substantially updated since archival
- Articles whose decayed confidence has fallen below 0.4 and should be re-verified

### Confidence Decay

When running lint, recompute each article's effective confidence based on its `Decay` field and the time since `Updated`:

- `slow` — multiply by 0.95 per year since Updated
- `medium` — multiply by 0.90 per 6 months since Updated
- `fast` — multiply by 0.80 per 3 months since Updated

Do not write the decayed value back to the article. The stored `Confidence` is the last asserted value; decay is applied at read time. Lint reports articles whose effective confidence has fallen below 0.4 so the user can re-verify and either refresh `Updated` (if still accurate) or re-compile (if now stale).

### Post-Lint

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] lint | <N> issues found, <M> auto-fixed
```

---

## Vault Health

Summarize the state of the wiki. Examples of triggers:

- "What is my vault status?"
- "Show me vault health"
- "How is my wiki doing?"

### Steps

1. Read `wiki/index.md` to enumerate articles and topics.
2. Read frontmatter of each article to collect: Confidence, Decay, Updated.
3. Read `wiki/log.md` to summarize recent activity (last 10 operations).
4. Compute and report:
   - **Totals**: article count, topic count, source count (unique raw/ files referenced).
   - **Confidence**: average effective confidence (after decay), distribution across buckets (0.8–1.0, 0.6–0.8, 0.4–0.6, 0.0–0.4).
   - **Stale**: articles whose effective confidence has fallen below 0.4.
   - **Orphans**: articles with no inbound links from other wiki articles (excluding archive pages).
   - **Recent activity**: last 5 log entries.
5. Output the report in the conversation. Do not write files.

---

## Migration

The skill evolves. Existing vaults must not break when the schema changes.

### Current schema version

**`1.0.0`** — baseline. Frontmatter fields: `Title`, `Sources`, `Raw`, `Updated`, `Confidence`, `Decay`, `Relationships` (optional), `Schema` on articles; `Source`, `Collected`, `Published`, `Schema` on raw files; `Title`, `Sources`, `Archived`, `Schema` on archive pages.

All new files created by Ingest or Archive set `Schema: 1.0.0`. When the skill bumps the version, this section documents the new version and the migration path from each prior version.

### Triggers

Examples of user phrasing that triggers a migration:

- "Migrate my wiki"
- "Upgrade to the latest schema"
- "Run a migration"
- "Show me what a migration would change" (dry-run)

### Steps

1. **Identify targets.** Walk `wiki/` and `raw/`. For each file, read the `Schema` field. If absent, treat as `0.0.0`. Skip `wiki/index.md`, `wiki/log.md`, and archive pages (archives are historical snapshots — do not rewrite them).

2. **Compare versions.** For each file, compare the stored version against the current version above. Collect files that are strictly older.

3. **Plan rewrites.** For each outdated file, determine what fields must be added, renamed, or removed to match the current schema. Do not invent values for new required fields — if a field cannot be derived (e.g., `Confidence` for a pre-M2 article), leave the field out and note it so the user can supply it.

4. **Dry-run or apply.**
   - **Dry-run** (default when the user asks "what would change"): print a report to the conversation grouping files by target version → current version, listing the field changes per file. Do not write anything.
   - **Apply** (when the user explicitly says "apply" or "migrate for real"): rewrite the frontmatter of each outdated file in place. Bump its `Schema` field to the current version. Leave body content untouched unless the schema change requires it.

5. **Log.** On apply, append to `wiki/log.md`:

   ```
   ## [YYYY-MM-DD] migrate | <old> → <new> | <N> files updated
   - <path>
   - <path>
   ```

   On dry-run, do not write to the log.

### Version bump policy

- **Patch (x.y.Z)** — clarifying wording only, no frontmatter changes. No migration needed.
- **Minor (x.Y.0)** — additive fields. Migration backfills defaults where derivable, omits where not.
- **Major (X.0.0)** — breaking changes (renames, removals). Migration must rewrite every affected file; the user is asked to confirm before the first Apply.

---

## Hooks

External systems can drive Ingest without a human typing anything. The skill does not run a persistent process itself — hooks integrate with tools the user already has.

### Filesystem watcher (recommended for local vaults)

```bash
python -m tools.watcher --vault /path/to/vault --debounce 1.5
```

Watches `raw/` recursively. When a stable new `.md`, `.pdf`, or `.txt` file appears, invokes `claude -p "ingest raw/<topic>/<file>"` from the vault root. The skill's Ingest section handles the rest.

### Git post-commit hook

`hooks/post-commit.sample` ships a bash template: when a commit message begins with `raw:` or `source:`, it triggers an ingest for every newly-added file under `raw/` in that commit. Install with:

```bash
cp hooks/post-commit.sample .git/hooks/post-commit
chmod +x .git/hooks/post-commit
```

The hook is opt-in. Users who prefer the watcher can ignore it.

### Auto-enrichment during Query

While reading articles to answer a query, track low-confidence and stale pages as a side effect:

- If an article's effective confidence (after decay) is below `0.4`, flag it in the answer with a one-line note: "Source `X` is low-confidence (0.3); consider re-ingesting a newer source."
- If an article's `Updated` date is more than 180 days old and `Decay` is `fast`, flag it as stale.
- Do not block the answer. Do not rewrite the article. Surfacing the signal is enough — the user can re-ingest on demand.

Auto-enrichment is a read-time observation, not a write. It never modifies wiki files.

---
> Source: [dawnn07/second-brain](https://github.com/dawnn07/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
