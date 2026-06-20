## free-indulgence

> Academic English polishing and AI-pattern removal for scientific manuscripts. Default combines SCI-level polish + de-AI detection + rewrite. Supports -p (polish only), -d (de-AI only), -t (top-conference polish standard for NeurIPS/ICLR/ICML). Slash command is /fi (user-configurable). Use when user mentions "polish English", "de-AI", "reduce AI detection", "remove AI traces", or wants to improve English academic text for SCI/CS conferences.


# FreeIndulgence: Academic Polish & AI Pattern Removal

## Quick Start

```
/fi                                 # Default: SCI polish + de-AI
/fi -dp                             # Same as default (explicit)
/fi -p                              # SCI polish only
/fi -p -t                           # Top-conference polish only
/fi -t                              # Top-conference polish + de-AI
/fi -d                              # De-AI only
/fi path/to/file.tex                # Process a file with default mode

INVALID: /fi -d -t                  # -t requires polish, but -d skips polish
```

> **Note**: The slash command `/fi` can be renamed by editing the `name` field in this file's frontmatter or by the user's OpenCode/Claude Code configuration. The command name is not hardcoded.

---

## Mode Branching Logic

Before starting the workflow, determine which steps are active:

| Flag | Active Steps |
|------|-------------|
| (none) or `-dp` | Steps 1→2→3(SCI)→4→5→6→7→8→9→10→11 (full pipeline) |
| `-p` | Steps 1→2→3(SCI)→11 (polish only) |
| `-p -t` | Steps 1→2→3(TOP)→11 (top-conference polish only) |
| `-t` | Steps 1→2→3(TOP)→4→5→6→7→8→9→10→11 (top polish + full de-AI) |
| `-d` | Steps 1→2→4→5→6→7→8→9→10→11 (de-AI only, skip polish) |
| `-d -t` | **REJECT** — explain: "-t upgrades polish, but -d skips polish. Use -t without -d for top-conference polish + de-AI." |

---

## How AI Detectors Work — and Our Counter-Strategy

AI detectors (Turnitin, GPTZero, Originality.ai) don't look for specific words. They measure two statistical signals:

### Perplexity — Word Predictability

Given the preceding text, how predictable is the next word?

- **High perplexity** = unpredictable, varied word choices → reads as **human**
- **Low perplexity** = each word is the statistically most likely choice → reads as **AI**

GPT models produce low-perplexity text: the vocabulary is standardized, every word is the "obvious" choice. Claude naturally has higher perplexity — richer vocabulary, less predictable phrasing.

**Our counter-strategy**: After rewriting, sanity-check: if every word in a sentence feels like the most predictable choice, deliberately vary 2-3 words to equally valid but less common alternatives. This isn't about using obscure words — it's about breaking the statistical fingerprint of "the model always picks the median word."

### Burstiness — Sentence Length Variation

How much does sentence structure vary within a paragraph?

- **High burstiness** = sentences of dramatically different lengths, mixed structures → reads as **human**
- **Low burstiness** = sentences all similar length, uniform structure → reads as **AI**

GPT produces low-burstiness text: structured parallel sentences, uniform length. Claude naturally has higher burstiness — long sentences mixed with fragments, colloquial shifts, complex modifiers.

**Our counter-strategy**: After every 2-3 sentences of similar length, deliberately break the pattern. Insert a 3-word sentence. Merge two medium sentences into a long complex one. Use a sentence fragment for emphasis. The rhythm should feel like a person thinking, not a machine formatting.

### The Combined Effect

A detector's confidence score comes from the **interaction** of these two signals. Low perplexity + low burstiness = near-certain AI. High perplexity alone isn't enough if sentence structure is uniform. High burstiness alone isn't enough if every word is predictable.

**Our rewrite must address both simultaneously**: varied vocabulary (perplexity) AND varied sentence architecture (burstiness).

---

## Content Segmentation (MANDATORY first step)

Before any processing, segment the input into content zones. Different zones have different rules:

### Zone Types

