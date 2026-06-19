## obsidian-write

> >


# obsidian-write — Obsidian Vault Writing Convention SSoT

> A single authoritative document for repeatedly-applied decisions when writing Obsidian notes: heading depth, emphasis-breakage rule, 5-axis tags, sticker frontmatter, PARA classification. Other writing skills delegate to this file as the convention authority via a vault signal.

## Quick Map

| Scenario | What happens |
|---|---|
| User writes a new `.md` directly in the vault | This skill's full conventions apply + post-write grep self-check |
| User edits an existing `.md` (heading / emphasis / tag changes) | Conventions applied to the changed region + verification of touched area |
| Another skill (polymedia / knou-note-writer / etc.) triggers in the vault | That skill owns its workflow (interview, lecture analysis); *format conventions* of the produced note are delegated here |
| Outside the vault (plain markdown / Logseq / Notion / Bear) | This skill is inactive — host skill applies its own generic-markdown case |

## Core vs Recommended vs Optional

| Tier | Items | Apply when |
|---|---|---|
| **Core** (all Obsidian users) | Heading (§3) · Horizontal rule (§4) · Indentation (§5) · Emphasis-breakage rule (§6) | Always in Obsidian |
| **Recommended** (CJK authors) | CJK-specific emphasis-breakage procedure (§10 + `ref/cjk-language-extra-checks.md`) | Note is written in Korean, Japanese, Chinese, or mixed CJK + Latin |
| **Recommended** (Make.md or folder emoji users) | Sticker frontmatter (§2) | Make.md plugin or folder-emoji management is in use |
| **Optional** (your operational policy) | Writing style §1 · 5-axis tags §7 · PARA §8 · Two-tier emphasis hierarchy (§6 Optional) | Your vault adopts these specific patterns |

Adopting only the four core items already eliminates almost all Obsidian rendering breakage. The rest is *opinionated* — patterns one user validated in their own vault. Take them if they fit; ignore if not.

---

## Output Language Policy

This skill is convention infrastructure, not a content generator. The convention text in this file is in English so any audience can read it, but **the notes you produce follow the user's language**.

