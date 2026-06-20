## skill-digest

> |


# Skill Digest

Analyze any agent skill and present it in a standardized digest format.

## When to Use

- User installs a new skill and wants to understand it quickly
- User wants to understand how a skill works before using it
- User asks to compare, quiz, or practice with a skill

**When NOT to use:**
- User already knows the skill and just wants to run it
- Quality evaluation (use rate-skill)
- Security auditing (use skill-auditor)

---

## Input Resolution

Resolve the argument to a SKILL.md before analysis. Support all of the following:

| Input form | Example | Resolution |
|------------|---------|------------|
| Local skill name | `is`, `skill-digest` | Look up `{skills-dir}/{name}/SKILL.md` where `{skills-dir}` is the host's skills directory (e.g. `~/.claude/skills/` on Claude Code) |
| Local path | `<skills-dir>/is/SKILL.md` | Read directly |
| GitHub full URL | `https://github.com/user/repo/blob/main/skills/foo/SKILL.md` | Convert to raw URL and fetch |
| GitHub short form | `user/repo` or `user/repo/path/to/skill` | Fetch from `https://raw.githubusercontent.com/user/repo/main/{path}/SKILL.md` |
| GitHub org/collection | `vercel-labs/skills` | Fetch index, find all SKILL.md files |

**GitHub URL conversion rule:**
- `https://github.com/A/B/blob/BRANCH/PATH` → `https://raw.githubusercontent.com/A/B/BRANCH/PATH`
- Short form `user/repo` with no path → treat as collection (see Multi-Skill Resolution below)
- Short form `user/repo/path` → treat as single skill at that path

**Resolution priority:** local file → local skills dir → GitHub fetch

---

## Multi-Skill Resolution

If the resolved target contains **more than one SKILL.md** (e.g., a GitHub repo or directory with multiple skills):

1. List all discovered skills with their one-line descriptions
2. Immediately ask:

```
Found N skills in [source]:
  1. skill-foo — [one-line from frontmatter description]
  2. skill-bar — [one-line from frontmatter description]
  3. skill-baz — [one-line from frontmatter description]

Analyze:
[A] All — collection overview digest
[B] Pick one — enter number
```

- If user picks **[A]**: output Collection Digest format (see below)
- If user picks **[B]**: output Single Skill Digest format for that skill

---

## Localization

Detect the user's language from the conversation. If not English:

- **Keep the `◆` prefix** on every `##` section — it is a language-independent visual marker, never translate or drop it.
- **Translate the heading text only** (e.g., `## ◆ Steps` → `## ◆ 단계`, `## ◆ Watch Out` → `## ◆ 주의사항`).
- **Translate the `Key:` sentence** into the user's language while keeping the `**Key:**` label (or its localized equivalent) bold.
- **Write all body text** in the user's language.
- **Keep code blocks, skill names, and CLI examples** in their original form — do not translate identifiers, command names, or flag names.
- **Keep the compact menu labels** (`[1] Quiz`, `[2] Deep Dive`, etc.) in the same language as the rest of the output.

**Section heading translation reference (Korean):**

| English | Korean |
|---------|--------|
| Key | 핵심 |
| When | 언제 쓰나요 |
| Steps | 단계 |
| Watch Out | 주의사항 |
| Tips | 알아두면 좋은 것 |
| Try It | 사용 예시 |
| Related | 관련 스킬 |
| Skills at a Glance | 스킬 한눈에 보기 |
| Common Paths | 자주 쓰는 조합 |
| Quick start | 빠른 시작 |
| Learn more | 더 알아보기 |
| More details | 세부 정보 |
| What next? | 다음 단계? |

For other languages, derive equivalent translations. Do not use the English headings when the user's language is detected as non-English.

## README Integration

Before outputting the digest, check if a `README.md` exists alongside `SKILL.md` (same directory):

- If found: read it and **merge any additional context** not covered in SKILL.md — e.g., installation steps, known limitations, changelog, or extended examples
- Surface README-only information in the appropriate digest sections (extra TIPs, extra WARNs, or a new `## Setup` section if installation steps are present)
- If README contradicts SKILL.md, prefer SKILL.md for behavioral rules; prefer README for setup/environment facts

