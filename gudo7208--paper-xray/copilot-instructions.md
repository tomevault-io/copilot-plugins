## paper-xray

> |


# Paper X-Ray: Paper Analysis & Topic Research

> Four modes: Single deep analysis / Batch parallel / Quick triage / Topic auto-survey.

## Configuration

Customize these paths before first use. All paths are relative to your workspace root (current working directory).

| Variable | Description | Default |
|----------|-------------|---------|
| NOTES_DIR | Paper notes directory | `papers/notes/` |
| READING_LIST | Reading list file path | `papers/reading-list.md` |
| INDEX_FILE | Paper notes index file | `papers/notes/index.md` |
| COGNITIVE_BASELINE | Cognitive baseline files for personal reflection (optional) | Not enabled |
| RESEARCH_FOCUS | Default research direction for triage mode | Must be provided by user |

## Usage

```
# Single mode
/paper-xray <paper name | arxiv ID | DOI | URL | PDF path>

# Batch mode
/paper-xray batch: <id1> <id2> <id3> ... [category: <topic>]

# Triage mode (quick assessment, no notes generated)
/paper-xray triage: <id1> <id2> ... [context: <research direction>]

# Topic research mode
/paper-xray topic: <research topic> [scope: <constraints>] [n: <paper count>]
```

Single mode examples:
- `/paper-xray Attention Is All You Need`
- `/paper-xray 2301.07041`
- `/paper-xray https://arxiv.org/abs/2301.07041`
- `/paper-xray ~/Downloads/paper.pdf`

Batch mode examples:
- `/paper-xray batch: 2603.04756 2504.05738 2603.05344 category: Agent`

Triage mode examples:
- `/paper-xray triage: 2603.04756 2504.05738 context: AI-assisted software engineering`

Topic mode examples:
- `/paper-xray topic: Agent in Software Engineering`
- `/paper-xray topic: Code Generation Evaluation Benchmarks scope: 2024-2026 n: 8`

## Mode Routing

Auto-detect mode based on input:
- Contains `batch:` prefix → Batch mode
- Contains `triage:` prefix → Triage mode
- Contains `topic:` prefix → Topic research mode
- Matches arxiv ID / DOI / URL / PDF path / specific paper name → Single mode
- Ambiguous (e.g., "latest LLM Agent progress") → Default to Topic mode

## Paper Type Detection

After fetching a paper, auto-detect type and use corresponding template:

| Type | Identification | Template |
|------|---------------|----------|
| Research Paper | Has clear Method + Experiments, proposes new method/framework | `references/template.md` |
| Technical Report | Title contains model name (GPT/Claude/Qwen etc.), architecture+training+eval focused, no controlled experiments | `references/tech-report-template.md` |
| Survey Paper | Title contains Survey/Review/Taxonomy, primarily categorization | `references/survey-analysis-template.md` |

Priority: Technical Report > Survey > Research Paper (default).

## Principles

- **Objectivity first**: The analysis subject is the paper itself, not personal opinions. Deconstruct the paper first, then discuss personal insights.
- **Precise terminology**: Target audience is researchers/engineers. Use proper terms, but define key concepts clearly.
- **Honest uncertainty**: Distinguish "experimentally proven" from "author speculation". Mark insufficient information, don't fabricate.
- **Structured, not narrative**: Use tables, bullet points, comparisons. No essays.

---

## Single Mode Execution Flow

### Step 1: Fetch Paper

Route based on input type:

| Input Format | Recognition Rule | Fetch Strategy |
|---------|---------|---------|
| `2301.07041` or `arxiv:2301.07041` | Digits+dot or arxiv: prefix | → `https://arxiv.org/html/{id}` WebFetch |
| `https://arxiv.org/abs/...` | arxiv abs URL | → Replace with `/html/` WebFetch |
| `https://arxiv.org/pdf/...` | arxiv pdf URL | → Replace with `/html/` WebFetch |
| `https://arxiv.org/html/...` | arxiv html URL | → Direct WebFetch |
| `10.xxxx/...` | DOI format | → WebSearch for arxiv/html version |
| `~/xxx.pdf` or `/xxx.pdf` | Local path | → Read tool to read PDF |
| Other URL | Generic URL | → WebFetch |
| Paper name (natural language) | No match above | → WebSearch, prefer arxiv html |

