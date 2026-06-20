## save-to-kb

> Personal knowledge base manager for LLM-Brain-Bases. Four modes: (1) AUTO-SAVE — automatically called at end of significant conversations to save reusable knowledge as Cards; (2) INIT — run "save-to-kb init" on first install to create directory structure and configure KB path; (3) CHECK — run "save-to-kb check" to scan the entire knowledge base for broken links, inconsistencies, and missing data, then fix with user confirmation; (4) REPLAY — run "save-to-kb replay" to retroactively process historical sessions that were never saved. Config path: ~/.claude/skills/save-to-kb/.config


# save-to-kb

## Route: Which Procedure to Run

**Read the invocation context first:**

- Invoked with argument `init`        → run **[INIT PROCEDURE]**
- Invoked with argument `check`       → run **[CHECK PROCEDURE]**
- Invoked with argument `replay`      → run **[REPLAY PROCEDURE]**
- Invoked with argument `save`        → run **[AUTO-INIT CHECK]** then **[SAVE PROCEDURE]**
- Invoked with argument `file <path>` → run **[FILE PROCEDURE]** with the given path
- Invoked without arguments          → run **[MENU]**

---

## [MENU]

Present a **two-level menu** using AskUserQuestion (max 4 options per question).

**Level 1** — main actions:
- save   — 从当前对话提取知识并保存到知识库
- file   — 分析指定文件并提取知识（PDF / MD / txt）
- check  — 健康检查，扫描断链和不一致
- 管理知识库… — 进入二级菜单（init / replay）

If user selects "管理知识库…", show **Level 2**:
- init   — 初始化知识库或更改路径
- replay — 回溯历史 session，补录遗漏的知识

Route based on final selection:
- save   → **[AUTO-INIT CHECK]** then **[SAVE PROCEDURE]**
- file   → **[FILE PROCEDURE]**
- check  → **[CHECK PROCEDURE]**
- init   → **[INIT PROCEDURE]**
- replay → **[REPLAY PROCEDURE]**

---

## [AUTO-INIT CHECK]

Before running the save procedure, verify the KB is initialized:

1. Check if `~/.claude/skills/save-to-kb/.config` exists and contains a path
2. If missing or empty → **run [INIT PROCEDURE] first**, then continue with save

---

## [INIT PROCEDURE]

Two scenarios: **first-time setup** and **change path**. Both follow the same steps but with different prompts and behaviors.

### Step I-1: Detect Scenario

Read `~/.claude/skills/save-to-kb/.config`:

- **File missing or empty** → **First-time setup**. Proceed to Step I-2.
- **File exists with a valid path** → **Already initialized**. Report:
  ```
  Knowledge base is configured at: {current-path}
  ```
  Ask: "Change to a new path? (yes/no)"
  - If no → stop.
  - If yes → this is a **path change**. Ask:
    ```
    Migrate existing knowledge base to the new location?
    - yes: copy all files (Bases/, Cards/, Wiki/, Claude.md) to new path. Old files are kept.
    - no:  create a new empty knowledge base at new path. Old files untouched.
    ```
    Record the user's choice as `{migrate: true/false}`. Proceed to Step I-2.

### Step I-2: Ask for Path

Ask the user:
> "Enter the full path for the knowledge base (e.g. ~/Documents/LLM-Brain-Bases or /Users/you/Obsidian/Work/LLM-Brain-Bases):"

Expand `~` to the actual absolute home directory path. Show the resolved path and ask: "Confirm: create knowledge base at {resolved-path}? (yes/no)"

If no → ask again for a new path.

### Step I-3: Create Directory Structure

Create the following directories (skip any that already exist):
```
{KB}/
{KB}/Bases/
{KB}/Cards/
{KB}/Wiki/
```

### Step I-3b: Migrate Files (path change + migrate: true only)

Skip this step for first-time setup or if user chose not to migrate.

Copy all contents from `{old-KB}/` to `{new-KB}/`:
- `{old-KB}/Bases/` → `{new-KB}/Bases/`
- `{old-KB}/Cards/` → `{new-KB}/Cards/`
- `{old-KB}/Wiki/`  → `{new-KB}/Wiki/`
- `{old-KB}/Claude.md` → `{new-KB}/Claude.md`