---

## How It Works

1. **Resolve input** — determine source (local / GitHub), fetch SKILL.md
2. **Check for README** — read README.md from same directory if present
3. **Check for multiple skills** — if collection, ask All vs Pick (see Multi-Skill Resolution)
4. **Read skill files** — SKILL.md and all auxiliary files (reference/, rules/, scripts/, examples/)
5. **Analyze** — in this exact order, do not skip:
   a. Distill the single most load-bearing sentence about the skill → this becomes the `Key:` line
   b. Classify type (see Classifying Type)
   c. Classify complexity (see Classifying Complexity)
   d. Identify 1–3 `When` rescue scenarios (pain → what the skill does)
   e. Extract primary workflow as numbered sequence (3–7 steps, no sub-bullets)
   f. Identify warnings (⚠) — **only those explicitly documented** in SKILL.md / README. Search for phrases like "주의", "warning", "do not", "must not", "절대로 ~ 금지", "irreversible", "will silently". Quote or paraphrase the exact documented warning. **Do not infer warnings from the skill's architecture** — if docs don't warn, assume the skill already handles it. Fewer true warnings > more invented ones.
   g. Identify tips (💡) — **only those explicitly documented** as user-facing shortcuts, mode-switches, or invocation tricks. If the docs don't describe a shortcut, do not invent one.
   h. Check "When NOT to Use" for redirect skills — note as related
6. **Output in the appropriate digest format** (Single or Collection)
7. **Show only `→ [1] Learn more`** on the first output (progressive disclosure — expand to the full 5-item menu only when the user picks it)

---

## Single Skill Digest Format

Always output in this exact structure. Respond in the user's language.

The ◆ prefix on every section is mandatory — it makes the digest visually distinguishable from surrounding conversation. Do not drop it even when translating headings.

```
# [skill-name] — [15-word-max description of what it does]

> **Key:** [one bold sentence — the single reason this skill exists. This is what the user must remember 10 minutes later.]

## ◆ When
• [painful situation] → [what this skill does]
• [painful situation] → [what this skill does]
• [third — omit if not meaningfully distinct]

Quick start: [shortest invocation example]

## ◆ Steps
1. **[Verb]** — [what happens, what you control]
2. **[Verb]** — [what happens, what you control]
...

## ◆ Watch Out
⚠ [specific condition → what breaks, how to avoid it]
⚠ [specific condition → what breaks, how to avoid it]

## ◆ Tips
💡 [non-obvious shortcut or pattern]
💡 [integration or efficiency note]

---

<details>
<summary>More details</summary>

## ◆ Try It
```[lang]
[copy-paste ready example]
```
[One sentence: what this example demonstrates.]

## ◆ Related
[skill-name] — [one-line relationship description]

`[Type]` · `[Complexity]` · needs: [prerequisites or "nothing extra"]
</details>

→ [1] Learn more
```

