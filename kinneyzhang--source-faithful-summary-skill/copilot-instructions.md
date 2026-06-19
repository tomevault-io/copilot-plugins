## source-faithful-summary-skill

> Use when summarizing text, video transcripts, audio transcripts, podcasts, lectures, interviews, demos, or meetings where the output must preserve the source's main thread, attention weight, decision model, concrete examples, and evidence instead of becoming a polished but source-drifting essay.


# Source-Faithful Summary Skill

## Overview

Use this skill for source-grounded summaries of text, video transcripts, audio transcripts, podcasts, talks, interviews, demos, workshops, lectures, meetings, and other transcript-like source material.

The goal is not to write the most elegant essay. The goal is to preserve what the source actually does: the speaker's main thread, attention weight, concrete examples, sequence, failure modes, actions, and decision model.

A summary can be factually plausible and beautifully written while still being wrong for the task if it replaces the speaker's actual flow with the summarizer's cleaner framework.

Core rule:

```text
Faithfulness to the source > elegance of argument
Narrative weight > concept density
Concrete action/problem chain > abstract framework
Timestamps / section refs / evidence > vibes
Speaker/author/participant decision model > isolated tips
```

## What This Skill Optimizes

A good source-grounded summary is not merely compressed source text. It should recreate the understanding a careful reader, viewer, or listener would get after engaging with the source.

Optimize for five kinds of fidelity:

1. **Main-thread fidelity** — what the source is actually driving toward.
2. **Attention fidelity** — what the source spends time on, repeats, or uses to explain later actions/claims.
3. **Causal fidelity** — why one observation leads to the next decision, claim, or forecast.
4. **Action/argument fidelity** — what the speaker/author/participants actually do or argue, in source order.
5. **Decision-model fidelity** — the constraints, tradeoffs, failure modes, and uncertainty behind the source's recommendations or judgments.

For workflow/practice sources, named concepts must not be treated as decorative keywords. A concept is important if it controls later decisions.

Example:

```text
smart zone / dumb zone
```

In an AI coding workflow talk, this is not just a definition about context degradation. It may explain why tasks are sized small, why context is cleared, why external artifacts like PRDs/Kanban boards matter, and why review should happen in a fresh context.

Default extraction primitive:

```text
failure mode → underlying constraint → speaker's judgment → concrete action → tradeoff/limit
```

## When to Use

Use when the user asks to:

- summarize a YouTube, Bilibili, Vimeo, podcast, interview, lecture, demo, workshop, or recorded meeting;
- rewrite or publish a source-based summary;
- audit whether an existing summary missed the original material's key points;
- extract a speaker's workflow, examples, failure modes, or practical advice;
- turn a long transcript into a public article or private note;
- compare a summary against a source transcript;
- preserve what the source says before writing independent commentary.

Also use when the source is not literally video but has a strong original sequence, such as an interview article, lecture notes, meeting transcript, or conference talk transcript.

Do **not** use this as the primary workflow when the user explicitly wants:

- a free-form opinion essay inspired by a source;
- a broad research article that uses the material as one source among many;
- creative rewriting that intentionally departs from the source;
- marketing copy where source fidelity is not required.

If commentary is needed, first produce the source-faithful summary, then add a clearly separated section such as “Analysis”, “Implications”, or “What this means for us”.

## Required Inputs

Prefer one or more of:

- transcript with timestamps;
- source URL plus transcript tool access;
- existing summary to audit;
- target audience and output length;
- whether the result is public-facing or private notes;
- desired language.

If the transcript is unavailable, state the limitation explicitly and summarize only from available material.


## Output Language Policy

Source language and output language are separate decisions. Do not assume the output language should match the source language.

Use this priority order:

1. **Explicit user request wins.** If the user says “write in Chinese”, “English output”, “用中文”, “生成英文版”, or names a target locale, use that language.
2. **Conversation language next.** If the user does not specify, write in the language the user used to ask for the summary. A Chinese request gets Chinese output; an English request gets English output.
3. **Existing artifact language next.** If revising an existing article or note, preserve its language unless the user asks to change it.
4. **Source language last.** Only mirror the source language when none of the above is available.