- User writes in Korean → produce Korean notes. CJK emphasis-breakage procedure (§10) kicks in.
- User writes in English → produce English notes. CJK procedure is skipped (or run as a sanity pass anyway, since it's cheap).
- User writes in Japanese / Chinese → same as Korean. CJK procedure applies.
- User mixes languages → the dominant language drives §10; mixed segments run the relevant checks per language.

**Never translate the user's content into English just because this skill is in English.** Conventions are language-agnostic; voice and substance stay in the user's language.

---

## 1. Writing Style (Decision Logging)

1. **Record the why, not just the what** — conclusions decay without their reasoning. Six months later you should still see *why* you chose this path.
2. **ADR-style structure** — Context → Decision → Rationale → Consequences. Use this for any non-trivial decision recorded in a note.
3. **TODO markers** — Mark unresolved items with `TODO:` so they grep cleanly.
4. **Preserve deprecated decisions** — Use ~~strikethrough~~ rather than deleting, so the historical context survives.
5. **Tables for comparative analysis** — Markdown tables for trade-off matrices, not prose lists.
6. **Inline personal commentary** — Drop your own thinking, doubts, half-formed reactions into the note. Notes are for you, not for an audience.

> A note that only states the final decision is a dead artifact. A note that captures the deliberation is a living tool.

---

## 2. Frontmatter (Sticker / Folder Emoji)

**Note emoji**: Don't put emoji in the filename. Put it in frontmatter `sticker` using Make.md compatible format:

```yaml
---
sticker: emoji//{unicode-codepoint(lowercase hex)}
---
```

Examples:
- `sticker: emoji//1f48a` → 💊
- `sticker: emoji//1f34a` → 🍊
- `sticker: emoji//1f4d6` → 📖 (book)
- `sticker: emoji//1f3ae` → 🎮 (game)
- `sticker: emoji//1f3ac` → 🎬 (movie)
- `sticker: emoji//1f3b5` → 🎵 (music)
- `sticker: emoji//1f9e0` → 🧠 (thinking / essay)
- `sticker: emoji//1f4d8` → 📘 (academic note)

**Folder emoji**: Don't put emoji in the folder name (sync / cross-platform breakage). Use the `.md` file that shares the folder's name (a "folder spec note") and put sticker in its frontmatter:

```yaml
---
_filters: []
sticker: emoji//1f48a
color: ""
---
```

The folder spec note doubles as a place to record what the folder is *for* — directory README-style context.

---

## 3. Heading Rules

Obsidian's *Inline Title* renders the filename as h1. Using h1 in the body produces a duplicate title. Therefore:

- Body starts at **h2 (`##`)**
- Body ends at **h4 (`####`)** maximum
- `h1` (`#`) is **forbidden in the body**

```markdown
##(h2)  — main section
###(h3) — subsection
####(h4) — detail
```

Deeper hierarchy: use `**bold labels**` inside the body. Don't create h5 / h6.

**No numbered headers**: `## 1. Topic` ❌ → `## Topic` ✅. Outline view gets cluttered with numbers and reordering breaks them.

> This diverges from the CommonMark MD041 lint, but Obsidian is the deployment target — in-vault readability beats portability here.

> **Exception for skill documents**: This skill's own files (`SKILL.md`, `README.md`, `ref/*.md`) use numbered headers (`## 1. Title`) because they're primarily consumed on GitHub where numbered section scanability outweighs Obsidian-outline concerns. The convention applies to *vault notes you write*, not to convention documents that ship to external consumers.
>
> `examples/*.md` are *GitHub markdown documents that embed simulated vault notes inside fenced code blocks*. The outer GitHub document layer uses `# example-xxx.md` (h1) for file titling — fine because it's not a vault note. The embedded simulated vault note (inside the fence) follows the full convention set: h2 start, no numbered headers, etc. When running `scripts/audit-vault.sh` on this repo, expect 1 h1 hit per example file — that's the outer title, not a violation. When auditing your actual vault, this distinction does not apply because vault notes have no such embedding.

---

## 4. Horizontal Rule (`---`)

`---` is **only used between h2 sections**. Never between h3s — they're siblings under the same h2 context, separated by blank lines.

```markdown
## H2 Section A
... content ...

### H3 sub A-1
... content ...

### H3 sub A-2
... content ...

---

## H2 Section B
```

Exception: frontmatter delimiters at top of file.

---

## 5. Indentation

List sub-items use **tabs**, not spaces. Obsidian's behaviors (toggle, fold, drag) are tab-based:

```markdown
- Top-level item
	- Second level (1 tab)
		- Third level (2 tabs)
```

---

## 6. Emphasis (`**`, `*`, `_`) — CommonMark Flanking Rule

> The **single most important rule** for CJK note authors. Bold/italic markup breaks far more often in CJK text than in Latin text because of how particles attach to words.

### Master Rule (one line)

> **If the character immediately inside `**` is not a letter (Han / Latin / digit), it's at risk.** Whitespace, line-start, or line-end on the outside saves it.

Verified consistent across three parsers (cmark · markdown-it · @lezer/markdown). Direct consequence of CommonMark's flanking rule.

### When it breaks

Both conditions must hold simultaneously:
1. **Outside** the `**` has a word character (Han / Latin / digit) attached *without whitespace*
2. **Inside** the `**` (right against the delimiter) has Unicode punctuation: `"` `'` `(` `)` `.` `,` `:` `—` `!` `?` `;` `$`

**Either delimiter alone is enough to kill the entire bold.** Opening and closing delimiters are judged independently.

### Quick examples (CJK / Korean — the worst case)

```
❌ **활용(exploitation)**과     ← closing ** preceded by `)`, followed by Korean particle `과`
✅ **활용**(exploitation)과     ← keep only the Korean term bold; English gloss outside

❌ 이건**"한 우물 파기"**이다  ← both sides break
✅ 이건 "**한 우물 파기**" 이다  ← punctuation outside the bold

❌ **결론적으로,**이렇게       ← closing ** preceded by `,`, followed by `이`
✅ **결론적으로**, 이렇게      ← comma outside
```