**Fetch priority**: arxiv HTML > other HTML > PDF.

**Robust fetch strategy** (arxiv papers):
1. Try `https://arxiv.org/html/{id}`
2. If HTML unavailable (404 or incomplete), try `https://arxiv.org/abs/{id}` for abstract page
3. If abs page also fails, WebSearch `arxiv {id}` for alternative sources
4. Last resort: WebSearch paper title for any available version

**Long paper handling**: Fetch Abstract + Introduction + Conclusion first for global understanding, then fetch Method, Experiments etc. as needed. Don't dump everything at once.

**Metadata extraction**: Title, authors, publication date, venue, arxiv ID (if available).

**arxiv ID year parsing**: ID format `YYMM.NNNNN`, first two digits = year. `2602.09540` = Feb 2026, `2301.07041` = Jan 2023.

### Step 2: Problem & Motivation Analysis

- **Problem definition**: 1-2 sentences precisely describing the core problem
- **Existing method limitations**: Where previous work got stuck — theoretical/engineering/data bottleneck?
- **Key Insight**: What did the authors see that others missed?

### Step 3: Technical Approach Deconstruction

- **Overall architecture**: Mermaid diagram or structured description of the system/method flow
- **Key technical contributions**: Break down 2-3 core contributions, each explaining: what it is, why, and how it fundamentally differs from existing methods
- **Key formulas/algorithms**: Core formulas + symbol meanings + intuition. Don't skip the math, but make it understandable
- **Design choices & trade-offs**: Where did the authors make trade-offs?

### Step 4: Experiments & Results Evaluation

- **Experimental setup**: Datasets, baselines, evaluation metrics
- **Main results**: Key numbers in tables, annotate the most important comparisons
- **Ablation studies**: Contribution of each component
- **Result credibility**: Statistical significance, conclusion boundaries, reproducibility (code/data/hyperparameters)

### Step 5: Contributions & Limitations

- **Core contributions**: Substantive advancement to the field (not just restating the abstract)
- **Limitations**: Author-acknowledged + unmentioned but present
- **Open questions**: What remains unsolved?

### Step 6: Field Positioning & Connections

- **Academic lineage**: Who does it build on? Who does it complement/contradict?
- **Impact assessment**: Opens new direction / improves existing method / provides new evidence
- **Recommended reading**: 2-3 papers most worth reading next + reasons

### Step 6.5: Deep Reading — What the Paper Doesn't Say

- **Implicit assumptions**: Unstated assumptions — what happens if they don't hold?
- **Deliberately avoided questions**: Obvious comparisons/scenarios not discussed
- **Stories behind the data**: Anomalous numbers in results
- **Deeper perspective**: What does the methodology choice reveal about the authors' understanding?
- **If I were the author's next step**: Most valuable thing the authors didn't do

### Step 6.6: Reviewer Assessment

Structured review from a top-venue reviewer perspective:

- **Overall rating**: Strong Accept / Weak Accept / Borderline / Weak Reject / Strong Reject
- **Strengths** (3-5 points)
- **Weaknesses** (3-5 points, each with: issue, why it matters, how to improve)
- **Questions to Authors**: 2-3 questions targeting the weakest points
- **Missing References**
- **Verdict**: One sentence summary

Reviews should be fair but sharp. Good papers have weaknesses, bad papers have strengths. No pleasantries.

> **Note**: Technical reports (model releases) skip this step, use competitive analysis from the tech-report template instead.

### Step 6.7: Insights & Action Items