| Zone | Examples | Polish Rule | De-AI Rule |
|------|----------|-------------|------------|
| **Prose** | Paragraphs, sentences, captions | Full polish | Full de-AI |
| **Code Block** | Fenced ``` blocks, `\begin{lstlisting}`, `\begin{algorithm}`, `\begin{verbatim}`, `\begin{minted}`, indented code (4 spaces) | **SKIP entirely** | **Skip code body**, scan **comments** (auto-rewrite on confirm) and **print/log strings** (report only) |
| **String Literal in Code** | `print(f"...")`, `logger.info("...")`, `console.log(...)`, `System.out.println(...)`, `raise ValueError("...")`, `assert ... , "..."` | **SKIP** | **Report only** — flag AI patterns in string content, suggest rewrite. Do NOT modify unless user explicitly requests print string modification |
| **Inline Code** | Backtick-wrapped `` `code` ``, `\texttt{...}`, `\lstinline{...}` | **SKIP** | **SKIP** |
| **Math** | `$...$`, `$$...$$`, `\begin{equation}` | **SKIP** | **SKIP** |
| **LaTeX Commands** | `\cite{}`, `\ref{}`, `\label{}`, `\textbf{}` | **PRESERVE** | **PRESERVE** |

### Code Block Comment Handling (Three-Tier System)

**Tier 1 — KEEP (Concise "Why" comments)**:
Comments that explain rationale, assumptions, algorithm choices, or cite references. Keep but make concise (≤1 line preferred). **If the comment contains numbering or bullet markers, report it in the diagnostic even if classified as Tier 1** — the user may want to review.
```
OK:       # Apply L2 regularization to prevent overfitting (see §3.2)
REPORT:   # 1. Load weights  2. Normalize  3. Forward pass  ← numbering needs reporting
```

**Tier 2 — DE-AI REWRITE (Verbose "What" comments with AI-flavored language)**:
Comments that describe code behavior but use AI-typical vocabulary or are overly verbose. Rewrite to be concise and natural — **reduce information to essentials, keep only what a reader needs to understand the code's intent**. Strip AI vocabulary. Preserve comment syntax.
```
BEFORE: // This function leverages the robust framework to delve into the intricate
        // data processing pipeline and subsequently showcase the optimized results
AFTER:  // Process data and output results
```

**Tier 3 — REMOVE (Redundant "What" comments narrating obvious code)**:
```
// increment the counter by 1           ← remove
# print the result to the console       ← remove
```

NEVER modify code logic, variable names, function signatures, or syntax — only comments.

### Prose Bullet / Numbered List Handling

Bullet points and numbered lists in **prose paragraphs** are treated differently from code comments. Prose content is NOT deleted — it is **rewritten into flowing paragraphs**.

**Detection**: The script reports `bullet_density` (fraction of prose lines matching bullet/numbering patterns). Threshold: >30% triggers the prose rewrite path.

**Patterns detected in prose**: `-`, `*`, `+`, `•`, `·`, `1.`, `2.`, `1)`, `(1)`, `a)`, `i.`, emoji markers (`🚀`, `✅`, `💡`), bold-header markers (`**word**:`)

**Rule for prose bullets** (different from code comments):
| Condition | Action |
|-----------|--------|
| bullet_density ≤ 30% | Normal de-AI rewrite; isolated bullets are fine |
| bullet_density > 30% | Flag for **prose conversion** — rewrite entire section as flowing paragraphs |
| has_total_sub_total = True | Flag for **structural rewrite** — restructure away from template format |

**The prose rewrite must NOT**:
- ❌ Convert bullet-by-bullet into sentences (produces uniform-length "list with punctuation")
- ❌ Delete bullet content (that's the code comment rule, not the prose rule)

**The prose rewrite MUST**:
- ✅ Identify the single argument the bullet list is making
- ✅ Write one coherent paragraph making that argument, pulling data points as evidence at varying depths
- ✅ Drop bullets that are purely decorative or restate the heading
- ✅ Deliberately vary sentence architecture (burstiness) in the result

### Print / Log / Assert String Handling

String literals inside print/log/assert/raise statements are **prose embedded in code**. Scan for AI patterns.

**Rule**: Report only by default. DO NOT auto-rewrite. Only rewrite if user explicitly requests print string changes.

### Code Style Flagging (No Auto-Fix)

Flag only, never auto-modify.

| Issue | Example | Suggestion |
|-------|---------|------------|
| Overly verbose variable names | `processed_data_final_version_two` | Consider shortening |
| Unnecessary print/log statements | `print(f"Debug: processing {x}")` | Remove debug prints from paper code |
| Over-commented code | `# a=1 # assign 1 to a # then increment` | Keep only essential comments |