### Quick examples (English — happens but rarer)

```
❌ word**"text"**here           ← both sides break (no whitespace + punct inside)
✅ word "**text**" here

❌ conclusion**(note)**follows  ← same pattern
✅ conclusion (**note**) follows
```

### Universal fix

Put `**` so that the character immediately inside is a letter. If you need punctuation adjacent to bold, move it outside or extend the bold to include the particle.

### Same rule applies to `*` (italic), `_` (italic), and `==` (Obsidian highlight)

All emphasis delimiters follow the same CommonMark flanking rule. The grep patterns in §10 cover `**`; substitute `=` for `*` in those patterns to also catch `==highlight==` breakage in CJK notes. Highlight-specific grep variants are in `ref/obsidian-syntax-reference.md` §8.

### Optional: Two-tier emphasis hierarchy

Some vaults adopt a stricter emphasis policy so scanning has reliable visual weight. This is **opt-in** — when active, `**bold**` and `==highlight==` form a 2-tier hierarchy, and Korean prose forgoes italic entirely. §10 self-check picks up the extra grep automatically.

**Hierarchy (when enabled)**

| Tier | Marker | Use | Budget |
|---|---|---|---|
| L0 | plain | normal prose | 80%+ |
| **L1 — scanning** | `**bold**` | important content — not the absolute core, but worth catching on a skim (keywords, one-line conclusions, table cells deserving emphasis) | use as needed — multiple per paragraph is fine when each marks distinct importance; avoid bolding every sentence |
| L2 — crystallization | `==highlight==` | **must-not-miss points** — the lines a reader would actively suffer for skipping | sparingly — no flooding |

L2 is for content the reader cannot afford to miss. The hard counts above ("sparingly", earlier drafts said "0~1 per section") are guidance, not law — **two in one paragraph is fine** if both are genuine must-not-miss points. The real failure mode is flooding: if **every** paragraph's takeaway is `==highlighted==`, the hierarchy flattens and the marker stops carrying signal. The judgment is qualitative ("would the reader regret missing this?"), not a counter.

**Italic policy**

- ❌ Korean prose — single `*x*` / `_x_` banned. Korean fonts render italic as a slanted glyph that adds visual noise without semantic gain; the 2-tier hierarchy above covers what italic would have signaled.
- ✅ English-only italic still allowed — book titles (`*Thinking, Fast and Slow*`), foreign loanwords (`*ad hoc*`, `*de facto*`), deliberate stress inside English-only sentences.
- Mixed-language sentence with Korean as the matrix language → no italic. Move the English term into bold or backticks instead.

**Boundary cases**

- *Structural bold is exempt from the L1 budget* — `**기출X**:`, `**Q1:**`, `**답: ③**`, table-header bold, callout-internal labels (`> [!TIP] **요점:**`) are pattern markers, not hierarchy emphasis. They don't count toward the L1 budget at all. The budget is for *prose* bold.
- *No nesting between tiers* — `**중요 ==핵심==**` and `==핵심 **강조**==` are both forbidden. The two tiers are mutually exclusive; pick the higher one and drop the inner.
- *Callout titles minimize emphasis* — the callout itself is already visual separation. `> [!IMPORTANT] **중요**` doubles the signal; just `> [!IMPORTANT] 중요` is enough.
- *Highlight inside a quote/blockquote* is fine; it's the rare cross-tier marker that survives.

**Activating this policy in a vault**

Add to vault root `CLAUDE.md` (or equivalent agent context file):

```markdown
## Emphasis policy
Two-tier hierarchy ON (obsidian-write §6 Optional). Korean italic banned.
```

Host skills (knou-note-writer, polymedia-review-skill, etc.) that delegate format authority here automatically inherit the policy.

### Optional: Quantitative highlight budget variant