For multilingual or international users, ask only when the language signal is genuinely ambiguous. Otherwise choose and state the choice in metadata, for example:

```markdown
Output language: Chinese, chosen from the user request language.
```

Keep source-specific names, tool names, commands, repository names, product names, and technical terms in their original form unless the user explicitly requests translation.

Before finalizing, run a language audit:

- Is the main body in the chosen output language?
- Are only proper nouns / commands / quoted source phrases left untranslated?
- If the source is English but the user asked in Chinese, did we avoid translationese while preserving exact technical terms?
- If the user asked in English, did we avoid accidentally producing Chinese headings or commentary?
- For Chinese reader-facing articles, avoid the word “忠实” in the article title, headings, metadata labels, and body prose. The method can be source-faithful, but Chinese readers usually find literal labels like “一句话忠实总结” awkward. Prefer natural labels such as “一句话概括”, “主轴”, “基于原文的总结”, or simply omit the methodology label.

## Standard Workflow

### Step 1 — Fetch and validate source material

Get transcript or source text before summarizing.

Validation checklist:

- source text is non-empty;
- language is known;
- duration/segment count looks plausible if video/audio;
- timestamps are preserved when available;
- missing transcript or poor subtitle quality is noted.

If transcript fetch fails, retry with another language or source when possible. Do not hallucinate details from the title alone.

For extracted webpages/articles, run a source-quality check:

- Were headings preserved?
- Were figures, diagrams, tables, footnotes, code blocks, or links lost?
- Is the text extracted from the full article or a preview/summary?
- Does the final summary need to say “based on text extraction; figures/tables may be missing”?

### Step 2 — Classify the source type

Do not write the summary yet. First classify the material:

```text
1. Viewpoint talk: core is a thesis and supporting arguments.
2. Practical/workflow talk: core is a sequence of problems and actions.
3. Tutorial/demo: core is steps, commands, screens, and gotchas.
4. Interview: core is the guest's experiences, claims, and examples.
5. Product/update talk: core is features, changes, migration impact.
6. Research/lecture: core is definitions, evidence, method, conclusion.
7. Debate/panel: core is positions, disagreements, and evidence.
8. Technical guidance article / practical engineering article: core is problem/constraint → design choice → pattern taxonomy → selection criteria → caveats.
9. Long audio/podcast: core is speaker roles → chapter map → recurring themes → disagreements/uncertainties → evidence-backed synthesis.
```

Write one line before drafting:

```text
Source type: <type>; primary summary spine should be <argument chain / action chain / step chain / Q&A chain / feature-impact chain / disagreement map / constraint→judgment map>.
```

If the material is practical/workflow, the default spine must be:

```text
problem → failure mode → speaker's action → tool/prompt/process → result/lesson
```

If the material is a technical guidance article, the default spine should be:

```text
problem/constraint → design choice → pattern taxonomy → selection criteria → caveats → implementation notes
```

If the material is a long audio/podcast or panel, the default spine should be:

```text
speaker roles → chapter/topic map → claims by participant → disagreement/uncertainty map → evidence-backed synthesis
```

### Step 3 — Build a source map

Create a source map before prose. It should include:

```text
- Timestamp range or source section
- What happens in this segment
- Speaker's concrete example or claim
- Role: main thread / support concept / example / aside / implication
- Why it matters
```

Minimum shape:

```markdown
## Source map

- 00:00–01:10 — Setup: speaker frames the problem. [main]
- 01:10–03:40 — Failure mode: what breaks in the default workflow. [main]
- 03:40–05:15 — Concept used to explain the failure. [support]
```

Do not collapse practical details into abstract concepts during this step.

For text/articles, use section headings, paragraph ranges, or stable anchors instead of timestamps. For long audio/video, use a two-level source map:

1. **Chapter-level map** — every major topic/section in source order.
2. **Evidence-level map** — timestamped or section-specific anchors for claims used in the final summary.

For long sources, explicitly state what was fully read, chunked, sampled, or searched.

### Step 3.5 — Activate long-source protocol when needed

Use this protocol when any source exceeds roughly 50k characters, 60 minutes, 20 source-map nodes, or has multiple speakers and strong topic shifts.