---

## Workflow: Full Pipeline (default / -dp / --top)

### Step 1: Read Input
If a file path is given, read the file. If text is pasted inline, use that.

### Step 2: Segment Content
Execute **Content Segmentation** (see above). Identify all zone types.

### Step 3: Polish Prose Zones

Two polish levels. Choose based on `-t` flag.

---

#### Level A: SCI Basic Polish (default, used with `-p` or no flag)

Apply the following editing rules **only to Prose zones**:

**Vocabulary & Register**:
- No contractions (use "it is" not "it's", "does not" not "doesn't").
- Use simple, clear academic words. Avoid ornate or obscure vocabulary.
- Avoid possessive forms for method/model names: use "the performance of METHOD" not "METHOD's performance".

**Sentence Optimization**:
- Adjust structures for formality and logical coherence.
- Refine complex sentences for fluency. Eliminate awkward non-native phrasing.
- Fix all spelling, grammar, punctuation, and article errors.

**Format Preservation**:
- Keep abbreviations as-is (LLM, CNN, GPU).
- Preserve ALL LaTeX commands (`\cite{}`, `\ref{}`, `\eg`, `\ie`).
- Keep existing `\textbf{}` but NEVER add new emphasis.

**Structure**: NO itemized lists. Keep text as coherent paragraphs.

---

#### Level B: Top-Conference Polish (`-t`)

Role-play as the following expert editor. Apply ALL constraints below. The goal is zero-error publication standard for NeurIPS, ICLR, ICML.

```
# Role
You are a senior academic editor in computer science, specialized in polishing
manuscripts for top-tier conferences (NeurIPS, ICLR, ICML). Your goal is to elevate
the text to zero-error publication standard.

# Task
Deeply polish and rewrite the provided English text. Go beyond error correction —
improve academic rigor, clarity, and overall readability.

# Constraints
1. Academic Rigor & Sentence Optimization (Core):
   - Adjust sentence structures to meet top-conference writing standards. Enhance
     formality and logical coherence.
   - Refine complex sentences for fluency. Eliminate awkward phrasing from
     non-native writing.
   - Zero-error: fix all spelling, grammar, punctuation, and article errors.

2. Vocabulary & Register:
   - Formal register only. No contractions (use "it is" not "it's", "does not"
     not "doesn't").
   - Use simple, clear academic vocabulary. Avoid ornate or obscure words.
   - Avoid possessive forms for method/model names (use "the performance of
     METHOD" not "METHOD's performance").

3. Content & Format Preservation:
   - Keep common abbreviations as-is (e.g., LLM, CNN, GPU).
   - Preserve all LaTeX commands (\cite{}, \ref{}, \eg, \ie).
   - Keep existing formatting like \textbf{}, but NEVER add new emphasis formatting.

4. Structure:
   - NO itemized lists. Keep text as coherent paragraphs.

5. Output Format:
   - Part 1 [Edited English]: Only the polished English text. Escape special chars
     (% , _, &). Preserve math ($...$).
   - Part 2 [Literal Translation]: A direct Chinese translation (NO English glosses in parentheses).
   - Part 3 [Modification Log]: Brief notes on what was changed.
```

**Skip verbatim**: Code Blocks, Inline Code, Math, LaTeX commands — output exactly as-is.
Reassemble all zones in original order.

> When `-p` or `-p -t` is active: jump directly to Step 11 (Output) after reassembly.

---

### Step 4: Run Detection Script (Hard Patterns)

Run the bundled detection script for deterministic pattern matching:

```
python scripts/detect.py < input.txt
# or for a file:
python scripts/detect.py path/to/file.tex
```

The script outputs JSON with:
- **prose**: bullet_density, has_total_sub_total, total_words, unique_words, ai_vocab_hits, mechanical_connectors, hedging_stacks, filler_phrases, significance_inflation, em_dash_count, sentence_length_variance, connector_repetition
- **connector_blocks[]**: logical blocks detected by sequential signposting (e.g. "First... Second... Third... Finally"), each containing block_text, sequence_type, and markers_found
- **genai_risk**: risk tier classification (high/moderate/low) with numerical score and breakdown of contributing factors
- **code_blocks[]**: per-block language, extracted comments with tier classification (1/2/3) and reason
- **summary**: aggregate counts + genai_risk_tier + connector_blocks_count

