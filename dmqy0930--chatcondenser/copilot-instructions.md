## chatcondenser

> Use this skill when the user wants to convert a DeepSeek web chat (via share link) into a structured review outline for Obsidian knowledge management. It restructures Q&A into a logical knowledge framework, removes conversational noise, and outputs interlinked Markdown notes with wikilinks and YAML frontmatter. Triggered by DeepSeek share links or explicit requests to summarize learning-oriented AI chats.


# ChatCondenser — DeepSeek Chat → Obsidian 知识网络

Convert a messy, multi-turn DeepSeek learning conversation into a structured, interlinked Obsidian knowledge network. The output is not a chat log or FAQ list—it is a Map of Content (MOC) main note plus optional concept sub-notes, connected by `[[wikilinks]]` for bidirectional navigation.

## Non-Negotiables

- **REQUIRED COMPANION SKILL:** use `pdf` only for PDF-based source input (e.g., exported chat PDFs, screenshots). For plain share links or pasted text, `pdf` is not needed.
- **NO OTHER SKILLS:** when this skill applies, use only `chatcondenser` plus `pdf`. Do **not** invoke any other skill. Any task-splitting, verification, or dispatch logic is already defined inside this skill.
- **Primary deliverable is `.md` files in the Obsidian vault.** PDF compilation (`xelatex`) is strictly optional—only run it when the user explicitly requests a PDF export.
- When this skill applies, a chat-only or inline summary is **not** a successful final deliverable unless the user explicitly asks for summary-only output or opts out of file creation.
- **All `.md` files must be written to `{vaultPath}/{targetFolder}/`** (as configured in frontmatter metadata). Do not write notes anywhere else.
- The vault path is read from this SKILL.md's `metadata.vaultPath` field. If it is missing or points to a non-existent directory, stop and ask the user to set it.
- The input is primarily a **DeepSeek share link** (e.g., `https://chat.deepseek.com/share/...`). Use `WebFetch` to retrieve its content. If the link cannot be fetched, accept raw pasted text as a fallback.
- The default writing style is **Chinese-first**, with important technical terms followed by their English equivalents in parentheses.
- The output is a **review outline**, not a chronology. The original Q&A format must be completely dissolved and restructured by topic and logic.
- **No star ratings or decorative icons.** Use basic text formatting: bold, italic, `==highlight==`, Obsidian callouts (`> [!note]`), and Markdown tables.
- Every key concept must link to other notes via `[[wikilinks]]` for knowledge networking.
- All notes must include YAML frontmatter with at least `tags` and `created` fields.

## Skill Boundary

- Allowed skills for this workflow: `chatcondenser` and `pdf` only.
- Forbidden: all other skills, even if they appear relevant to planning, dispatch, or verification.

## Required Outcome

The default successful outcome for this skill is:

- a **main note (MOC)** saved as `Chat-<Topic>.md` inside `{vaultPath}/{targetFolder}/`
- zero or more **concept sub-notes** (stubs for independently meaningful concepts), each with bidirectional `[[links]]` back to the MOC
- all notes contain valid YAML frontmatter (`tags`, `created`, `source`) and Obsidian wikilinks
- a final response that reports the written file paths, linking coverage, and any blockers

**Optional:** if the user explicitly requests PDF export, additionally produce `Chat-<Topic>.pdf` via `xelatex` and report the PDF path.

The following are **not** successful completions:
- an inline chat summary only
- a document that still retains a Q&A or conversational structure
- stopping at analysis without writing `.md` files to the vault
- generating files outside `{vaultPath}/{targetFolder}/`
- wikilinks that point to non-existent notes without creating stubs (when `autoCreateStubs: true`)

## When to Use

Use this skill when the user wants to turn a learning chat with DeepSeek into Obsidian revision material. Typical triggers:

- User provides a `chat.deepseek.com/share/...` link and asks to "整理成复习提纲", "提炼知识点", "做成笔记"
- User pastes a long DeepSeek conversation and wants structured, interlinked notes
- User mentions "对话总结", "聊天记录提炼", "把和DeepSeek的对话变成Obsidian笔记"

Do not use this skill for:
- summarizing meetings, interviews, or non-learning conversations
- turning a chat into an FAQ or Q&A list (unless the user explicitly asks)
- processing chats from platforms other than DeepSeek (unless the user explicitly confirms format compatibility)
- producing PDF-only output when the user has not asked for PDF

## Portability