**Section rules:**
- `> **Key:**` line is mandatory — one bold sentence naming what this skill automates, prevents, or accelerates. Place immediately after the title.
- `## ◆ When`: 1–3 bullet entries. Each = [situation that causes pain] → [what the skill does]. Followed by one-line `Quick start:` showing shortest invocation. Omit `Quick start:` if no clear single-command entry point exists.
- Use `•` bullets (not `▎` or `-`) throughout for visual consistency.
- `## ◆ Watch Out`: **documented-only, no inference.** Include an entry only when the skill's own SKILL.md (or README) explicitly warns about the behavior — a sentence like "주의", "warning", "do not", "must not", "절대로 ~ 금지", "will silently ~", "irreversible", etc. Quote or paraphrase that explicit warning. **Do NOT derive warnings from your own reasoning about the skill's architecture** (e.g. "parallel execution could cause conflicts" when the skill doesn't say so — it may already mitigate that). If the skill's docs do not explicitly warn about something, **omit the section entirely** rather than fabricate. LLMs cannot reliably infer a skill's real hazards from reading the workflow, and invented warnings erode user trust. Maximum 3 entries. Each entry starts with `⚠ ` (warning sign + space). Must also be **user-facing** (visible while invoking the skill), not internal execution rules for the agent.
- `## ◆ Tips`: **documented-only, no inference.** Include only tips, shortcuts, or mode-switching notes that the skill's own SKILL.md (or README) explicitly describes. If the docs don't spell out a shortcut, do not invent one. Maximum 3 entries. Each entry starts with `💡 ` (lightbulb + space). Must be **user-facing** (input tricks, invocation modes), not internal implementation trivia. Omit the section entirely if no user-facing tips are explicitly documented.
- `## ◆ Related`: only include if there is clear directional relationship. Do not list every skill with a similar name.
- `Try It` / `Related` / metadata line live inside the collapsible `<details>` block — progressive disclosure. Do not promote them above the divider.
- The `◆` prefix is mandatory on every `##` section — it is how the user recognizes "this is a digest block" vs. ordinary conversation.
- Never write placeholder text ("No warnings", "No tips") — omit the section.
- Section body text (not the `◆` prefix): translate to user's language per Localization rules.

**Menu progressive disclosure:**
- First-time output shows only `→ [1] Learn more` (Hick's law — one choice, zero decision cost).
- When the user picks `[1]`, expand to the full menu:
  ```
  → [1] 🧩 Quiz  [2] 🔬 Deep Dive  [3] 🎯 Practice  [4] ⚖️ Compare  [5] 🗺️ Skill Map
  ```
- After completing any follow-up action, show the full compact menu again (not `Learn more`).

---

## Collection Digest Format

Use when analyzing a GitHub repo or directory containing multiple skills.

```
# [collection-name] — [N] skills
> Source: [URL or path]

## ◆ When
• [situation where this collection as a whole helps]
• [second scenario — omit if not distinct]

## ◆ Skills at a Glance
| Skill | Type | Complexity | One-liner |
|-------|------|------------|-----------|
| [name] | [type] | [level] | [≤10 words] |
| [name] | [type] | [level] | [≤10 words] |

## ◆ Common Paths
[skill-a] → [skill-b]  "[when to chain these]"
← only include if SKILL.md files have explicit chain evidence

---
What next?
[A] 📦 Analyze all — deep digest for every skill (may be slow)
[B] 🔍 Pick one — enter skill name or number
[5] 🗺️ Skill Map — collection-wide layout
```

**Collection rules:**
- Table: all skills listed, sorted by complexity (Simple first)
- `## ◆ Common Paths`: only if SKILL.md files explicitly mention chaining. Do not invent from name similarity.
- `## ◆ When`: applies to collection as a whole, not individual skills.
- `◆` prefix is mandatory on every `##` section, same as the Single Skill template.

---

## Analysis Guidelines

### Classifying Type

| Type | Signals |
|------|---------|
| **Methodology** | Phases, gates, checklists, process steps, verification steps |
| **Rule Catalog** | Priority tables, rule lists, rules/ directory, DO/DON'T patterns |
| **Tool-Reference** | Commands, scripts/, setup instructions, API patterns, CLI usage |

### Classifying Complexity

| Level | Criteria |
|-------|----------|
| **Simple** | Under 100 lines, 1–3 step workflow, no prerequisites |
| **Moderate** | 100–400 lines, multi-step workflow, some reference files |
| **Advanced** | 400+ lines or many reference files, multi-phase process, external dependencies |

### Writing "Use this when"

Trigger condition, not a persona description:
- Good: "when you're shipping and want to verify the deployment didn't break login"
- Bad: "when you need a QA skill"

### Writing "When" bullets

Pain-first, not feature-first. Each entry = a moment the user is stuck → what this skill does:
- Good: `• A teammate says "try this skill" but you worry one wrong flag will wipe data → the Watch Out section flags the worst mistakes before you hit them`
- Bad: `• When you need skill analysis functionality`

### Writing "Watch Out" entries

**Two tests, both must pass:**

1. **Documented test**: can you point to an explicit warning phrase in the skill's own SKILL.md / README? (e.g. "do not", "절대로 ~ 금지", "will silently", "irreversible"). If you cannot cite the source, **omit the entry**.
2. **Perspective test**: does the user see this while *invoking* the skill, or does only the agent see it while *executing*? If only the agent sees it, **omit the entry**.

Fabricating warnings from your own analysis of the skill's architecture is explicitly forbidden. An empty `## ◆ Watch Out` section is better than an invented one — LLMs cannot reliably infer real hazards.

- Good (documented + user-facing): `⚠ Any digit inside a natural-language prompt flips the skill from "create new issue" mode to "reference existing issue #N" mode — "fix bug 91" will try to process issue #91, not create a new one`   *(skill docs explicitly warn about this mode switch)*
- Good (documented + user-facing): `⚠ "--merge" is hard-coded for PR closing, not "--rebase" — your main branch will show merge commits regardless of your repo's usual convention`   *(skill docs state "반드시 --merge를 사용한다")*
- Bad (internal execution rule, invisible to user): `⚠ Sub-agents must never call gh issue create — only the orchestrator may`
- Bad (inferred from architecture, not documented): `⚠ Parallel sub-agents could race on the same files` — the skill uses per-issue worktrees and does not warn about this; LLM invented the risk.
- Bad (internal retry policy, user doesn't invoke retries): `⚠ Test failure retries are capped at 3 before labeling need-test`

### Writing "Tips" entries

Same two tests — must be documented **and** user-facing. No invented shortcuts.

- Good (documented input shortcut): `💡 Use "pr 45" to resume unfinished feedback on a PR without remembering the branch name`   *(docs explicitly list this mode)*
- Good (documented mode-switching gotcha): `💡 Pure natural language with zero digits = create-new-issue mode; any digit at all = reference-existing mode`   *(docs explicitly state this rule)*
- Bad (implementation trivia, not user-facing): `💡 Each sub-agent runs in /tmp/issue-N worktree for isolation`
- Bad (invented — not in docs): `💡 You can run /is during off-hours for faster response`

---

## Follow-up Actions

### [1] Quiz

Generate exactly 3 questions with fixed type distribution:

- **Q1 Recognition** — "Which of these would you use this skill for?" (4 options, one correct)
- **Q2 Workflow** — "Put these steps in correct order:" (shuffle 3–4 steps from the digest)
- **Q3 Gotcha** — derived from a specific WARN entry (open-ended, one-sentence answer)

Format:
```
## Quiz: [skill-name]

Q1. [Recognition]
{question}
  A) ...  B) ...  C) ...  D) ...
<details><summary>Answer</summary>Answer: {X} — {one sentence why}</details>

Q2. [Workflow]
Put these steps in correct order: {shuffled list}
<details><summary>Answer</summary>Correct order: {sequence} — {one sentence why}</details>

Q3. [Gotcha]
{question from a real WARN entry}
<details><summary>Answer</summary>{one sentence answer referencing skill content}</details>
```

After quiz, show compact menu.

### [2] Deep Dive

Do NOT present all possible sub-topics. Instead:

1. Identify the 2–3 most non-obvious aspects of this specific skill. Selection priority:
   - Behavior that contradicts user intuition
   - A step the workflow summarized but didn't explain
   - A failure mode or edge case unique to this skill
   - A non-obvious integration with another skill
2. Present only those as lettered choices:
   ```
   Which aspect?
   [A] {most distinctive mechanic}
   [B] {second most non-obvious aspect}
   [C] {edge case or integration — only if genuinely distinct}
   ```
3. After user picks: 150–300 word focused explanation with one concrete example.
4. If you cannot identify 2+ non-obvious aspects: say "This skill is straightforward — the digest covers all key behavior." Do not invent generic topics.

After explanation, show compact menu.

### [3] Practice

Generate one scenario. Constraints:
- Software development context (not abstract)
- Requires this specific skill — not solvable without it
- 3–5 sentences: situation, what went wrong or what is needed, what the user faces
- End with: "What would you do?" — open-ended, no sub-steps, no hints

After user responds, evaluate against the skill's actual workflow:
- One sentence: what they got right
- One sentence: what was missing or wrong (if anything)
- Correct approach in 2–3 sentences, citing specific skill steps

Then show compact menu.

### [4] Compare

Always available (not conditional). Steps:

1. **List installed skills** — enumerate every skill name under the host's skills directory (e.g. `~/.claude/skills/` on Claude Code).
2. **Sort by priority:**
   - **Tier 1 (Related):** skills named in the current digest's `## ◆ Related` section
   - **Tier 2 (Similar):** skills sharing a name prefix (e.g. `gstack-*`), same Type·Complexity, or overlapping domain keywords (issue/deploy/QA/etc.)
   - **Tier 3 (Other):** alphabetical
3. **Print the list and ask the user to pick:**

```
Pick a skill to compare with [skill-name]:

Related
  1. feature-dev — designs and implements a single feature
  2. gstack-review — PR review workflow

Similar
  3. gstack-autoplan — automatic planning
  4. gstack-investigate — root-cause bug tracing

Other
  5. viz
  6. skill-digest
  … and N more (enter a number or skill name)
```

4. **When the user enters a number or name**, print the comparison table:

```
## [skill-A] vs [skill-B]

| | [skill-A] | [skill-B] |
|---|---|---|
| When to use | ... | ... |
| Type | ... | ... |
| Complexity | ... | ... |
| Key difference | ... | ... |

Pick [skill-A] when: ...
Pick [skill-B] when: ...
```

**List display rules:**
- If 5+ skills share a prefix like `gstack-`, collapse them into one row labeled `gstack-* (N skills)`; expand on selection.
- If total skills > 20, show only Related + Similar first and offer a "show N more" option.
- If the target skill's SKILL.md has not been read, read it before generating the table.

Then show compact menu.

### [5] Skill Map (conditional)

Only offer when 5+ skills are installed.

**Hard output limit: 30 lines total. Enforce this strictly.**

**gstack-prefix rule:** If 30%+ of installed skills share a common prefix (e.g., `gstack-`), group them as a single row labeled with that prefix. Do not enumerate them individually.

**Chain rule:** Only include a chain if there is explicit evidence in SKILL.md files (e.g., "after X, run Y", "feeds into", redirect in When NOT to Use). Do not invent chains from name similarity.

Format:
```
## Skill Map ([N] skills)

**Where [current-skill] fits:**
[predecessor] → **[current-skill]** → [successor]

**By Type**
| Type | Skills | Count |
|------|--------|-------|
| [type] | skill-a, skill-b | N |
| [prefix]- | (N skills) | N |
| Other | — | N |
Maximum 5 rows. Extras → "Other".

**Common Paths**
[skill-a] → [skill-b] → [skill-c]  "when to use this path"
Maximum 5 paths. 2–3 skills per path only.

**Standalone** (no typical chain)
skill-x, skill-y … and N more
Maximum 5 listed.
```

Then show compact menu.

---

## Troubleshooting

**Skill not found (local)**
Tell the user the exact path searched (e.g. `~/.claude/skills/{name}/SKILL.md` on Claude Code, or whichever skills directory the host uses) and list 3–5 skills that do exist nearby.

**GitHub fetch fails**
Report the URL attempted. Suggest trying the raw URL directly or checking repo visibility.

**Collection has too many skills (20+)**
List first 10 in the table, add `… and N more`. Offer `/skill-digest [name]` for individual deep-dives.

**Digest seems too thin**
Say explicitly: "This skill has minimal documentation — digest reflects what's available." Do not pad.

**Menu option not available**
Remove [4] silently if no related skill is named in the digest. Remove [5] silently if fewer than 5 skills installed. Never show an option that cannot be completed.

**Watch Out / Tips section empty**
Omit the `## ◆ Watch Out` or `## ◆ Tips` header entirely. Do not write "No warnings" or "No tips."

**Key line unclear**
If you cannot distill a single load-bearing sentence, write what the skill *prevents* or *automates* in the user's life. Never leave `Key:` as a generic description of features.

**Deep Dive produces generic output**
If you cannot find 2+ non-obvious aspects, skip the choice list and say "This skill is straightforward — the digest covers all key behavior." Do not offer generic aspects like "how it works."

---
> Source: [egoing/skill-digest](https://github.com/egoing/skill-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