Before summarizing, write a compact **long-source plan**:

```markdown
## Long-source plan

- Source size: <duration / chars / sections / speakers>
- Coverage strategy: <full read / chunking / chapter map / keyword recurrence checks>
- Required coverage: beginning, early thesis, middle development, late synthesis, ending summary
- Evidence budget: <which claims require timestamps/section refs>
- Known omissions: <Q&A, side stories, low-value chatter, ads, repeated setup>
```

Coverage rules:

- Cover the opening framing, at least one mid-source development section, and the final synthesis.
- For sources with chapters, include every chapter in the chapter-level map even if the final prose compresses some chapters.
- For repeated named concepts, build a **recurrence index**: first appearance, major reappearances, and final use.
- For demos/workshops, capture screen/action artifacts separately: commands, files, tools, agent actions, feedback loops, tests, and human decisions.
- For podcasts/panels, capture speaker roles and disagreements before writing synthesis.
- State sampling/chunking limitations in the audit if the source was not fully processed.

### Step 4 — Extract named concepts and source roles

List speaker-coined, repeated, or emphasized terms.

For each term, classify it:

- **Controlling concept:** explains several later actions or decisions.
- **Support concept:** helps explain the main thread but is not the spine.
- **Example label:** names one concrete case.
- **Aside:** interesting but not central.

Rules:

- A controlling concept must appear in the summary.
- Support concepts cannot become the headline or spine unless the source itself is a concept lecture.
- If a named concept explains multiple actions, write the action chain it controls.

### Step 5 — Extract the concrete chain

For practical/workflow material, answer:

```text
1. What did the speaker try first?
2. What failed?
3. What symptoms did they observe?
4. What constraint or mental model explains the failure?
5. What judgment did the speaker make because of that constraint?
6. What did they change?
7. What tool/skill/prompt/file/process did they use?
8. What improved?
9. What tradeoff, cost, or remaining limitation did they mention?
```

For AI/software workflow videos, also ask:

```text
- What AI-specific failure modes were named?
- What behaviors of models/agents were observed?
- What context/window/memory limits shaped the workflow?
- What feedback loops were recommended?
- What guardrails were proposed?
- Which parts are safe to delegate, and which require human review?
- Which named concepts control later actions rather than merely illustrate them?
```

### Step 6 — Extract the speaker's decision model

Before drafting, write a compact decision-model map:

```markdown
## Decision model map

- Constraint: <context degradation / forgetting / review overload / missing feedback>
  - Failure it causes: <what goes wrong>
  - Speaker's judgment: <what the speaker decides because of it>
  - Workflow action: <what they do in practice>
  - Evidence: <timestamp/source section>
```

If this map is missing, the summary will usually become a list of tips instead of a faithful account of the speaker's thinking.

### Step 7 — Draft a source-grounded outline

Choose outline by source type.

For practical/workflow videos:

```markdown
# <Speaker/topic>: <actual workflow or problem>

## One-sentence faithful summary
## 1. The problem the speaker was reacting to
## 2. The first failure mode they observed
## 3. The constraint or mental model behind the fix
## 4. The workflow/tool/process they introduced
## 5. The concrete checklist readers/viewers/listeners can copy
## 6. Limits, warnings, and where judgment remains necessary
## Source notes / timestamped facts
```

For viewpoint talks:

```markdown
## One-sentence thesis
## 1. What the speaker argues against
## 2. The speaker's main claim
## 3. Supporting arguments in source order
## 4. Examples and evidence
## 5. What the speaker does and does not claim
## Timestamped facts
```

For tutorials/demos:

```markdown
## What this teaches
## Prerequisites/context
## Step-by-step flow
## Gotchas and failure modes
## Final result
## Timestamped command/action notes
```

For interviews:

```markdown
## Who is speaking and why it matters
## Main claim or experience arc
## Key stories in source order
## Repeated themes
## Concrete examples
## What is claimed vs inferred
```

For long podcasts/panels:

```markdown
## One-sentence faithful summary
## 1. Who is speaking and what role each person plays
## 2. Chapter/topic map in source order
## 3. Major claims by participant
## 4. Disagreements, uncertainty, and forecast boundaries
## 5. Recurring technical/business concepts
## 6. What the source does and does not conclude
## Evidence notes / chapter references
```

For technical guidance articles:

```markdown
## One-sentence faithful summary
## 1. Problem or constraint the article reacts to
## 2. Core distinction or taxonomy
## 3. Pattern/options in source order
## 4. Selection criteria and caveats
## 5. Implementation notes and examples
## Source quality notes
```


### Step 7.5 — Choose delivery mode: working artifact vs reader-facing article

Do not confuse intermediate evidence artifacts with the final reading experience. Source maps, long-source plans, named-concept tables, and decision-model maps are thinking tools. They may be included in an appendix or evaluation report, but they should not automatically become the opening of a public-facing summary article.

Before drafting, choose one delivery mode:

```text
Mode A — Working artifact / audit report
Use when the user needs to inspect evidence, scoring, or methodology.
Order: metadata → source-quality note → source map → named concepts → decision model → summary → audit.

Mode B — Reader-facing source-faithful article
Use when the user asks for an article, publishable summary, or readable note.
Order: title → source line → source type + one-sentence faithful summary → numbered narrative sections → decision model/checklist → source notes appendix.
```

For reader-facing articles:

- The title should name the source and the actual workflow/problem, not a generic lesson.
- Start with the source's main spine in 1–2 sentences.
- Use numbered sections whose headings are source-specific actions or constraints.
- Put timestamps inside sections or in a final source-notes appendix; do not force readers through raw source maps first.
- For practical/workflow sources, include a **copyable workflow checklist** near the end.
- Include a **do-not-misread** section when the source is easily mistaken for a generic thesis, tool demo, or hype piece.
- End with a compact final conclusion that restates the speaker/author's control model.

Quality test for Mode B:

```text
If the source map and audit blocks were removed, would the article still read as a coherent, useful summary?
Would a reader know why each workflow step exists, not just that it exists?
```

#### Reader-facing article craft rules

For a reader-facing article, optimize the final draft as an article, not as a complete inventory. Preserve source fidelity, but shape the reading path.

Use this article rhythm for practical/workflow sources:

```text
source type + spine + one-sentence summary
→ constraints / failure modes
→ first concrete action
→ external artifact or process
→ feedback / review loop
→ architecture or boundary if central
→ scale/parallelization if central
→ decision model / checklist / do-not-misread / source notes
→ final conclusion
```

Heading rules:

- A good heading names a source-specific constraint, action, artifact, or judgment.
- Avoid generic headings such as “The problem”, “The workflow”, “Key takeaways”, or “Main ideas” when the source contains concrete named steps.
- Prefer headings like “Constraint: smart zone / dumb zone”, “Action: use Grill Me to turn vague requests into shared understanding”, or “Review: use fresh context instead of self-reviewing in the dumb zone”.

When improving an existing reader-facing article:

- Do not mechanically append missing points as new sections.
- First decide whether each missing point should become a new section, merge into an existing section, rename an existing heading, or only appear in source notes.
- Preserve article rhythm over exhaustive section count.
- If new information is important but breaks flow, integrate it through a transition sentence that explains how it serves the source's control model.
- Keep the original article's strongest title, opening spine, and final conclusion unless they are wrong.

Old-good / new-complete merge rule:

```text
If version A reads better and version B covers more facts, do not replace A with B.
Use A as the narrative spine; absorb B's missing facts only where they strengthen that spine.
```

### Step 8 — Write the summary

Writing constraints:

- Preserve the source's main sequence unless there is a strong reason not to.
- Prefer concrete verbs: tried, observed, changed, scanned, generated, tested, reviewed.
- Include timestamps for key claims when useful.
- Do not add facts not in the source unless clearly marked as external context.
- Do not turn examples into generic principles too early.
- Do not let books/frameworks/concepts steal the main role from the speaker's actual actions.
- If translating, preserve tool names, command names, repository names, and technical terms accurately.
- Keep personal implications separate from source summary.

Good phrasing:

```text
The speaker first describes a concrete failure: they tried X, observed Y, and changed Z because constraint C made the original approach unreliable.
```