Covers 20+ languages.

### Step 5: Run Forced Diagnostic Checklist

Go through this **mandatory checklist** item by item. Do NOT skip any item.

```
□ 1. Mechanical Connectors: Count of "First and foremost" / "Notably" / "Moreover" (3+/para)?
   Script found: [N]. My semantic scan found: [N]. Combined verdict: [PASS / FLAG]

□ 2. AI Vocabulary: Any of "delve into, leverage, tapestry, realm, crucial, pivotal,
   showcase, robust, intricate, underscore, landscape, paradigm, groundbreaking"?
   Script found: [list]. My semantic scan: any false positives in technical context? [Y/N]

□ 3. Bullet-Point Density: Script reports [X]%. If >30% → FLAG for rewrite to prose.
   Are bullets actually helpful (e.g., listing hyperparameters) or decorative?

□ 4. Total-Sub-Total Structure: Script reports [True/False].
   Does the text follow "overview → point 1 → point 2 → point 3 → summary"
   in a way that feels templated?

□ 5. Sentence Rhythm: Script reports variance [X]. If variance < 5.0 and
   sentence_count > 5 → FLAG for uniform rhythm.

□ 6. Em-Dash Count: Script reports [N]. If >2 in any paragraph → FLAG.

□ 7. Hedging Stacks: Script found: [list]. Any multi-word hedging?

□ 8. Filler Phrases: Script found: [list]. Can any be cut?

□ 9. Significance Inflation: Script found: [list]. Are claims proportionally supported?

□ 10. Code Comments: Script found [N] Tier-2, [N] Tier-3 across [N] code blocks.
    Any false positives/negatives in the script's classification?

□ 11. Connector Logic Blocks: Script found [N] logical blocks with sequential signposting
    (type: [list types]).

□ 12. GenAI Risk Tier: Script reports [high/moderate/low] (score [N]/19, high≥5 moderate≥1).
    Key factors: [list top 3 contributors to score]

□ 13. Parenthesis Over-Explanation: Script reports density [N] per sentence.
    If >0.8 per sentence → FLAG for over-use of parenthetical asides.

□ 14. Bold-Header + Sub-Total in Small Paragraphs: Script reports [N] paragraphs
    with bold-header + sub-total pattern in paragraphs <100 words.
    If ≥2 → FLAG for template paragraph structure.
```

### Step 6: Produce Combined Diagnostic Report

Merge script results + checklist answers into the final diagnostic. Template:

```
## AI Pattern Diagnostic Report

### Hard Detection (Script)
| Metric | Value | Threshold | Status |
|--------|-------|-----------|--------|
| Bullet density | X% | >30% flagged | [PASS/FLAG] |
| Total-sub-total | [Yes/No] | - | [Note] |
| AI vocab hits | N | >0 flagged | [list top 5] |
| Mechanical connectors | N | >0 flagged | [list] |
| Hedging stacks | N | >0 flagged | [list] |
| Filler phrases | N | >0 flagged | [list] |
| Em-dash count | N | >2/para flagged | [PASS/FLAG] |
| Sentence length variance | X.X | <5.0 flagged | [PASS/FLAG] |
| Connector logic blocks | N blocks | types: [...] | [PASS/FLAG] |
| Parenthesis density | X/para | >0.8/para flagged | [PASS/FLAG] |
| Bold-sub-total paragraphs | N | ≥2 flagged | [PASS/FLAG] |
| GenAI risk tier | [high/moderate/low] | score [N]/19 | [Action] |

### GenAI Risk Breakdown
| Factor | Value | Contribution |
|--------|-------|-------------|
| AI vocab density | X% | [+N to score] |
| Mechanical connector density | X% | [+N to score] |
| Vocabulary diversity | X% | [+N to score] |
| Sentence length variance | X.X | [+N to score] |
| Bullet density | X% | [+N to score] |
| Hedge stack density | X | [+N to score] |
| Filler density | X | [+N to score] |
| Significance inflation | N | [+N to score] |
| Parenthesis density | X/para | [+N to score] |
| Bold-sub-total paragraphs | N | [+N to score] |
| **Total Score** | **N/19** | **→ [high (≥5) / moderate (1-4) / low (0)]** |

### Connector Logic Blocks
| # | Sequence Type | Markers | Block Text (first 80 chars) |
|---|--------------|---------|---------------------------|
| 1 | first-second-third | First, Second, Third, Finally | ... |

### Semantic Analysis (LLM)
[Template paragraph structure, tone issues, context-sensitive false positives]

### Prose Zones
| Category | Hits | Examples Found |
|----------|------|----------------|
| ... | ... | ... |

### Code Comment Zones (Script + LLM verified)
| Tier | Count | Examples |
|------|-------|----------|
| Tier 1 - KEEP | N | ... |
| Tier 2 - REWRITE | N | ... |
| Tier 3 - REMOVE | N | ... |

### Print / String AI Patterns (⚠️ Report Only)
| Location | Current | Suggested Rewrite |
|----------|---------|-------------------|
| ... | ... | ... |

### Code Style Flags (advisory)
- [List]

**Overall assessment**: [Natural / Mild AI traces / Obvious AI traces / Heavy mechanization]
```