- Do not assume any custom skill other than `pdf` exists.
- This skill reads its own configuration from the `metadata` fields in its YAML frontmatter.
- If subagents are available, prefer them for topic-clustering readers (Step 4) and the verifier (Step 11). If not, execute sequentially.
- `WebFetch` is the preferred method for retrieving DeepSeek share link content.

## Workflow Model

The workflow follows a controller-reader-verifier architecture specialized for conversational content and Obsidian output:

```
Intake → Fetch & Clean → Topic Clustering → Correction Chain →
Wikilink & Split → Restructure & Tables → Glossary → Merge →
Verify → Write .md files → (Optional PDF) → Report
```

## Step-by-Step Execution

---

### Step 1. Intake

- Accept a DeepSeek share link (e.g., `https://chat.deepseek.com/share/...`) or raw pasted text.
- **Read vault configuration:** parse the `metadata.vaultPath`, `metadata.targetFolder`, and `metadata.autoCreateStubs` fields from this SKILL.md. Stop if `vaultPath` is missing or the directory does not exist.
- Construct the output path: `{vaultPath}/{targetFolder}/`. Create `{targetFolder}/` if it does not exist.
- Scan the vault for existing `.md` notes to build a **note index** for later wikilink resolution (Step 6). This can be a lightweight scan: collect all `.md` filenames (without extension) under the vault root. Skip system directories like `.obsidian/`, `.trash/`, `.git/`.
- Determine the conversation topic from the share link title or the first meaningful user question. Use this as the `<Topic>` stem.

---

### Step 2. Fetch & Parse the Conversation

- If input is a share link, use `WebFetch` to retrieve the page content. Parse the conversation into structured turns.
- If `WebFetch` fails (blocked, empty, or non-text), request the user paste the conversation text directly.

**Turn classification** — label every turn with one of:

| Type | Speaker | Description | Action |
|------|---------|-------------|--------|
| `NEW_QUESTION` | User | Asks about a topic new to the conversation | Start a new topic cluster |
| `FOLLOW_UP` | User | Deepens or refines the current topic | Merge into current topic cluster |
| `CORRECTION` | User | Points out an error in the AI's answer | Mark prior AI claims on this topic for review |
| `CLARIFICATION` | User | Requests simpler/different explanation | Merge into current topic |
| `DIGRESSION` | User | Off-topic or tangential question | Isolate; assign to appendix if substantive |
| `GREETING` | User | "hello", "thanks", "bye" | **Discard** |
| `CONFIRMATION` | User | "is that right?", "anything else?" | **Discard** unless it triggers new AI content |
| `AI_ANSWER` | AI | Normal informative response | Extract knowledge claims |
| `AI_CORRECTION` | AI | Explicitly acknowledges an error and provides a revised answer | Supersedes prior AI_ANSWER on this topic |
| `AI_FRAMING` | AI | "That's a great question!", "Let me explain..." | **Discard** (conversational noise) |

**FOLLOW_UP vs NEW_QUESTION:** If the user asks "那这个呢？" referring to a related concept, classify as `FOLLOW_UP`. If the follow-up is semantically a new topic (unrelated subject), classify as `NEW_QUESTION`.

---

### Step 3. Noise Removal & Cleaning

For each classified turn:

1. **Discard** all `GREETING` and pure `CONFIRMATION` turns.
2. **Strip** `AI_FRAMING` phrases from `AI_ANSWER` and `AI_CORRECTION` turns. Remove: conversational openers, self-praise, filler transitions, signoffs. Keep the factual content intact.
3. **Normalize** formatting: remove excessive newlines, collapse repeated punctuation, unify quote styles.
4. Save the cleaned conversation internally. Do not write intermediate files to disk outside `{vaultPath}/{targetFolder}/`. If a helper file is absolutely necessary, place it in the OS temp directory and delete it before final response.

---

### Step 4. Topic Extraction & Clustering

**Phase 1 — Extract topic seeds:**
- Each `NEW_QUESTION` starts a topic cluster.
- `FOLLOW_UP` and `CLARIFICATION` turns attach to the most recent `NEW_QUESTION` cluster.
- `CORRECTION` + the corresponding `AI_CORRECTION` attach to the topic being corrected.
- `DIGRESSION` turns are set aside. After clustering, evaluate each for substantive knowledge content. If it contains meaningful information, place it in an "附录：补充讨论" appendix section. Otherwise discard.