- **Directly reusable**: Methods, datasets, evaluation frameworks, code
- **Borrowable ideas**: Problem framing, experimental design, narrative structure
- **Triggered new ideas**: New research questions or engineering ideas
- **Action items**: Specific next steps

### Step 7 (Conditional): Personal Cognitive Reflection

> This step is optional and only executes when configured. Users can enable it by setting `COGNITIVE_BASELINE` in the Configuration section to point to their personal reflection files.

If `COGNITIVE_BASELINE` is configured:
1. Read the specified cognitive baseline files
2. Find collision points: paper findings vs personal beliefs → reinforce/revise/overturn?
3. Output a one-sentence delta. delta ≈ 0 is a normal result — don't force it.

If not configured, skip this step entirely.

### Step 8: Archive to Workspace

#### 8.1 Determine Topic Category

Categorize paper into topic directory: `{NOTES_DIR}/{topic}/`

Common topics: Agent / LLM / Knowledge Representation / Software Engineering / Multimodal / RL / AI Safety. Create new topics as needed.

#### 8.2 Generate Report File

Select template based on paper type (`template.md` / `tech-report-template.md` / `survey-analysis-template.md`).

File naming: `{Paper Title}.md`, use concise recognizable English. Save to `{NOTES_DIR}/{topic}/`.

#### 8.3 Auto Knowledge Network Links

1. Search existing notes in the workspace for related papers using keyword matching
2. Add `[[Related Paper]]` links in the new report's "Knowledge Network" section + relationship description
3. **Reverse update**: In existing related paper notes, append a link back to the new paper

#### 8.4 Update Index

- Update `{NOTES_DIR}/{topic}/_index.md` (create if not exists)
- Update `{INDEX_FILE}` under the corresponding topic category

#### 8.5 Update Reading List

Check `{READING_LIST}`: if the paper has an entry, mark `[ ]` → `[x]` and append ` → [[{Paper Title}]]`.

#### 8.6 Custom Post-Processing (Optional)

If a `post-archive.sh` script exists in the skill directory, execute it after archiving. This allows users to integrate custom index builders or other automation.

---

## Batch Mode Execution Flow

Enters this mode when input contains `batch:` prefix.

### B1: Parse Input

Extract paper list (arxiv IDs / URLs / names) from after `batch:`, plus optional `category:` topic.

If no category specified, auto-determine topic from paper content.

### B2: Parallel Analysis

Launch independent Agent per paper, executing Single mode steps 1 through 6.7 (steps 2-6.7 for analysis, step 8 for archiving).

Each Agent's prompt template:
```
You are a paper analysis expert. Perform deep structural analysis on the following paper.

Paper: {arxiv ID or title}
Fetch method: WebFetch https://arxiv.org/html/{id} (if fails, try abs page or WebSearch)

Execute Single mode steps 1 through 6.7 completely, output structured report.

Paper type detection: Determine if research paper/technical report/survey based on content, use corresponding template structure.

After analysis, save report to: {NOTES_DIR}/{topic}/{Paper Title}.md

Note:
- Only write the paper note file itself, do NOT update _index.md or index files (main process handles this)
- In the "Knowledge Network" section, search workspace for related notes and add [[]] links
- Check reading list, mark entry as complete if found
```

Agent configuration: model: sonnet, subagent_type: general-purpose

### B3: Collect Results & Unified Index Update

After all Agents complete:

1. **Unified index update** (avoid concurrent conflicts):
   - Read current `_index.md`, append all new paper entries at once
   - Read current index file, append all new entries at once
2. **Reverse links**: For all new papers, append reverse `[[]]` links in existing related paper notes
3. **Run post-processing** if `post-archive.sh` exists

### B4: Cross-Paper Insights Summary

Generate a brief cross-paper insight (output directly to user, no separate file):

- Relationships between papers (complementary/competing/progressive)
- Common trends they point to
- 1-2 papers most worth diving deeper into

### B5: Report Completion

Report to user: each paper's save path, assigned topic, number of links established.