### Step 7: Pause for User Confirmation

**STOP** and ask:

> "Diagnostic complete. Prose: [N] AI traces found, [N] bullet sections suggested for prose conversion, total-sub-total [Yes/No]. Comments: Tier2 [N] / Tier3 [N].\n> \n> ⚠️ Advisory (unchanged by default): Print/String [N] / Code Style [N]\n> \n> Proceed? 'Yes' = process all prose + comments."

Do NOT proceed until the user confirms.

### Step 8: Execute De-AI Rewrite

**Core principle**: You are an editor, not a regex engine. Read for meaning first. The script's pattern flags tell you WHERE to look — they do not tell you WHAT to write. Your output must read like a human wrote it from scratch, and it must survive a statistical detector scan (perplexity + burstiness).

**Process for prose**:

1. **Read the paragraph** as a whole. What is it actually saying? Strip the decoration and state the core argument in one sentence in your head.

2. **Rewrite from that core**. Express the same idea naturally, as if explaining it to a colleague. Don't look at the original while you write — look at your understanding of it.

3. **Check against the flags**. After writing, glance at the script's flag list. Did any flagged words survive? If yes: is each one genuinely carrying weight in THIS sentence, or is it decoration? If decoration, refine.

4. **Perplexity check**. Read your rewrite and ask: "If I were a language model, would I have predicted every word in this sentence?" If yes — vary 2-3 words to equally natural but less predictable alternatives. Example: "showed improvement" → "demonstrated gains" / "yielded better results". The goal is unpredictability without weirdness.

5. **Burstiness check**. Count words per sentence. If 3+ consecutive sentences are within ±3 words of each other, break the pattern. Insert a fragment. Merge two into a complex sentence.

**For prose with heavy bullet points** (script: bullet_density >30% or has_total_sub_total):

The bullet-point structure itself is both an AI surface pattern AND a burstiness tell. Converting bullet-by-bullet into prose sentences is insufficient — that just produces uniform-length sentences with punctuation instead of bullets.

Instead:
1. **Identify the single argument** the bullet list is trying to make.
2. **Write one coherent paragraph** that makes that argument, pulling the key data points as supporting evidence at varying depths.
3. **Drop any bullets** that are decorative or repetitive.
4. **Deliberately vary sentence architecture** — don't let it settle into "one bullet = one sentence."

For long bullet-heavy sections, delegate to a sub-agent to keep context clean:

```
task(
  category="deep",
  load_skills=[],
  run_in_background=false,
  prompt="""Rewrite the following academic text. It currently uses bullet points
and a structured format that reads like AI output — both in surface patterns
and in its underlying statistical signature (low burstiness, uniform sentence length).

Your job:
1. Read the entire text and identify the core argument(s).
2. Rewrite from scratch as flowing academic prose with high burstiness — deliberately
   vary sentence length: short punchy sentences, then longer explanatory ones,
   occasionally a fragment for rhythm.
3. Use varied vocabulary (maintain perplexity). Don't let every word be the most
   predictable choice.
4. Preserve all technical data, numbers, citations, and LaTeX commands.
5. Drop any bullet points that are decorative, repetitive, or merely restate
   the section heading.
6. Output only the rewritten prose. No explanations.

[INPUT TEXT]:
[paste the bullet-heavy prose here]"""
)
```