**Phase 2 — Build the knowledge tree:**
- Merge related topic clusters that cover the same concept at different depths.
- Order topics logically: foundational → advanced, or by domain taxonomy.
- For cross-domain conversations, the agent decides whether to split into multiple independent notes or structure as a single note with domain-specific subheadings. Guidelines:
  - If domains are clearly separable (e.g., math + history) → split into `Chat-Math-<Topic>.md` and `Chat-History-<Topic>.md`, cross-linked.
  - If domains are interleaved and mutually dependent → keep as one MOC.
- Assign a **priority tier** to each topic:
  - `CORE`: central to the conversation, high detail
  - `SUPPORTING`: related but not central, compressed
  - `MARGINAL`: mentioned once, one-line summary or footnote

---

### Step 5. Correction Chain Resolution

Scan each topic cluster for conflicting AI statements.

**Conflict detection:** two `AI_ANSWER` or `AI_CORRECTION` turns within the same topic cluster that make contradictory factual claims.

**Resolution priority (highest to lowest):**

1. `AI_CORRECTION` (AI explicitly acknowledges error) → the corrected answer is **authoritative**.
2. `CORRECTION` by user + subsequent `AI_ANSWER` that adopts the correction without explicit acknowledgement → the post-correction answer is authoritative.
3. Last `AI_ANSWER` in the cluster → preferred over earlier answers when no explicit correction occurred.
4. More detailed `AI_ANSWER` from a `FOLLOW_UP` → preferred over the initial brief answer.

**"常见误区" (Common Pitfall) rule:** When a `CORRECTION` → `AI_CORRECTION` chain exists, preserve the original incorrect answer as an educational note. Format:

```markdown
> [!warning] 常见误区
> （对话中 AI 最初回答为：XXX，经用户纠正后修正。需注意区分。）
```

This adds pedagogical value for the reader who may share the same misconception.

**Uncertainty flagging:** If the AI uses hedging language ("可能", "一般", "通常", "I think..."), mark the claim with a confidence note. Do not invent certainty.

---

### Step 6. Wikilink Generation & Note Splitting

This step produces the Obsidian knowledge network.

**Part A — Build the main MOC note structure:**

```
Chat-<Topic>.md
  ├── YAML frontmatter (tags, created, source)
  ├── 目录
  ├── Topic sections (restructured outline)
  │   ├── Concepts with [[wikilinks]]
  │   └── 📎 相关概念 (related links block)
  └── 术语表
```

**Part B — Identify candidate concept sub-notes:**

A concept qualifies for an independent sub-note when it meets at least 2 of:
- Has a clear, complete definition from the conversation
- Has ≥ 2 distinct knowledge points (mechanism, example, application, etc.)
- Has ≥ 1 natural cross-reference to another concept
- Appears in ≥ 3 conversation turns (high engagement)

**Part C — Generate wikilinks and stubs:**

For every key concept mentioned in the MOC:
1. Look up the concept name (Chinese) in the vault note index (collected in Step 1).
2. If found → generate `[[exact-filename]]`.
3. If not found + `autoCreateStubs: true` → create a stub `{概念名}.md` in `{vaultPath}/{targetFolder}/` with minimal YAML frontmatter and a backlink to the MOC, then generate `[[概念名]]`.
4. If not found + `autoCreateStubs: false` → do not create a link; use bold text instead.

**Stub note template:**

```markdown
---
tags: [deepseek-chat, <领域标签>]
created: <today>
source: "[[Chat-<Topic>]]"
---

# <概念名>

> 来源：[[Chat-<Topic>|DeepSeek 对话：<主题>]]

<从对话提取的定义>

## 相关笔记
- [[Chat-<Topic>]]
```

**Part D — Bidirectional linking:**
- MOC links → each concept sub-note via `[[概念名]]`.
- Each concept sub-note links ← back to the MOC via `[[Chat-<Topic>]]`.
- If concept sub-notes reference each other, add cross-links in their "相关笔记" section.

---

### Step 7. Restructure into Outline

Dissolve all Q&A formatting and build a textbook-style outline.

**For each topic cluster (in knowledge tree order):**

```markdown
## [[概念名]]
### 定义
> 精确定义（来自对话最终确认版本）
### 关键要点
- 要点 1（附英文术语）
- 要点 2（附关键技术参数或关键数值）
### 示例 / 应用
（如有对话中的实例）
> [!warning] 常见误区
（如有纠错链，放入此 callout）
### 📎 相关概念
- [[关联概念A]] | [[关联概念B]]
```