A further opt-in *on top of* the two-tier hierarchy above. The qualitative
"would the reader regret missing this?" rule scales with judgment, but exam-prep
or scan-heavy vaults benefit from a hard count — readers actively skim for
`==highlights==` as the answer-key layer, so flooding kills the signal fast.

When this variant is active:

| Deepest header in branch | Budget per leaf section |
|---|---|
| `#### ` (h4 leaf) | **3 highlights** |
| `### ` (h3 leaf — no h4 below) | **4 highlights** |
| `## ` (h2 leaf — no h3, h4 below) | **5 highlights** |

Rule: base **3** at the deepest level, **+1** per shallower step. A shallower section covers more material, so it earns a small budget increase.

**Parent-header lead paragraphs** — when an h2 or h3 has child headers, its own direct body (the lead paragraph before the first child header) gets a **budget of 1** highlight. Enough for a single banner term, not a full takeaway.

**Counting**

- Each `==occurrence==` costs 1 — repeating the same word twice still costs 2.
- Budget applies per *leaf section* (the deepest header in that branch).
- Callout-internal highlights count toward the parent section's budget.
- Structural emphasis (table headers, `**Q1:**`, `**답:**`) is bold-only — highlights have no structural exemption.
- Front-matter and code fences are not counted.

**Why these numbers**

3 maps to a reader's working-memory chunk for one subsection (a deepest leaf is the smallest navigable unit — usually one concept, one formula, or one pitfall). +1 per shallower step keeps the parent banner usable without inheriting full child budgets. Parent-lead =1 is a banner allowance, not a full takeaway.

**Activation** — replace the two-tier activation line in `CLAUDE.md` with:

```markdown
## Emphasis policy
Two-tier hierarchy ON (obsidian-write §6 Optional). Korean italic banned.
Quantitative highlight budget ON — h4=3 / h3=4 / h2=5, parent-lead=1 (obsidian-write §6 variant).
```

§10 self-check picks up the per-section counter via `scripts/highlight-budget.sh`.

### Deep-dive

See `ref/emphasis-breakage-deep-dive.md` for:
- Full CommonMark flanking spec walkthrough
- Per-parser behavior comparison
- All 6 breakage patterns with safe rewrites
- The "risky characters" full catalog
- Myth-busting (smart quotes, live-preview "bugs", `_..._` alternative)

---

## 7. Five-Axis Tag Model

Tags are **horizontal metadata** — orthogonal to folders and links.