**For connector logic blocks** (script: connector_blocks not empty):

**Script-LLM division of labor**:
1. **Script** finds candidate blocks: ≥2 sequential markers within 200 words.
2. **LLM** verifies each candidate: does "First" mean "the first argument/point" (signposting → rewrite) or "the first experimental subject/object" (legitimate ordinal → keep)?

The entire signposting block must be rewritten as ONE coherent argument. Do NOT strip markers and leave sentences otherwise unchanged. Do NOT synonym-swap markers ("First" → "To begin").

**For prose flagged by GenAI risk tier**:

| Risk Tier | Score | Action |
|-----------|-------|--------|
| **high** | ≥5/16 | **Flag for de-AI rewrite** (NOT re-polish). User can override with "skip high risk." |
| **moderate** | 2-4/16 | **Warn the user**: "GenAI risk: moderate (score [N]/16)." User decides. |
| **low** | 0-1/16 | **Skip de-AI rewrite**. Inform user: "GenAI risk is low — the text's statistical features resemble human writing." Then ask: "Would you like to run polish or --top polish on it?" |

**Edge case — high risk with no structural patterns**: Check if the high AI vocab density is caused by **domain-specific technical terminology** being incorrectly matched (e.g., "robust optimization" in OR is technical, not AI decoration). If yes → warn, do NOT rewrite. If no → flag for de-AI rewrite.

**For code comments**: Script classification + LLM verification:
- Tier 1: Keep.
- Tier 2: Rewrite to be concise and natural. Strip decoration. Preserve comment syntax.
- Tier 3: Remove.
- Comments with bullet/numbering: If ≤10 words AND explains why → keep. Otherwise → **delete**.

**For print/log strings**: Report only. No automatic changes.

**Code body / Math / Inline Code**: Output verbatim.

Reassemble all zones in original order.

### Step 9: Self-Audit — Anti-AI Smell Check

After the first rewrite, perform a critical self-audit from two perspectives:

**Perspective A — Surface patterns**: "If I were a reviewer who has read 1000 AI-generated papers, what in this rewritten text would still make me suspect it's machine-written?"

**Perspective B — Statistical fingerprint** (for detector evasion): "If I fed this text to GPTZero, would the perplexity and burstiness scores flag it? Why?"

Common tells to check:

| Residual Pattern | Check |
|-----------------|-------|
| Rhythm uniformity (burstiness) | Are 3+ consecutive sentences within ±3 words of each other? |
| Predictable vocabulary (perplexity) | Are there stretches where every word is the most obvious choice? |
| Connector residue | Did "Moreover" become "Furthermore" instead of being cut? |
| Bullet conversion quality | Did bullets become prose, or just bullets with punctuation added? |
| Template feel | Does each paragraph still open the same way? |
| Vocabulary swap | Did "leverage" → "utilize" (low-perplexity synonym swap — still detectable)? |
| Hedging stack | Did "may potentially" → "might possibly"? |
| Clean but soulless | Perfect grammar, predictable word choices, uniform rhythm — the detector trifecta |

### Step 10: Second Pass Fix

Based on the audit bullets, make ONE MORE targeted editing pass:
- **Surgical** — fix only what the audit identified.
- **Rhythm-first**: vary 2-3 sentence lengths.
- **Cut, don't replace**: DELETE mechanical connectors rather than finding synonyms.
- **Add a pulse**: inject one sentence of genuine reaction where appropriate.

### Step 11: Output

Structure output in three parts:
- **Part 1 [Final Text]**: The rewritten text (LaTeX compatible, special chars escaped).
- **Part 2 [Literal Translation]**: A direct Chinese translation for the user to verify logic is preserved.
- **Part 3 [Modification Log]**: Chinese bullet points summarizing what was changed.

If the text was already natural: inform the user, then ask whether they want polish.

### Key Principle: Better Omitted Than Done Poorly
If the input text is already natural and native-like, do NOT rewrite just for the sake of rewriting. Inform the user and offer alternatives.

---

## AI Pattern Reference