---

## Triage Mode Execution Flow

Enters this mode when input contains `triage:` prefix. Quick relevance assessment, no full notes generated.

### TR1: Parse Input

Extract paper list and optional `context:` research direction description.

If no context provided and `RESEARCH_FOCUS` is configured, use it as the default research direction. Otherwise, ask the user to specify their research direction.

### TR2: Quick Abstract Fetch

Fetch Abstract for each paper in parallel (only need abs page, not full text):
- arxiv ID → WebFetch `https://arxiv.org/abs/{id}`
- Other formats → WebSearch for abstract

### TR3: Assess & Rank

For each paper output:

| Paper | Relevance | Novelty | Practicality | Recommendation | Reason |
|-------|-----------|---------|-------------|----------------|--------|
| {title} | High/Med/Low | High/Med/Low | High/Med/Low | Must-read / Worth-reading / Skip | {one-sentence reason} |

Assessment dimensions:
- **Relevance**: Direct connection to the context research direction
- **Novelty**: New method/perspective, or incremental improvement
- **Practicality**: Directly useful inspiration for current work

### TR4: Output Recommendations

Output results table sorted by recommendation, with suggestions:
- "Must-read" papers → suggest using `/paper-xray batch:` for batch processing
- "Skip" papers → explain skip reason

No files generated, no index updates. Pure conversation output.

---

## Topic Research Mode Execution Flow

Enters this mode when input contains `topic:` or is judged to be a research topic.

### T1: Parse Research Question

Extract: core topic, scope constraints (time/venue), paper count (default 5-8, `n:` to specify).

Break topic into 2-3 search dimensions.

### T2: Multi-Round Search — Build Candidate Pool

Use WebSearch with multiple search dimensions in parallel (via Agent tool):

```
For each search dimension, launch an Agent to:
1. WebSearch with dimension-specific keywords
2. Collect arxiv IDs, titles, and brief descriptions
3. Return structured results
```

Merge all results, deduplicate, sort by relevance.

### T3: Filter — Determine Analysis Paper List

Filtering principles:
- **Coverage**: Cover different sub-directions of the topic
- **Representativeness**: Prioritize foundational work + latest advances
- **Diversity**: At least one each of method/system/evaluation/survey
- **Deduplication**: Keep only the more representative of highly similar papers

Present filtered results to user (paper list + selection rationale), confirm before continuing.

### T4: Parallel Deep Analysis

Launch independent Agent per paper, executing Single mode steps 1 through 6.7 (analysis only, no cognitive reflection).

Agent prompts include topic context, requiring analysis of paper's relationship to the topic.

**Important**: Each Agent only writes the paper note file, does NOT update index files (main process handles this).

### T5: Synthesis — Generate Survey Report

After collecting all Agent results:

1. **Technical approach comparison**: Build comparison matrix
2. **Field evolution**: Timeline, inheritance relationships, technical route analysis, trends
3. **Consensus & disagreements**: Common conclusions, contradictions, blank directions
4. **Research frontier assessment**: Most promising directions, biggest unsolved problems, suggested entry points

### T6: Archive

1. **Unified index update**: Batch update all `_index.md` and index files
2. **Bidirectional links**: Establish `[[]]` links between papers
3. **Generate survey report**: Save using `references/survey-template.md` to `{NOTES_DIR}/{topic}/!survey-{topic-short}-{date}.md` (`!` prefix ensures it sorts first in directory)
4. **Run post-processing** if `post-archive.sh` exists

---

## Output Quality Standards

- Main body is objective analysis (steps 2-6), comprising 80%+ of the report
- Precise terminology, key concepts defined
- Mathematical formulas preserved and explained
- Experimental evaluation is critical, not just restating the results section
- Personal reflection is optional and never forced
- Structured presentation: tables, bullet points, comparisons — no essays

---
> Source: [gudo7208/paper-xray](https://github.com/gudo7208/paper-xray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