- **Folders** — primary classification (1:1)
- **Links / MOC** — explicit intentional connections
- **Tags** — calling notes from a different axis (a *weak* connector, in Luhmann's Zettelkasten sense)

### The five axes (use tags only along these)

| Axis | Examples | Could be a folder? |
|---|---|---|
| **Status** (state / action) | `#review-needed` `#draft` `#evergreen` `#exam-critical` | ❌ varies over time |
| **Type** (note kind) | `#concept` `#literature-note` `#adr` `#guide` `#essay` | ⚠️ strongly horizontal |
| **Domain** (field / subfield) | `#cs` `#database` `#math` `#psychology` | ❌ forbidden if folder already covers |
| **Topic** (primary concept) | `#deep-learning` `#regularization` | ⚠️ prefer wikilink if a `[[X]]` MOC exists |
| **Context** (situational / meta) | `#mobile` `#review-2026q2` | ❌ ephemeral / meta |

### Forbidden

- Tag with the same name as the containing directory (duplicates the classification axis)
- Free-form tags outside the 5 axes (taxonomy decays)

### Naming

- lowercase + kebab-case + singular (`#literature-note` ✅, `#Literature_Notes` ❌)
- Keep native language (`#복습-필요`, `#推荐`), English singular (`#book` ≠ `#books`)
- Pick one writing system per concept: `#deep-learning` ↔ `#딥러닝` — don't mix

### Operating principles

- **2~5 tags per note** is the sweet spot. 10+ is tag-explosion noise.
- **A Topic tag accumulating 5+ notes → graduate to a MOC**: create `MOC - {topic}.md`, switch primary navigation to wikilinks, keep the tag only as a search aid.
- **Status tags need an expiry**: `#exam-critical` is time-bound — note expiry in frontmatter (`tag-expires: 2026-06`) or it becomes a ghost tag.
- **Quarterly pruning**: consolidate/delete tags used by only 1–2 notes; merge spelling variants.

### Wikilink vs Tag — decision

- Want to see concept clustered → separate `[[X]]` MOC note + wikilinks
- Want fast search only → tag
- Both → wikilink primary, tag secondary

---

## 8. PARA Classification (Optional, separate reference)

If you adopt Tiago Forte's PARA method, this skill's conventions work well with it. The four-folder structure (`1. Projects` / `2. Area` / `3. Resource` / `4. Archives`), the **folder spec note** pattern (a `.md` file with the same name as its folder, holding sticker + context), and the **medium-first Resource layout** (`PARA/3. Resource/{medium}/reviews/{title}.md`) are documented in detail in `ref/para-classification.md` along with PARA-vs-other-systems decision rules, migration tips, and how PARA interacts with the 5-axis tag model.

PARA-skippers (Johnny Decimal, LATCH, Bullet Journal, flat folders, topic-driven hierarchies, etc.) can ignore this section entirely — the rest of this skill's conventions (sticker, tags, emphasis, self-check) are layout-agnostic.

---

## 9. Delegation Interface (for skill authors, separate reference)

Building another Obsidian writing skill (review notes, lecture notes, journal automation, etc.) that should delegate convention authority to this one? See `ref/delegation-for-skill-authors.md` for the detection signal priority, the delegatable / non-delegatable convention areas, the standard delegation flow, common mistakes to avoid, versioning stability rules, and worked examples (polymedia-review-skill, knou-note-writer).

Regular skill *users* writing notes in their own vault don't need this section — it's purely for other AI agent skill developers.

---

## 10. Post-Write Self-Check (Mandatory)

Run immediately after Write / Edit, **before** reporting completion to the user. Don't wait to be asked.

### Two-stage strategy

Stage 1 is a narrow grep that catches real breakage. Stage 2 is a wider grep for borderline cases that need human judgment.

#### Stage 1 — High-confidence breakage patterns

```bash
cd "<note folder>"
TARGET="*.md"   # or a specific file

# (1) Closing-side break: inside-punct + ** + word-character outside (CJK-heavy)
grep -nP '[\)\]"'\''\.,:;!?\$…—–]\*\*[가-힣\p{Han}\p{Hiragana}\p{Katakana}A-Za-z0-9]' $TARGET

# (2) Opening-side break: word-char outside + ** + opening punct inside
grep -nP '[가-힣\p{Han}\p{Hiragana}\p{Katakana}A-Za-z0-9]\*\*["\(\[\$「『]' $TARGET

# (3) h1 in body (forbidden) — exclude `# comment` inside code blocks
for f in $TARGET; do
  awk -v file="$f" '/^```/{in_code = !in_code; next}
                    !in_code && /^# /{print file ":" NR ": " $0}' "$f"
done

# (4) Numbered headers (`## 1. Topic` only — does not flag `802.1X` etc.)
grep -nE '^##+ [0-9]+\. ' $TARGET

# (5) Korean italic — only when Two-tier emphasis hierarchy is ON (see §6 Optional)
#     Strip **bold** first (so `*` inside `**...**` won't false-flag), then catch any
#     remaining `*X*` on a line that also contains a Korean character.
for f in $TARGET; do
  awk -v file="$f" '
    /^```/ { in_code = !in_code; next }
    in_code { next }
    {
      line = $0
      gsub(/\*\*[^*]+\*\*/, "", line)
      if (line ~ /\*[^*[:space:]][^*]*[^*[:space:]]\*/ && line ~ /[가-힣]/)
        print file ":" NR ": " $0
    }
  ' "$f"
done
```

If Stage 1 returns zero lines, the note has likely passed. Any hits are almost certainly real — fix with §6 patterns immediately. For check (5), false positives are rare but possible in math/code-adjacent prose (`*x*²` in a sentence about variables); review hits manually and either rewrite or move to a code block.

> ⚠️ For check (3), a naive `grep -nE '^# '` falsely flags `# comment` lines inside Python / Bash code blocks. The awk version tracks the code-block toggle correctly.

#### Stage 2 — Wide flanking grep (advisory)

```bash
# (5) Closing-side suspects: non-word-char inside + word-char outside
grep -nP '(?<=[^\s\w가-힣\p{Han}\p{Hiragana}\p{Katakana}])\*\*(?=[\w가-힣\p{Han}\p{Hiragana}\p{Katakana}])' $TARGET

# (6) Opening-side suspects: word-char outside + non-word-char inside
grep -nP '(?<=[\w가-힣\p{Han}\p{Hiragana}\p{Katakana}])\*\*(?=[^\s\w가-힣\p{Han}\p{Hiragana}\p{Katakana}])' $TARGET
```

These don't know whether each `**` is opening or closing, so they also flag the inverse case (which is actually safe). Walk each hit manually:

1. Identify which side of the bold pair the matched `**` belongs to
2. Check what's *inside* (against the delimiter, on the bold's side)
3. Inside is a letter → false positive (master rule: safe)
4. Inside is punctuation + outside is a letter → real break → fix per §6