**Rules:**
- Copy, do not move. Old files remain untouched.
- Do not overwrite files that already exist at the new location.
- Wikilinks inside the files are relative (`[[Cards/slug]]`), so they require no modification.
- After copying, report: `Migrated {N} files to {new-KB}.`

### Step I-4: Write Master Index

**First-time setup**: Write `{KB}/Claude.md` only if it does not exist (never overwrite an existing index).
**Path change**: Write `{KB}/Claude.md` only if it does not exist at the new location.

```markdown
# LLM-Brain-Bases — Master Knowledge Index

*Auto-maintained by Claude. Last updated: {YYYY-MM-DD}.*

## How to Use This Index

Before starting any non-trivial task:
1. Check the **Scenario Quick-Lookup** table below for the task type
2. If found, read the linked Cards directly
3. If not found, identify the relevant category in **Knowledge Map** and open the Wiki file

To save new knowledge: invoke `save-to-kb` at conversation end.
To health-check the KB: invoke `save-to-kb check`.

---

## Knowledge Map

| Wiki | Cards | Focus | Key Tags |
|------|-------|-------|----------|
| [[Wiki/ai-tools]] | 0 | Claude Code, LLM tools, prompting, skills | claude, hooks, skills, llm |
| [[Wiki/workflow]] | 0 | Development process, tooling, automation | git, ci, planning |
| [[Wiki/debugging]] | 0 | Systematic diagnosis patterns | debugging, logging, tracing |
| [[Wiki/architecture]] | 0 | System design, decisions, tradeoffs | design, patterns, architecture |
| [[Wiki/knowledge-management]] | 0 | Knowledge base, Obsidian, PKM | obsidian, pkm |
| [[Wiki/frontend]] | 0 | Vue3, Nuxt, React, SSR, UI | vue3, nuxt, react, ssr |
| [[Wiki/backend]] | 0 | APIs, databases, server logic | postgres, redis, api |
| [[Wiki/infra]] | 0 | Deployment, CI/CD, cloud | docker, vercel, github-actions |
| [[Wiki/business]] | 0 | Non-technical context, decisions | client, scope, product |

---

## Scenario Quick-Lookup

| Scenario | Start Here |
|----------|-----------|
| *(empty — populated after first save)* | — |

---

## Cross-Domain Cards

*(empty — populated after first save)*

---

## Recently Added

*(empty — populated after first save)*

---

## Stats

- Total cards: 0
- Total bases (sessions): 0
- Wiki categories: 0
- Last save: —
```

### Step I-5: Write Config File

Write the confirmed KB path (single line) to `~/.claude/skills/save-to-kb/.config`:
```
/full/expanded/path/to/KB
```

### Step I-6: Update Global CLAUDE.md

Read `~/.claude/CLAUDE.md`. Find the `<!-- LLM_BRAIN_BASES_BLOCK:BEGIN -->` marker.

**First-time setup — block does not exist**: Append the following block at the end of the file:

```markdown
<!-- LLM_BRAIN_BASES_BLOCK:BEGIN -->
## LLM-Brain-Bases 个人知识库

知识库路径：`{KB}`
（路径配置文件：`~/.claude/skills/save-to-kb/.config`）

### 规则 1：任务前查库

在开始以下类型的非平凡任务前，先查询知识库：
- 调试不熟悉的错误或行为
- 集成特定的库、API 或服务
- 系统设计或架构决策
- 涉及之前接触过的代码库或技术栈

操作步骤：
1. 读取 `{KB}/Claude.md`（主索引）
2. 从"场景快查"或"知识地图"定位相关 Wiki 和 Cards
3. 读取相关 Cards 后再开始工作

以下情况可跳过：纯机械操作、纯写作任务、与已有知识库完全无关的领域。

### 规则 2：对话结束自动保存

对话产出以下任何一项时，在对话末尾主动调用 `save-to-kb`：

- 调试发现了非显而易见的根因
- 发现了某 API/库/工具的文档未记录的行为
- 做出了有明确推理的架构或设计决策
- 总结出让重复任务明显更高效的工作流模式
- 验证了某种不该重蹈的失败路径

**判断标准**：刚加入项目的工程师在做类似任务前读到这张卡片会有收益吗？有则保存。

以下情况不保存：纯机械任务输出、标准教科书知识、一次性交付物、结论未明确的探索。

保存操作静默执行，无需向用户宣告（除非被问及）。
<!-- LLM_BRAIN_BASES_BLOCK:END -->
```