Risky phrasing:

```text
The core of the talk is that timeless engineering principles matter.
```

The second sentence may be true, but it likely over-abstracts a practical talk.

### Step 9 — Run audits before publishing

#### A. Main-thread audit

Ask:

```text
If a reader only reads this summary, what would they think the source is mainly about?
Does that match the transcript's actual main thread and time allocation?
```

If not, rewrite.

#### B. Missing-practical-details audit

For practical/workflow videos, check whether the summary includes:

- initial failed/default workflow;
- observed symptoms;
- concrete tools/prompts/skills/files;
- implementation steps;
- feedback loops;
- limitations and warnings;
- human review boundaries.

#### C. Concept-overreach audit

Check:

- Did a support concept become the headline or spine?
- Did the summary replace speaker actions with general principles?
- Did we add our own framework without marking it?
- Did implications crowd out source content?

#### E. Long-source coverage audit

For long sources, verify:

- Did the source map cover beginning, middle, and ending?
- Were all major chapters/topics represented, even if compressed?
- Were repeated controlling concepts checked for recurrence?
- Is the sampling/chunking strategy disclosed?
- Would a reader understand the actual scope of the original source?

#### F. Speaker/participant audit

For podcasts, panels, interviews, and meetings, verify:

- Are host questions separated from guest/participant conclusions?
- Are speaker roles and disagreements preserved?
- Is one participant overrepresented simply because their claims are easier to summarize?
- Are uncertain forecasts labeled as forecasts, not facts?

#### D. Decision-model audit

For each major workflow action in the summary, verify:

```text
Do we explain why the speaker does this, not only that they do it?
Which constraint or failure mode makes this action necessary?
Would a reader be able to copy the speaker's judgment, not just the surface step?
```

#### G. Near-perfect revision without quality regression

Use this only after the summary already passes the main-thread and reader-facing audits. The goal is not to inflate the article to chase a perfect score; it is to fix small, source-supported omissions without damaging the reading experience.

Classify every non-perfect score as one of three types:

1. **Fixable omission.** A source-supported detail is missing and can be added in one sentence, one source note, or one compact appendix entry. Fix it.
2. **Source-quality ceiling.** The source lacks timestamps, images, figures, audio cues, or full context. State the limitation; do not invent detail. This may cap the score below perfect.
3. **Reader-experience tradeoff.** Adding every detail would make the article worse. Prefer a compact “details worth noting” paragraph or source note over expanding the main narrative. If even that hurts the article, keep the lower score and explain why.

Before raising a score to perfect, check:

- Did the revision add source-supported information only?
- Did it avoid length bloat, checklist dumping, and duplicate explanation?
- Did it preserve the article's strongest spine and headings?
- Would a knowledgeable reader prefer the revised version, not just the scoring sheet?

If the answer is no, keep the lower score. A clean 28/30 can be better than a bloated 30/30.

Challenge prompts:

- Which speaker-coined or repeatedly used term is absent?
- Which concept explains several later actions but was treated as a minor definition?
- Which failure mode was omitted, causing the recommendation to look arbitrary?
- Which human-vs-agent boundary is unclear?
- If someone followed this summary, where would they misuse the workflow?

## Output Templates

### Short summary

```markdown
Core conclusion: <one sentence faithful to the source>

Source spine:
1. <problem/failure mode>
2. <speaker action/workflow>
3. <result/lesson>

Easy-to-miss details:
- <detail with timestamp>
- <detail>
```

### Reader-facing public article

```markdown
# <Source / speaker>: <how the source turns the problem into a controllable workflow>

来源：<URL / source id>
Transcript：<language, duration, segment count, quality note>

## Source type

This is a <practical/workflow / viewpoint / tutorial / interview / podcast> source.

主轴：<constraint/failure → action/process → feedback/review → limit>.

一句话概括：

> <Specific summary that names the actual workflow/problem; avoid generic lessons.>

## 1. <Opening constraint or failure mode>
## 2. <First concrete action/process introduced by the source>
## 3. <Next artifact/tool/process and why it exists>
## 4. <Feedback/review/control loop>
## 5. <Architecture / boundary / caveat if central>
## 6. <Parallelization / scaling / final workflow if central>

## Decision model map

<Constraint → failure → judgment → action → evidence.>

## 可复制的工作流清单

<Only include steps the source supports.>

## 不要误读这份材料

<List common wrong readings and the more accurate reading.>

## Timestamp source notes

- <timestamp/section>: <source fact>

## 最终结论

<Compact conclusion.>
```