These are **alerts, not rules**. Each pattern flags something to check. Whether you change it depends on context: a word that is decoration in one sentence may be precise and necessary in another. The script provides hard counts; your job is semantic judgment.

### Guiding Principle

Before touching any pattern, read the full paragraph and answer:

> "If I rewrite this paragraph from scratch — keeping all the facts but using my own words — would this word or phrase still be there?"

If yes → keep it. If no → it's decoration, cut it or find what the sentence actually needs.

### 1. Mechanical Connectors

**Script catches**: `First and foremost`, `It is worth noting that`, `It should be noted that`, `In light of the above`, `Taken together`, `In a nutshell`, `Notably,`

**What to do**: These are almost always decoration. But don't just delete them — that leaves the sentence orphaned. Instead, **rewrite the sentence** so it starts naturally.

```
BEFORE: First and foremost, our results show a 12% improvement. Moreover,
        the ablation study confirms this trend. Furthermore, qualitative
        analysis supports our conclusion.
AFTER:  Our results show a 12% improvement across all benchmarks.
        Ablation confirms that each component contributes. Qualitative
        analysis of the output reveals consistent patterns.
```

**Beware**: Deleting "Moreover" and leaving "the ablation study confirms this trend" unchanged is still AI writing — just with fewer words. The sentence itself needs to change.

**Exception**: `Moreover` / `Furthermore` / `Additionally` appearing once in a long paragraph is fine — it's the stacking (3+ in proximity) that signals AI rhythm.

### 2. Overused AI Vocabulary

**Script catches**: `delve into`, `leverage`, `tapestry`, `realm`, `crucial`, `pivotal`, `showcase`, `robust`, `intricate`, `underscore`, `landscape`, `paradigm`, `groundbreaking`, `holistic`, `synergistic`, `transformative`

**What to do**: For each flagged word, ask: "Is this word carrying weight, or just sounding important?"

- `leverage` is legitimate when describing a specific technical mechanism. It's decoration when it just means "use."
- `crucial` / `pivotal` — if the thing is genuinely make-or-break, keep it.
- `robust` — legitimate for robustness to noise/attacks. Decoration when it just means "works well."
- `tapestry`, `realm`, `intricate` — almost never justified in technical writing. Cut them.
- `showcase` — legitimate for demos/tables/figures. Decoration when it means "show."

**The rewrite test**: Don't find-and-replace. Read the sentence, drop the flagged word, and ask what the sentence actually needs there. Sometimes the answer is nothing. Sometimes it's a more specific word.

```
BEFORE: This framework leverages a robust pipeline to delve into the intricate
        tapestry of multimodal interactions and showcase the crucial findings.
STEP 1 (drop decoration): This framework [uses] a [working] pipeline to
        [study] multimodal interactions and [present] the [key] findings.
STEP 2 (add specificity): This framework processes multimodal data through
        a three-stage pipeline and presents the resulting interaction patterns.
```

### 3. Uniform Sentence Rhythm

**Script catches**: sentence_length_variance < 5.0 with >5 sentences.

**What to do**: Vary rhythm intentionally. Break the pattern: insert a 5-word sentence, merge two medium ones into a 35-word complex sentence, or split a long one into fragments.

```
BEFORE: The model achieves SOTA on all benchmarks. (9 words)
        It outperforms previous methods by 12%. (8 words)
        The improvement is consistent across datasets. (7 words)
        These results validate our choices. (7 words)
AFTER:  The model achieves state-of-the-art performance on all benchmarks,
        outperforming previous methods by 12–18% across five datasets. (20 words)
        The improvement holds. (3 words)
        It validates our choices. (5 words)
```

### 4. Overuse of Em-Dash (—)

**Script catches**: >2 em-dashes in any paragraph.

**What to do**: Replace with commas, parentheses, or by restructuring the clause. One em-dash in a paper is fine; a cluster is AI.

### 5. Template Paragraph Structure

**Script can't catch — LLM judgment required.**

**What to check**: Do 3+ consecutive paragraphs open the same way? Does every paragraph follow "Topic sentence → evidence → transition"?

**Fix**: Vary paragraph openers. Start one with a finding, the next with a question, another with a specific data point.

### 6. Excessive Hedging

**Script catches**: `may potentially suggest`, `could possibly indicate`, `might perhaps be`, `could potentially be argued`, `it may be possible that`.