**CORE topics:** expand with full definition + all key points + examples + pitfalls.
**SUPPORTING topics:** compressed definition + 1-2 key points.
**MARGINAL topics:** one-line summary in a footnote or "其他讨论" appendix.

---

### Step 8. Generate Comparison Tables

When the conversation compares two or more concepts (e.g., "X vs Y", "what's the difference between..."), convert the comparison into a Markdown table:

```markdown
| 维度 | [[概念A]] | [[概念B]] |
|------|----------|----------|
| 核心差异 | ... | ... |
| 适用场景 | ... | ... |
```

Do not fabricate comparisons that the conversation did not discuss.

---

### Step 9. Glossary Module

Extract a terminology table from all topic clusters:

```markdown
| 术语 | English | 释义 |
|------|---------|------|
| 概念A | Concept A | 一句话定义 |
```

- Include every term that has an English equivalent mentioned in the conversation.
- Place the glossary at the end of the MOC, before the appendix.
- Each term in the glossary should be a `[[wikilink]]` if a note exists or a stub was created.

---

### Step 10. Merge & Finalize Markdown Draft

Merge all sections into a single MOC Markdown draft.

**Draft quality rules:**
- No `AI_FRAMING` phrases anywhere.
- No Q&A markers ("Q:", "A:", "User:", "Assistant:").
- All headings use `#` through `####` hierarchy (Obsidian outline panel compatible).
- All internal links use `[[双括号]]` format.
- YAML frontmatter is complete and valid at the top of the main note.
- Sections are ordered: 目录 → CORE topics → SUPPORTING topics → 对比辨析 → 术语表 → 附录

---

### Step 11. Verification Gate

Run a fresh verifier pass with fresh context. The verifier must check against the cleaned conversation text (not the draft alone).

**Verifier output:**

```
verdict: APPROVED | REJECTED

coverage:
  - All NEW_QUESTION topics present in draft? Yes/No (list missing)
  - All CORRECTION chains resolved to final authoritative answer? Yes/No
  - Any DIGRESSION with substantive content lost? Yes/No

fidelity:
  - All definitions match the FINAL corrected version? Yes/No (list discrepancies)
  - Any mid-correction wrong answers present as fact? Yes/No
  - Any external knowledge not present in the conversation? Yes/No

structure:
  - Topic-based outline or Q&A format? (latter → REJECTED)
  - Topic ordering follows foundational → advanced? Yes/No
  - Conversational noise (framing phrases) remaining? Yes/No (list examples)

wikilinks:
  - All key concepts have [[wikilinks]]? Yes/No (list unlinked)
  - All created stubs have bidirectional backlinks? Yes/No
  - Any [[links]] pointing to non-existent notes without stubs? Yes/No

style:
  - YAML frontmatter valid (tags, created present)? Yes/No
  - No star ratings or decorative icons? Yes/No
  - English terms present for key concepts? Yes/No
  - Common pitfalls callouts present where correction chains existed? Yes/No
```

Proceed to writing only on `APPROVED`. If `REJECTED`, fix the draft and re-verify.

---

### Step 12. Write Files to Vault

Write all files to `{vaultPath}/{targetFolder}/`:

1. **Main MOC:** `Chat-<Topic>.md`
2. **Concept stubs:** `{概念名}.md` for each auto-created stub
3. **Do not** write intermediate files (cleaned text, topic maps, verifier notes) to the vault. If needed, use OS temp directory and delete after.

**Before writing each file,** verify that no existing note with the same name would be silently overwritten. If a file already exists, append a numeric suffix (`Chat-<Topic>-2.md`) and log a warning in the final response.

---

### Step 13. (Optional) PDF Export

Only run this step if the user explicitly requests a PDF.

- Use `xelatex` for compilation. If `xelatex` is not available, report the blocker and deliver only the `.md` files.
- Write `Chat-<Topic>.tex` to `{vaultPath}/{targetFolder}/`, compile, and place `Chat-<Topic>.pdf` in the same folder.
- LaTeX preamble: use `ctexart` for Chinese, `hyperref` for links, `longtable`/`booktabs`/`colortbl` for tables, `xcolor` for color.
- Map Obsidian-specific syntax to LaTeX:
  - `[[wikilinks]]` → plain text (or `\href` if internal PDF linking is feasible)
  - `==highlight==` → `\colorbox{yellow}{...}`
  - `> [!note]` / `> [!warning]` → `tcolorbox` environments
- Clean up `.aux`, `.log`, `.out`, `.toc` after successful compilation.