**Path change — block already exists**: Find the line starting with `知识库路径：` inside the block and update the path value to the new `{KB}` path. Leave all other content unchanged.

### Step I-7: Confirm

Report to user:
```
✓ Knowledge base initialized at: {KB}
✓ Directory structure created: Bases/ Cards/ Wiki/
✓ Master index written: Claude.md
✓ Config saved: ~/.claude/skills/save-to-kb/.config
✓ CLAUDE.md rules updated

Run `save-to-kb check` at any time to audit the knowledge base.
```

---

## [REPLAY PROCEDURE]

Retroactively process historical session files that were never saved to the knowledge base.

### Step R-1: Load Config

Read KB path from `~/.claude/skills/save-to-kb/.config`. If missing → report "Not initialized. Run `save-to-kb init` first." and stop.

### Step R-2: Locate Session Files

Claude Code stores session logs as JSONL files. Find the directory for the current working directory:

```bash
# Project dir = cwd with / replaced by - (e.g. /Users/tvwoo → -Users-tvwoo)
ls ~/.claude/projects/{project-dir}/*.jsonl
```

Run this bash command to list all session files with their dates:
```bash
for f in ~/.claude/projects/{project-dir}/*.jsonl; do
  ts=$(grep -m1 '"timestamp"' "$f" | grep -o '"timestamp":"[^"]*"' | cut -d'"' -f4)
  echo "$ts $f"
done | sort
```

### Step R-3: Identify Unprocessed Sessions

Read all files in `$KB/Bases/` and extract the `session-id:` field from each frontmatter.

Build a set of already-processed session IDs. A session is **already processed** if its JSONL filename (without `.jsonl`) matches any `session-id` in Bases/.

Sessions with no matching ID are **candidates**, regardless of date.

Report to user:
```
Found {N} session files. {M} already have Base records.
{K} unprocessed sessions:
  [1] 2026-04-07  abc123.jsonl  (~42 messages)
  [2] 2026-04-06  def456.jsonl  (~18 messages)
  ...
Process all? Or enter numbers to select (e.g. "1 3"): 
```

### Step R-4: Extract Conversation Content

For each selected session file, parse the JSONL to reconstruct the conversation:

```bash
# Extract user and assistant messages in order
jq -r 'select(.type == "user" or .type == "assistant") 
  | "\(.type | ascii_upcase): \(.message.content // "" | if type == "array" then .[0].text else . end)"' \
  {session-file}.jsonl
```

Each message should be extracted with its role (user/assistant) and content text. Ignore metadata entries (`permission-mode`, `file-history-snapshot`, tool results, etc.).

### Step R-5: Process Each Session

For each selected session, treat the extracted conversation as if it were the current session context and run the full **[SAVE PROCEDURE]** (Steps S-1 through S-6).