**What to do**: Use ONE qualifier: "may" or "suggests" or "is consistent with."

### 7. Copula Avoidance

**Script can't catch — LLM judgment required.**

**What to check**: `serves as`, `stands as`, `represents a`, `boasts`, `features` where a simple `is`/`are`/`has` would work.

### 8. Superficial -ing Analyses

**Script can't catch — LLM judgment required.**

**What to check**: Sentences ending with `, highlighting...` / `, underscoring...` / `, reflecting...`. Cut the -ing phrase. If the point matters, move it to a separate sentence with evidence.

### 9. Significance Inflation

**Script catches**: `marks a pivotal moment`, `serves as a testament`, `underscores its vital role`, `sets the stage for`, `represents a paradigm shift`, `a turning point in`, `reshaping the landscape`, `at the forefront of`.

**What to do**: State facts. Let the reader decide if it's a "paradigm shift."

### 10. Filler Phrases

**Script catches**: `In order to`, `Due to the fact that`, `At this point in time`, `In the event that`, `Has the ability to`, `It is important to note that`.

**What to do**: Cut to 1-2 words. "In order to achieve" → "To achieve". "It is important to note that the data shows" → "The data shows."

### 11. Excessive Parenthetical Over-Explanation

**Script catches**: parenthesis_density > 0.8 per sentence.

**What to check**: Too many parenthetical asides in close succession. AI text over-explains by stuffing clarifications, examples, or alternative names into parentheses — e.g., "Our method (a transformer-based architecture) achieves SOTA (95.2% accuracy) on five benchmarks (see Table 2)."

**What to do**: Evaluate each parenthetical:
- Can it be integrated into the sentence flow without parentheses?
- Can it be removed entirely (reader doesn't need this clarification)?
- Is it a genuine citation or reference (`(see Fig. 2)`, `(§3.2)`) — those are fine.
- Break the cluster: if 3+ parentheticals appear in 2 sentences, spread them out or cut some.

```
BEFORE: The model (a 12-layer transformer) achieves 95.2% accuracy (5.3% improvement)
        on ImageNet (1000-class) using our method (Algorithm 1).
AFTER:  Our 12-layer transformer achieves 95.2% accuracy on ImageNet (1000 classes),
        a 5.3% improvement over the baseline (Algorithm 1).
```

**Exception**: Math/technical writing may legitimately use parenthetical definitions `(where x ∈ R^n)`. Don't flag these — focus on explanatory/contextual parentheses.

### 12. Bold-Header + Sub-Total in Small Paragraphs

**Script catches**: ≥2 small paragraphs (min(100, 10% of text) words) with bold-header openers and sub-total internal structure.

**What to check**: Small paragraphs that open with `**bold term**:` followed by a description, then sub-points or sub-categories, then a summary sentence. This is a miniaturized version of the total-sub-total template and is highly characteristic of AI-generated survey sections or related work.

**What to do**: For each flagged paragraph:
- Check if the bold header actually adds value or is decorative.
- If the paragraph is genuinely short (<3 sentences), consider whether it needs to exist at all or could be merged.
- Reorder: try stating the finding/summary first, then explaining.
- Break the template by varying opener style — use a question, a data point, or a contrast instead of `**Term**: definition`.

```
BEFORE: **Transformer**: A neural architecture based on self-attention. It processes
        sequences in parallel. Key components include multi-head attention and FFN.
        Transformers are widely used in NLP and CV.

AFTER:  Self-attention is the core idea behind transformers. By processing sequences
        in parallel rather than sequentially, architectures like multi-head attention
        and FFN layers achieve strong results across NLP and computer vision tasks.
```

---

## Paraphrase for Similarity Reduction (Optional)

Use when the user specifically asks to reduce similarity to published work:

```
Please paraphrase the following text to lower the similar phrase to academic publications. You can replace phrases or reorder the content. Please do not replace the academic terms and do keep the passive and active tone consistent. Please also make sure your output in an academic tone. To avoid the AI detection, the text needs to have proper amount of perplexity and burstiness. Summarize what you have done to the text in the end of your output in Chinese:

[INSERT TEXT HERE]
```

---
> Source: [AsdfAlex-learning/free-indulgence](https://github.com/AsdfAlex-learning/free-indulgence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