---

## Style Rules

**Markdown / Obsidian:**
- Chinese-first, concise bullets (not prose paragraphs).
- Key terms in English follow first occurrence in Chinese: `关键术语 (Key Term)`.
- Use `**bold**` for critical definitions and parameters.
- Use `==highlight==` for exam-level emphasis.
- Use `> [!note]` for supplementary context.
- Use `> [!warning]` for common pitfalls / misconceptions.
- Use `> [!info]` for cross-references to related topics.
- Headings: `#` (title), `##` (chapter), `###` (section), `####` (sub-section). No deeper than `####`.
- Tables: Markdown native format, concise headers, compact cells.
- No `★`, `☆`, `▲`, `※`, emoji, or decorative bullets.

**LaTeX / PDF (if requested):**
- A4, 2 cm margins, `ctexart`, 12pt, 1.5 line spacing.
- `hyperref` with `linkcolor=blue`.
- Tables: three-line style (`booktabs`), bold header, alternating light-gray rows (`\rowcolors{1}{}{LightGray}`).
- Callout boxes: `tcolorbox` with defined colors.
- Footer: centered page number only.

## Output Naming

- Main MOC: `Chat-<Topic>.md`
- Concept stub: exact concept name as filename, e.g., `子宫内膜异位症.md`
- PDF (optional): `Chat-<Topic>.pdf`
- All files in: `{vaultPath}/{targetFolder}/`
- If filename collision: append `-2`, `-3`, etc.

## Common Failure Modes

- Preserving Q&A or conversational structure instead of restructuring into a topic outline.
- Leaving `AI_FRAMING` phrases ("这是一个好问题", "让我来详细解释") in the final notes.
- Keeping a mid-correction wrong answer as the main definition.
- Discarding CORRECTION chains entirely instead of preserving them as "常见误区".
- Failing to generate `[[wikilinks]]` for key concepts.
- Generating `[[wikilinks]]` but not creating stub notes when `autoCreateStubs: true`.
- Creating stub notes without bidirectional backlinks to the MOC.
- Missing YAML frontmatter or invalid `tags`/`created` fields.
- Writing files to a directory other than `{vaultPath}/{targetFolder}/`.
- Writing intermediate files (cleaned text, topic maps) to the vault.
- Silently overwriting existing vault notes with the same name.
- Treating `FOLLOW_UP` as `NEW_QUESTION` and fragmenting a single topic.
- Skipping the verifier and writing an unverified draft.
- Using `> [!note]` as a substitute for proper restructuring.
- Failing to batch wikilinks: every concept in the glossary must link back to its section.

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "The Q&A format is clearer for readers" | A topic-structured outline is the core deliverable. Q&A must be dissolved. |
| "The AI's first answer was correct enough" | If a CORRECTION chain exists, only the final corrected version is authoritative. |
| "I don't need to create stubs, the user can do it later" | When `autoCreateStubs: true`, stub creation is part of the workflow. |
| "I can summarize the conversation in one pass" | Long conversations need clustering first, or coverage will drift. |
| "English terms aren't important" | Bilingual labeling is a core deliverable. |
| "The topic order in the chat is fine" | Restructure by knowledge logic (foundational → advanced), not chronology. |
| "I'll skip the verifier, the draft looks good" | Unverified drafts risk silent errors. The verifier is mandatory. |
| "Writing to the vault feels risky, I'll use summary/ instead" | The skill config explicitly specifies `{vaultPath}/{targetFolder}/`. |
| "I already have the draft in mind, so the task is done" | Files must actually be written to the vault and verified. |

## Red Flags

All of these mean: **stop, fix, and re-verify before finalizing.**

- Q&A format preserved in the final outline
- No topic clustering performed (conversation treated as one flat narrative)
- No `[[wikilinks]]` in the final .md files
- Stub notes promised but not created on disk
- Stub notes created but without `source: "[[Chat-<Topic>]]"` backlink
- No YAML frontmatter in output notes
- Intermediate files left in the vault or source directory
- Final response claimed completion before verifying file existence on disk
- Conversation noise (framing phrases, greetings) visible in output
- MID-CORRECTION wrong answer used as the authoritative definition
- Verifier skipped
- `NEW_QUESTION` topic missing from the final draft with no documented reason
- Files written to a directory other than `{vaultPath}/{targetFolder}/`

---
> Source: [dmqy0930/chatcondenser](https://github.com/dmqy0930/chatcondenser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