### Audit report

```markdown
## Verdict

- Faithfulness: pass/fail
- Main-thread match: pass/fail
- Missing practical details: <list>
- Concept overreach: <list>
- Decision-model fidelity: <pass/fail>

## Evidence

- 01:12–02:14: <transcript-backed point>
- 06:38–07:15: <transcript-backed point>
```

## Common Pitfalls

1. **Writing a better essay than the source summary.** Good prose is not enough. If the main thread changed, it failed.

2. **Letting support concepts steal the spine.** A speaker may cite famous frameworks to explain practice. Unless the talk is about those frameworks, they are support, not the main story.

3. **Ignoring time allocation.** If the source spends five minutes on workflow failures and thirty seconds on a concept name, reflect that weight.

4. **Compressing examples into slogans.** Keep concrete examples, especially for practical takeaways.

5. **Mixing implications into source summary.** Public readers usually need the source first. Put analysis in a separate section.

6. **Only auditing facts, not task fit.** A summary can have no factual errors and still answer the wrong question.

7. **No reverse-summary check.** Always ask what a reader would think the original was about after reading your summary.

8. **No timestamp trail.** Key claims should be traceable to source ranges, even if the final prose does not cite every line.

9. **Responding to critique with paragraph patches.** Critique from a knowledgeable reader, viewer, or listener often means the summary method missed the source's decision model. Audit the method before patching prose.

10. **Dumping working artifacts before the article.** Source maps and long-source plans are useful, but if they appear before the narrative in a reader-facing article, the output feels like an evaluation log rather than a good summary. Choose delivery mode before drafting.

11. **Mechanical completeness patches.** When a later run discovers missing details, do not bolt them onto the article as isolated new sections. Decide whether they belong as a new section, a renamed heading, a paragraph inside an existing section, a checklist item, or a source note. A complete but patchy article is worse than a slightly shorter article with a strong spine.

12. **Generic titles that flatten the source.** “Fundamentals still matter” may be true, but if the source demonstrates a control workflow, title the article around that workflow.

## Verification Checklist

Before finalizing:

- [ ] Source/transcript fetched or limitation stated.
- [ ] Source-quality limitations checked: missing figures/tables/code/links, subtitle/STT quality, preview vs full source.
- [ ] Source type classified.
- [ ] Source map exists.
- [ ] Delivery mode chosen: working artifact/audit report OR reader-facing article.
- [ ] For reader-facing articles, source map/long-source plan are not placed before the narrative unless explicitly requested.
- [ ] Long-source plan exists when source exceeds ~50k chars / 60 minutes / 20 source-map nodes.
- [ ] Named concepts are listed and role-classified.
- [ ] Main thread is stated in one sentence.
- [ ] Practical/workflow material includes the concrete action/failure chain.
- [ ] Decision-model map exists for major actions.
- [ ] Support concepts are not the main spine unless source type justifies it.
- [ ] Public/private implications are separated.
- [ ] Missing-practical-details audit passed.
- [ ] Decision-model audit passed.
- [ ] Long-source coverage audit passed when applicable.
- [ ] Speaker/participant audit passed when applicable.
- [ ] Reverse summary matches source main thread.
- [ ] Reader-facing article has a source-specific title, one-sentence faithful summary, numbered narrative sections, and a final conclusion.
- [ ] Reader-facing article headings name source-specific constraints/actions/artifacts, not generic categories.
- [ ] If revising from an earlier good article, missing facts were integrated into the narrative spine rather than appended mechanically.
- [ ] If published, the final page or file was fetched back and checked.

---
> Source: [Kinneyzhang/source-faithful-summary-skill](https://github.com/Kinneyzhang/source-faithful-summary-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