#### Stage 3 — `---` between h3s (optional)

```bash
for f in $TARGET; do
  awk -v file="$f" '
  /^---$/ && NR > 5 { hr_line = NR; in_hr = 1; next }
  in_hr { if (/^$/) next
    if (!/^## /) print file ":" hr_line ": --- not followed by h2 → " $0
    in_hr = 0 }
  ' "$f"
done
```

### CJK note? Run the extra procedure

If the note is in Korean / Japanese / Chinese / mixed CJK, run **`ref/cjk-language-extra-checks.md`** procedure additionally. CJK particles attach to `**` differently from Latin, and the extra checks catch the breakages Stage 1/2 might miss.

### Pass condition

Stage 1 all empty → pass. Stage 2 is advisory (high false-positive rate). Stage 3 when suspected.

**Do not report "done" to the user without passing this self-check.**

---

## 11. Obsidian-Specific Syntax (separate reference)

This skill's §1–§10 are *convention rules* (when to use what, formatting policy). The *catalog of syntax* that Obsidian adds on top of CommonMark + GFM lives in a separate reference file:

`ref/obsidian-plugin-essentials.md` covers the plugin ecosystem most relevant for this skill — three-tier categorization (Required: Make.md / Dataview; Recommended: Templater / Periodic Notes / Linter / Tasks / Tag Wrangler; Compatible: Excalidraw / Citations / Hover Editor / etc.) with first-time-user-friendly "why install" rationale, a 4-stage progressive rollout plan, and an explicit overlap analysis between Linter and §10 self-check (they complement rather than replace each other — Linter for broad GFM coverage, this skill's grep for CJK-specific emphasis breakage).

`ref/obsidian-syntax-reference.md` covers:
- Wikilinks (`[[Note]]`, `[[Note|Display]]`, `[[Note#Heading]]`, `[[Note#^block]]`, same-note links) + disambiguation rules
- Block IDs (`^id`) — paragraph / list-item / quote-block placement
- Embeds (`![[...]]`) — notes, sections, blocks, images with width+height, audio, video, PDF with page, query embeds
- Callouts — full type catalog (13 canonical types + aliases like `summary`/`tldr` = `abstract`), collapsing (`-`/`+`), nesting, custom CSS callouts
- Properties — Obsidian 1.4+ typed properties (text/list/number/checkbox/date/datetime), standard names (`tags`, `aliases`, `cssclasses`, `publish`, `cover`)
- Inline tags — character rules, frontmatter vs inline trade-offs
- Comments (`%%hidden%%`) — Obsidian-specific, **not** portable to other markdown parsers (use HTML comments for portability)
- Highlight (`==text==`) — follows the same flanking rule as `**` (see §6 above)
- Math — MathJax 3.x, inline + block + `align` environments
- Mermaid — full type list + `class NodeName internal-link;` for vault-note linking (Obsidian-only, won't navigate on GitHub)
- Footnotes — standard `[^1]` + inline `^[...]` (portability warning)
- Task lists — GFM `- [ ]` + Obsidian extensions (`- [/]` in-progress, `- [-]` cancelled, `- [?]`, `- [!]`, `- [<]`, `- [>]`, etc.) + Tasks plugin emoji metadata
- Strikethrough, HTML embedding subset, Dataview integration

Verified against Obsidian 1.5+. Use this when you need to look up the *syntax* itself; this SKILL.md handles the *conventions* around when to use it.

## 12. Agent Context Files (CLAUDE.md / AGENTS.md / etc., separate reference)

This skill also writes and maintains the markdown files that AI coding agents read to understand a project — `CLAUDE.md` (Claude Code), `AGENTS.md` (emerging cross-agent standard), `AGENT.md`, `.cursorrules` (Cursor), `.codex.md` (Codex CLI), `.windsurfrules` (Windsurf), and the equivalent files for future agents. For an Obsidian vault, these live at vault root and tell the agent: who maintains this vault, what organizing system is in use, what skills are installed, which plugins are active, what writing voice to adopt.

When this skill writes or edits an agent context file, it applies a **tailored subset** of the conventions in §1–§7 — heading depth, hr, indent, emphasis-flanking, and §10 self-check all apply; sticker frontmatter and 5-axis tags and PARA path do *not* (agent context files aren't classified vault notes). Numbered headers are acceptable here for the same reason they're acceptable in this skill's own SKILL.md / ref docs (see §3 exception clause) — explicit section numbers serve agents reading the file.

`ref/agent-context-files.md` covers:
- The catalog of agent context formats (CLAUDE.md / AGENTS.md / .cursorrules / .codex.md / .windsurfrules) — which agent reads which, scope inheritance, format expectations
- "Why agent context files matter" specifically for Obsidian vault owners (not codebases)
- A structured 8-section template for vault-flavored agent context (Owner / Structure / Active projects / Conventions / Skills / Plugins / Voice / External)
- Convention application matrix — which §1–§10 rules apply to agent context files vs. which don't
- "Single file or many?" trade-offs (CLAUDE.md only / AGENTS.md primary / per-agent files) + symlink strategy
- Maintenance cadence (quarterly review, on skill install/uninstall, on structural changes)
- A worked CLAUDE.md / AGENTS.md skeleton you can adapt

If you maintain an Obsidian vault and want any agent walking into it to be calibrated within 30 seconds, this is the file format that does that.

## 13. Related Skills

External Obsidian writing skills that delegate to this one:

| Skill | Domain | Delegation |
|---|---|---|
| [polymedia-review-skill](https://github.com/rhino-ty/polymedia-review-skill) | Review notes (books, games, movies, music) | Detects this skill via signal in its "Note system adapter" section → auto-delegates |
| [review-myblog-converter](https://github.com/rhino-ty/review-myblog-converter) | Obsidian note → blog tone conversion | Input-only consumer; no direct convention overlap |
| Lecture / academic note skills (vault-local) | Course notes, study material | Delegates §6 + §10 self-check; keeps domain-specific grep (e.g., header-number rule, exam-meta phrases) |
| Interactive visualization skills (dataviewjs) | In-note interactive widgets | Owns code-block content; convention overlap minimal |

---

## License

MIT. Use the convention bundle if it fits your vault; modify freely. The CJK emphasis-breakage rule (§6, §10, `ref/`) is particularly valuable for Korean / Japanese / Chinese authors — even users who don't adopt the rest may want to keep these procedures.

---
> Source: [rhino-ty/obsidian-write](https://github.com/rhino-ty/obsidian-write) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