Key differences from normal save:
- The Base `date:` should use the session's actual date (from `timestamp`), not today
- The Base `slug:` should include the session date
- Check for duplicates against **all existing Cards** (not just today's)

### Step R-6: Report

After processing all selected sessions:
```
Replay complete.
  Sessions processed: {K}
  Cards created: {N}
  Cards updated: {M}
  Bases written: {K}
```

---

## [CHECK PROCEDURE]

Audit the entire knowledge base for broken links, inconsistencies, and data quality issues. Present findings and fix with user confirmation.

### Step C-1: Load Config

Read `~/.claude/skills/save-to-kb/.config` to get KB path.
If missing → report "Knowledge base not initialized. Run `save-to-kb init` first." and stop.

### Step C-2: Scan Cards/

List all `.md` files in `{KB}/Cards/`. For each card, check:

| Check | Issue Type |
|-------|-----------|
| Required frontmatter fields present: `title`, `slug`, `category`, `tags`, `scenario`, `created`, `updated`, `confidence` | `MISSING_FIELD` |
| `slug` matches the filename (without `.md`) | `SLUG_MISMATCH` |
| `category` value corresponds to an existing `Wiki/{category}.md` file | `BROKEN_CATEGORY` |
| Each entry in `related:` has a matching `Cards/{slug}.md` file | `BROKEN_RELATED` |
| Each entry in `sources:` has a matching `Bases/` file | `BROKEN_SOURCE` |
| Required body sections present: `## What`, `## Why It Matters`, `## How` | `MISSING_SECTION` |
| `confidence` value is one of: high, medium, low | `INVALID_CONFIDENCE` |

### Step C-3: Scan Bases/

List all `.md` files in `{KB}/Bases/`. For each base, check:

| Check | Issue Type |
|-------|-----------|
| Required frontmatter fields: `date`, `session-title`, `slug` | `MISSING_FIELD` |
| Each entry in `cards-created` and `cards-updated` has matching `Cards/` file | `BROKEN_CARD_REF` |

### Step C-4: Scan Wiki/

List all `.md` files in `{KB}/Wiki/`. For each Wiki file, check:

| Check | Issue Type |
|-------|-----------|
| All `[[Cards/slug]]` wikilinks have matching files in `Cards/` | `BROKEN_WIKILINK` |
| All `[[Wiki/category]]` wikilinks have matching files in `Wiki/` | `BROKEN_WIKILINK` |
| `card-count` in frontmatter matches the actual number of `[[Cards/` entries in the file | `COUNT_MISMATCH` |

### Step C-5: Scan Claude.md

Read `{KB}/Claude.md`. Check:

| Check | Issue Type |
|-------|-----------|
| All `[[Cards/slug]]` references have matching files in `Cards/` | `BROKEN_WIKILINK` |
| All `[[Wiki/category]]` references have matching files in `Wiki/` | `BROKEN_WIKILINK` |
| Stats `Total cards` matches actual file count in `Cards/` | `STATS_MISMATCH` |
| Stats `Total bases` matches actual file count in `Bases/` | `STATS_MISMATCH` |

### Step C-6: Report Issues

Present a structured report:

```
Health Check Report — {YYYY-MM-DD}
====================================
Cards/ ({N} files scanned)
  ✓ card-a.md
  ⚠ [#1] card-b.md — BROKEN_RELATED: slug 'missing-card' not found in Cards/
  ⚠ [#2] card-c.md — MISSING_SECTION: '## Why It Matters' not found

Bases/ ({N} files scanned)
  ✓ 2026-04-09-session.md
  ⚠ [#3] 2026-04-08-old.md — BROKEN_CARD_REF: 'Cards/deleted-card' not found

Wiki/ ({N} files scanned)
  ✓ ai-tools.md
  ⚠ [#4] frontend.md — COUNT_MISMATCH: card-count=5 but 3 actual entries found

Claude.md
  ⚠ [#5] Stats: Total cards=10, actual=8

Found 5 issues.
Fix all? Enter "yes" to fix all, or "skip 2,4" to fix all except those numbered.
```

If no issues found:
```
✓ Health check passed — {N} cards, {N} bases, {N} wiki files. No issues found.
```

### Step C-7: Apply Fixes

If user confirms, apply the following repair actions per issue type:

| Issue Type | Repair Action |
|------------|--------------|
| `BROKEN_RELATED` | Remove the broken slug from `related:` list |
| `BROKEN_SOURCE` | Remove the broken entry from `sources:` list |
| `BROKEN_CARD_REF` | Remove the broken entry from `cards-created`/`cards-updated` |
| `BROKEN_WIKILINK` | Remove the broken `[[link]]` entry from the file |
| `BROKEN_CATEGORY` | Ask user: create the missing Wiki file, or reassign card to existing category? |
| `SLUG_MISMATCH` | Update `slug:` in frontmatter to match the filename |
| `MISSING_FIELD` | Report field as missing, ask user what value to set, then write it |
| `MISSING_SECTION` | Add the missing section with placeholder text: `*(to be filled)*` |
| `INVALID_CONFIDENCE` | Set `confidence: medium` as default |
| `COUNT_MISMATCH` | Recount actual `[[Cards/` entries and update `card-count` |
| `STATS_MISMATCH` | Recount actual files and update Stats numbers in Claude.md |

After fixes, report:
```
Fixed 4 of 5 issues. Skipped: #2 (manual action required).
Run `save-to-kb check` again to verify.
```

---

## [SAVE PROCEDURE]

Save reusable knowledge from the current conversation into LLM-Brain-Bases.

### Step S-0: Load Config

Read KB path from `~/.claude/skills/save-to-kb/.config`.

### Step S-1: Extract Knowledge Points

Review the full conversation. A card must be **non-obvious** and **transferable** — meaning someone encountering a similar situation in the future would genuinely benefit from reading it.

Knowledge falls into these saveable categories:

**Technical**
- A bug's non-obvious root cause and fix
- Undocumented or surprising API / library behavior
- Integration constraints between two systems or tools
- Non-obvious performance characteristics

**Architecture & Design**
- A design decision with clear reasoning that generalizes beyond this project
- A tradeoff conclusion worth remembering (why X was chosen over Y)

**Product & Requirements**
- A non-obvious insight about user behavior or intent revealed during development
- A requirement that turned out to mean something different than assumed — and why
- A product decision pattern that resolved ambiguity in a generalizable way

**AI & Vibe Coding**
- Non-obvious Claude / LLM behavior (prompt patterns, context limits, model quirks)
- Effective Skill, Hook, or tool configuration patterns for Claude Code
- Vibe coding workflow patterns that meaningfully improve AI-assisted development
- Lessons from AI agent coordination (task splitting, context management, handoff)

**Hard rejections — do NOT save:**
- Generic process templates or documentation structures
- Decisions or bugs entirely specific to one project with zero generalizable signal
- Mainstream best practices any competent engineer already knows
- Anything that belongs in CLAUDE.md rather than a reusable card
- Unresolved explorations with no confirmed conclusion

**Judgment test**: "Would someone hitting a similar situation in the future — whether a bug, a decision, a product question, or an AI interaction — be better equipped after reading this card?" If yes, save it.

### Step S-2: Check for Duplicates

Before writing any card:
1. Read `$KB/Claude.md` — scan Knowledge Map and Scenario Quick-Lookup
2. Check the relevant Wiki file for existing entries
3. Search `$KB/Cards/` for files with similar slugs or title keywords

If a close match exists: **update the existing card** (add content, update `updated:` date, add new Base to `sources:`). Never create two cards for the same concept.

### Step S-3: Write Cards

For each knowledge point, write `$KB/Cards/{slug}.md`.

**Slug rules**: lowercase, hyphens only, no special chars, ≤60 chars.

```markdown
---
title: "{Title}"
slug: {slug}
category: {category}
tags: [{tag1}, {tag2}]
scenario: [{scenario1}, {scenario2}]
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
confidence: {high|medium|low}
related:
  - {slug-of-related-card}
sources:
  - "[[Bases/{YYYY-MM-DD}-{session-slug}]]"
---

## What
One sentence. What is this knowledge point. No hedging.

## Why It Matters
When does this matter. What breaks without it.

## How
Concrete procedure, pattern, command, or code.

## Gotchas
Edge cases, common mistakes. Omit section if none.

## Connections
Prose explaining how this relates to other concepts. Use [[wikilinks]] inline.

## References
- [[Bases/{YYYY-MM-DD}-{session-slug}]] — {one-line session description}
```

**Category options** (create new Wiki file if none fit):
`frontend` | `backend` | `infra` | `ai-tools` | `workflow` | `debugging` | `architecture` | `knowledge-management` | `business`

**Confidence**: `high` = confirmed; `medium` = partially tested; `low` = speculative.

### Step S-4: Write the Base

First, get the current session ID by running:
```bash
ls -t ~/.claude/projects/$(echo $PWD | sed 's|/|-|g' | sed 's|^-||')/*.jsonl 2>/dev/null | head -1 | xargs basename | sed 's/.jsonl//'
```
If this fails or returns empty, use `"unknown"` as the session-id.

Write one Base file at `$KB/Bases/{YYYY-MM-DD}-{session-slug}.md`. Append-only — never edit after creation.

```markdown
---
date: {YYYY-MM-DD}
session-id: {session-id-from-jsonl-filename}
session-title: "{Brief title}"
slug: {YYYY-MM-DD}-{session-slug}
project: "{Project name, or 'General'}"
tech: [{tech1}]
scenario: [{scenario1}]
cards-created:
  - "[[Cards/{slug}]]"
cards-updated:
  - "[[Cards/{slug}]]"
---

## Session Summary
2–4 sentences.

## Key Discoveries
- {Discovery} — why it matters

## Cards Written
- [[Cards/{slug}]] — description
```

### Step S-5: Update Wiki Files

For each card's `category`, update `$KB/Wiki/{category}.md`:
1. Add card under the correct tech-stack and scenario sections
2. Add to Cross-Wiki Connections table if it bridges categories
3. Increment `card-count` in frontmatter
4. Append to Recent Additions (keep last 20)

If the Wiki file doesn't exist, create it using `references/wiki-template.md`.

### Step S-6: Update Master Index

Edit `$KB/Claude.md`:
1. Update `Cards` count in Knowledge Map for affected categories
2. Add scenario entry-points to Scenario Quick-Lookup
3. Add cross-domain cards to Cross-Domain Cards
4. Prepend to Recently Added (keep last 10)
5. Update Stats (total cards, bases, last save date)

---

## What NOT to Save

- Task-specific outputs (delivered code, one-time reports)
- Preferences already in CLAUDE.md
- Pure reference facts (version numbers, general documentation)
- Incomplete explorations with no resolved conclusions

---

## [FILE PROCEDURE]

Extract reusable knowledge cards from a file (PDF, Markdown, txt, or any readable format).

### Step F-1: Load Config

Read KB path from `~/.claude/skills/save-to-kb/.config`.
If missing → report "Not initialized. Run `save-to-kb init` first." and stop.

### Step F-2: Get File Path

If invoked from **menu selection**: ask the user with AskUserQuestion:
> "请输入文件路径（支持 PDF / Markdown / txt）："

If invoked with **argument** (`save-to-kb file <path>`): use the provided path directly.

Expand `~` to the absolute home directory. Verify the file exists before proceeding.
If file not found → report the resolved path and stop.

### Step F-2b: Copy File to Knowledge Base

Copy the source file into `{KB}/Sources/` to preserve a local reference:

```bash
mkdir -p "{KB}/Sources"
cp "{source-path}" "{KB}/Sources/{filename}"
```

If the file already exists at the destination, skip copying (do not overwrite).
Report: `Source file copied to: {KB}/Sources/{filename}`

### Step F-3: Read File

Use the Read tool to read the file.
- For PDF: read all pages. If the PDF has more than 20 pages, read in batches of 20 pages.
- For text/markdown: read the full file.

### Step F-4: Extract Knowledge Points

Analyze the file content. Apply the same filter as [SAVE PROCEDURE] Step S-1 — same categories (Technical, Architecture & Design, Product & Requirements, AI & Vibe Coding), same hard rejections, same judgment test.

Before extracting, report a summary to the user:
```
File: {filename}
Pages/size: {N} pages / {size}
Candidate knowledge areas identified:
  - {area 1}
  - {area 2}
  ...
Proceed to extract cards? (yes/no)
```

### Step F-5: Check for Duplicates

Same as [SAVE PROCEDURE] Step S-2 — check Claude.md and existing Cards before writing.

### Step F-6: Write Cards

Same format as [SAVE PROCEDURE] Step S-3.

In the `sources:` field, reference the Base written in Step F-7 (use the slug you plan to write).

### Step F-7: Write the Base

Write one Base file at `$KB/Bases/{YYYY-MM-DD}-{file-slug}.md`.

Use `session-id: file:{filename}` to distinguish file-sourced bases from session-sourced bases.
Reference the local copy (`Sources/{filename}`) as the source, not the original path.

```markdown
---
date: {YYYY-MM-DD}
session-id: "file:{filename}"
session-title: "{Brief description of file content}"
slug: {YYYY-MM-DD}-{file-slug}
project: "General"
tech: [{tech1}]
scenario: [{scenario1}]
cards-created:
  - "[[Cards/{slug}]]"
cards-updated: []
---

## Source

File: `[[Sources/{filename}]]`
Original path: `{original-source-path}`
{2–4 sentences describing what the file covers and why it was imported.}

## Key Discoveries
- {Discovery} — why it matters

## Cards Written
- [[Cards/{slug}]] — description
```

### Step F-8: Update Wiki and Master Index

Same as [SAVE PROCEDURE] Steps S-5 and S-6.

### Step F-9: Report

```
File analysis complete.
  Source: {filename}
  Cards created: {N}
  Cards updated: {M}
  Base written: {slug}
```

---
> Source: [verycafe/save-to-kb](https://github.com/verycafe/save-to-kb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
