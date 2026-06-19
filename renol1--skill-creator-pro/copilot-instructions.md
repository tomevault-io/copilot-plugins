## skill-creator-pro

> Create new skills, modify and improve existing skills, and measure skill quality. ALWAYS use this skill when users want to create a skill from scratch, edit or optimize an existing skill, extract a skill from a conversation, test a skill, or optimize a skill's description for better triggering accuracy. Also trigger when user says "turn this into a skill", "make a skill", "skill from this", "improve my skill", "fix my skill's triggering", "help me build a skill", "teach Claude to do X", "how do I make Claude always do X", "can Claude learn this workflow", or similar. Do NOT use for general coding tasks, documentation writing, or template creation without skill context.


# Skill Creator Pro

Create, test, and iteratively improve agent skills. Works in claude.ai, Claude Code, and Cowork. Cross-platform: installs on Cursor, Codex, Gemini CLI, Windsurf, and 10+ tools.

For a working mini-example, see `references/example-skill.md` (35 lines). For a medium-complexity example with lookup tables and references, see `references/example-medium-skill.md` (120 lines). For design principles, see `references/gold-standard.md`.

## Configuration

Adjust these defaults to match your workflow:
- **default_platform**: claude.ai | claude-code | cowork (default: claude.ai)
- **test_rigor**: vibe | standard | rigorous (default: standard)
- **output_language**: en | sv | de | fr (default: en)

## The Core Loop

1. **Understand** — What should the skill do? When should it trigger?
2. **Draft** — Write the SKILL.md file and bundled resources
3. **Test** — Run realistic prompts, compare output to expectations
4. **Evaluate** — Review results with the user, grade quality
5. **Improve** — Revise based on feedback, retest
6. **Optimize** — Tune the description for reliable triggering
7. **Package** — Deliver the `.skill` file
8. **Deploy** — Install cross-platform, track performance

Jump in wherever the user is. If the user says "just vibe with me", skip the formal eval process — set `test_rigor` to `vibe`.

---

## Routing

| User says | Workflow |
|-----------|----------|
| "Make a skill for X" | **Create** → Step 1 |
| "Turn this into a skill" | **Extract** → Step 1b |
| "My skill doesn't trigger" | **Optimize** → Step 6 |
| "Improve my skill" / shows SKILL.md | **Improve** → Step 5 |
| "Test my skill" | **Test** → Step 3 |

### Platform Detection

Detect your environment and adapt:
- **Claude Code / Cowork**: Subagents available → spawn parallel test runs, use eval viewer, run description optimizer.
- **claude.ai**: No subagents → test conversationally, review inline, skip quantitative benchmarks.

---

## Step 1: Understand the Intent

1. **What should this skill enable Claude to do?** (the capability)
2. **When should it trigger?** (phrases, contexts, file types, keywords)
3. **What's the expected output?** (file format, structure, tone)
4. **What should NOT trigger it?** (adjacent tasks sharing keywords)
5. **Are test cases useful?** Verifiable outputs (file transforms, code gen) → yes. Subjective outputs (writing, art) → usually no. Suggest default, let user decide.

Proactively ask about edge cases, dependencies, and what "good" looks like. Check available MCPs for research.

### Edge Case Discovery

Probe systematically based on skill type:

| Skill type | Edge cases to ask about |
|------------|------------------------|
| **Data transform** (schema→code, CSV→report) | Missing fields, nulls, unexpected types, empty input, huge input, encoding |
| **Document generation** (reports, specs) | No data for a section, conflicting inputs, multilingual, very long/short content |
| **Code generation** (clients, tests, configs) | Reserved words, special characters, circular deps, platform differences |
| **Workflow automation** (deploy, build) | Partial failure, missing credentials, rate limits, idempotency |
| **Content/creative** (writing, design) | Ambiguous tone, conflicting instructions, sensitive topics, brand constraints |

Most skills span multiple types — check all matching rows. Ask 2-3 edge case questions before drafting.

### Step 1b: Extract from Conversation

When the user says "turn this into a skill":

1. Review the conversation — tools used, sequence, corrections made
2. Identify what Claude didn't know that the conversation taught it (= the skill's value)
3. Note corrections (= "Common Mistakes" section)
4. Note input/output formats
5. Confirm understanding before drafting

The extraction question: **"If a fresh Claude session got this same request, what would it get wrong?"** That's what the skill needs to teach.

---

## Step 2: Write the SKILL.md File

### Skill Structure

```
skill-name/
├── SKILL.md          (required — main instructions)
├── references/       (optional — templates, docs, loaded on demand)
├── scripts/          (optional — executable code for deterministic tasks)
└── assets/           (optional — fonts, icons, files used in output)
```

### Progressive Disclosure

| Level | What | When loaded | Size target |
|-------|------|-------------|-------------|
| **Metadata** | name + description | Always in context | ~100 words |
| **SKILL.md file body** | Main instructions | When skill triggers | <500 lines |
| **Bundled resources** | References, scripts, assets | On demand | Unlimited |

### Skill Size Guide

| Complexity | Lines | When |
|------------|-------|------|
| Simple (one task) | 30–60 | Single output, few edge cases. See `references/example-skill.md` |
| Medium (multi-step) | 60–150 | Multiple steps, lookup tables, domain rules. See `references/example-medium-skill.md` |
| Complex (with refs) | 150–500 | Multiple variants, large templates |

Over 500 lines → move detail to `references/`. Under 30 lines → skill probably isn't adding value.

### Starter Template

```markdown
---
name: my-skill-name
description: [What it does]. ALWAYS use when [triggers]. Also trigger when [secondary]. Do NOT use for [false positives].
version: 1.0.0
author: [name or handle]
license: MIT
---

# [Skill Name]

[What the user gets. Not how the skill is organized — what they walk away with.
BAD: "Three modes: analyze, explain, optimize."
GOOD: "Paste a LinkedIn post → get told why it flopped and a rewritten version that fixes it."]

## Workflow

- [ ] Step 1: [Name] — [why]
- [ ] Step 2: [Name] — [why]
- [ ] Final: Review output with user before delivering

## Output Format

[Exact skeleton, not just description.]

## Examples

**Example 1: [Scenario]**
Input: [user says]
Output: [skill produces — if you have an output template above, show it FILLED IN here, not just referenced]

**Example 2: [Edge case]**
Input: [tricky input]
Output: [how to handle it]

## Edge Cases

[Missing data, nulls, empty input, huge input, unexpected formats.]

## Common Mistakes

1. **[Mistake]** — [Why it happens] → [What to do instead]
```

**Extended frontmatter** (optional, for published skills):

| Field | Purpose | Example |
|-------|---------|---------|
| `version` | Semver tracking | `1.2.0` |
| `author` | Attribution | `@username` |
| `license` | Distribution terms | `MIT`, `Apache-2.0` |
| `allowed-tools` | Tool restrictions | `Read, Write, Bash(npm:*)` |
| `last_reviewed` | Last verified date | `2026-04-09` |
| `review_interval_days` | Stale threshold | `90` |

Claude ignores unknown frontmatter — these are consumed by tooling (registries, stale-scanners), not by the model.

### Writing the Description

The description is the primary trigger mechanism — the single most important piece.

**Be pushy.** Claude under-triggers. Include: what it does, ALWAYS triggers, casual phrasings, file types, DO NOT triggers. Max 1024 chars, third person, no XML tags.

### Writing the Body — Core Principles

Read `references/gold-standard.md` for the full checklist and anti-patterns.

**1. Only add what Claude doesn't already know.** Structure (exact templates), Process (checklists), Domain (your conventions).

**2. Match freedom to fragility.** High fragility → exact template. Low → general direction. See `references/patterns.md`.

**3. Explain WHY, not just WHAT.** Reasoning generalizes better than rigid rules.

**4. Keep it under 500 lines.** Move detail to `references/`.

**5. Consistent terminology.** "Skill" = the concept. "SKILL.md file" = the file. Pick terms, stick with them.

**6. Always include a review step.** Last workflow step should ask specific review questions about the choices made — not just "does this look good?"

**7. Prioritize checklists.** If your skill has a multi-point checklist, make clear which 2-3 items matter most. Users won't do all 14. "If you fix nothing else, fix the hook and remove the link" beats an unprioritized wall of checkboxes.

### The Reference File Pattern

Extract a real, approved output into a reference file with `{placeholders}`. Model copies-and-adapts instead of creating from scratch. See `references/gold-standard.md`.

**Keep references one level deep.** Never SKILL.md → ref.md → detail.md.

### Bundling Scripts

Pre-made scripts beat generated code. Handle errors with verbose messages. Include install commands. For MCP tools, use fully qualified names: `ServerName:tool_name`.

---

## Step 3: Test the Skill

### Claude Code / Cowork — Parallel Testing

Spawn two subagents per test case in the **same turn** (with-skill + baseline). Save test cases to `evals/evals.json`. See `references/schemas.md` for the schema. Use the grader agent (`agents/grader.md`) for assertions.

### claude.ai — Conversational Testing

1. **Load** the skill's SKILL.md file → **Write** a realistic test prompt → **Run** → **Compare** → **Note gaps** → **Fix and rerun**

### What Makes a Good Test

- Targets the skill's value-add (what Claude gets wrong WITHOUT the skill)
- Uses realistic, messy input
- Includes at least one edge case
- **Tests with both rich AND minimal input** — minimal reveals missing gather-steps
- Has a clear expected output defined before running

Run at least 2–3 tests before showing the skill to anyone.

---

## Step 4: Evaluate Results

### Claude Code / Cowork

1. Grade with grader agent → 2. Aggregate: `python -m scripts.aggregate_benchmark` → 3. Generate eval viewer: `python eval-viewer/generate_review.py` (do this BEFORE evaluating yourself) → 4. Read `feedback.json`

For A/B comparison: `agents/comparator.md`. For root-cause analysis: `agents/analyzer.md`.

### claude.ai

Present results inline. Show prompt + output + ask "How does this look?"

---

## Step 5: Improve the Skill

1. **Generalize**, don't overfit. 2. **Keep it lean.** 3. **Explain the why.** 4. **Bundle repeated code.** 5. **Watch Claude's behavior** — unexpected reading order, missed references, ignored files.

Iterate: improve → retest → feedback → repeat.

---

## Step 6: Optimize the Description

Well-optimized descriptions improve activation from ~20% to ~90%.

Write 16–20 trigger eval queries (half should-trigger, half should-not). Good should-triggers are realistic and casual. Good should-not-triggers are near-misses sharing keywords.

**Claude Code:** `python -m scripts.run_loop --eval-set <path> --skill-path <path> --model <model-id> --max-iterations 5`

**claude.ai:** Test queries mentally, add missing keywords and exclusions.

---

## Step 7: Package and Deliver

**With terminal:** `python -m scripts.package_skill <path>` or `zip -r skill.skill skill-name/`

**Without terminal:** Present files individually for download.

**Updating:** Preserve original name. Copy to writable location first. Package as `skill.skill`, not `skill-v2.skill`.

---

## Step 8: After Deployment

### Cross-Platform Install

```bash
./scripts/install.sh path/to/my-skill  # Auto-detects Claude Code, Cursor, Codex, Gemini
```

Read `references/cross-platform.md` for manual install, legacy format conversion, and platform limitations.

### Track Performance

1. Watch for repeated corrections → gap in the skill
2. Note triggering issues → adjust description
3. Log changes in the Changelog

---

## Available Scripts and Agents

### Scripts (`scripts/`)

| Script | Purpose |
|--------|---------|
| `install.sh` | Auto-detect platforms and install skill to all |
| `package_skill.py` | Validate and package as `.skill` zip |
| `quick_validate.py` | Check structure and frontmatter |
| `run_loop.py` | Automated description optimization loop |
| `run_eval.py` | Run single eval iteration |
| `improve_description.py` | Generate improved description from failures |
| `aggregate_benchmark.py` | Aggregate eval results into benchmark.json |
| `generate_report.py` | Generate human-readable eval report |

### Agents (`agents/`) — Claude Code / Cowork only

| Agent | Purpose |
|-------|---------|
| `grader.md` | Grade assertions against skill output — pass/fail per criterion |
| `comparator.md` | Blind A/B comparison between two skill versions |
| `analyzer.md` | Root-cause analysis — why one version beat another |

### References (`references/`)

| File | When to read |
|------|-------------|
| `gold-standard.md` | Before writing — design principles, quality checklist, 5 anti-patterns |
| `patterns.md` | When stuck — 21 patterns including adaptive output, conversation loops, review output, dimensional framework |
| `example-skill.md` | Starting simple — complete 35-line meeting-notes skill |
| `example-medium-skill.md` | Starting medium — 120-line skill with lookup table and references |
| `schemas.md` | Writing evals — JSON schemas for evals, grading, benchmarks |
| `cross-platform.md` | Distributing — install per platform, format conversion, limitations |

---

## Edge Cases

- **Unknown domain**: If the user wants a skill for a topic you don't know well, research first (web search, MCPs). Don't guess domain conventions.
- **Non-English skills**: The SKILL.md body can be any language. Frontmatter fields (name, description) should stay English for cross-platform compatibility.
- **Skills needing MCP tools**: Use fully qualified names (`ServerName:tool_name`). Note MCP dependency in description.
- **Very large skills**: If approaching 500 lines even after extracting to references, consider splitting into multiple focused skills (see Chunked Skills pattern).

## Common Mistakes

1. **Not eating your own dogfood** — If the skill teaches a practice (changelog, frontmatter, review step), the skill itself should follow it. Users notice hypocrisy immediately.
2. **Structure-first opening** — "Three modes: analyze, explain, optimize" describes how the skill is organized. Nobody cares. Lead with what the user gets: "Paste your post → get a rewritten version." This mistake cascades — if the overview is structural, the whole skill reads like documentation instead of a tool.
3. **Description too narrow** — "Creates meeting notes" misses "clean up my notes", "what were the action items", "summarize this call". Add casual phrasings.
4. **Testing only happy path** — Rich input works, minimal input ("write a postmortem") halluccinates. Always test with both.
5. **Frontmatter without version** — Published skills without `version` can't be tracked by registries or updated safely.
6. **Explaining what Claude already knows** — Don't teach Python syntax in a Python skill. Add YOUR conventions, not general knowledge.
7. **Output template without filled example** — A template with `[placeholders]` shows structure. A filled example shows what the user actually gets. Without one, users don't know if the skill produces three sentences or three paragraphs.
8. **Unprioritized checklist** — 14 checkboxes with equal weight = user does none. Mark the top 2-3: "If you fix nothing else, fix THESE."

---

## Checklist Before Shipping

- [ ] Description: specific, third-person, ALWAYS triggers, casual phrasings, DO NOT triggers
- [ ] SKILL.md file body under 500 lines
- [ ] Detail in reference files (one level deep)
- [ ] Consistent terminology, no time-sensitive info
- [ ] Data/facts live in ONE place — SKILL.md references, doesn't duplicate refs
- [ ] Concrete examples included (at least one edge case)
- [ ] Freedom matches fragility, WHY explained
- [ ] Transformative skills (rewrite, convert, summarize): has "did the original survive?" verification step
- [ ] At least 2–3 realistic tests run (rich AND minimal input)
- [ ] Triggering tested with near-miss queries
- [ ] Frontmatter: version, author, license
- [ ] Changelog started
- [ ] Scripts: error handling, install commands, verbose messages
- [ ] Skill follows its own advice (dogfood check)

---

## Changelog

### v10.0.0 (2026-04-09)
- Added pattern: "Adaptive Output" for input-dependent templates (CSV analyst)
- Added pattern: "Conversation Loop" for multi-turn interactive skills (interview coach)
- Added pattern: "Review/Audit Output" for skills that judge instead of produce (document reviewer)
- Total patterns: 21. Found by: batch test of three skills targeting untested skill types

### v9.0.0 (2026-04-09)
- Added pattern: "Dimensional Framework" for subjective domains (tone, style, design)
- Added pattern: "User Intent Mapping" for ambiguous input phrases
- Added checklist: input-preservation verification for transformative skills
- Found by: tone-adapter skill test — first subjective skill revealed all three gaps

### v8.0.0 (2026-04-09)
- Starter template examples now require filled output templates, not just placeholders
- Added principle #7: "Prioritize checklists" — top 2-3 items must be marked
- Added checklist item: no data duplication between SKILL.md and references
- Added Common Mistakes #7 (unfilled output template) and #8 (unprioritized checklist)
- Found by: linkedin-algorithm skill analysis revealed all three gaps

### v7.0.0 (2026-04-09)
- Fixed starter template: opening must lead with user value, not skill structure
- Added Common Mistake #2: "Structure-first opening"
- Template now shows BAD/GOOD example for overview sentence

### v6.0.0 (2026-04-09)
- Added own frontmatter (version, license, last_reviewed)
- Added Common Mistakes, Edge Cases, Configuration, Changelog sections
- Added casual triggers to description ("help me build a skill", "teach Claude to")
- Added scripts/agents inventory tables
- Added medium-complexity example (references/example-medium-skill.md)
- Fixed terminology: "skill" = concept, "SKILL.md file" = file
- Skill now follows all practices it teaches (dogfood compliance)

### v5.0.0 (2026-04-09)
- Added cross-platform support (install.sh, references/cross-platform.md)
- Added patterns: self-refinement, tunable params, stale detection, safety rails
- Extended frontmatter: version, author, license, allowed-tools, last_reviewed

### v4.0.0 (2026-04-09)
- Added Edge Case Discovery table with hybrid-skill note
- Added minimal input testing guidance
- Added Safety Rails and Review Questions patterns

### v3.0.0 (2026-04-09)
- Moved pitfalls/patterns to references/patterns.md
- Removed duplication between SKILL.md and gold-standard.md
- Added concrete test example (meeting-notes)
- Added packaging fallback without terminal
- Added example-skill.md (35-line complete skill)
- Added Size Guide and Changelog pattern

### v1.0.0 (2026-04-09)
- Initial public version. claude.ai-first, no personal references.

---

## Learned

- Testing with minimal input is as important as rich input — v3 test revealed this
- Edge Case Discovery needs hybrid-skill awareness — v4 test revealed this
- Skills that teach practices must follow them — v6 self-audit revealed 7 violations
- Opening must lead with user value, not skill structure — linkedin skill revealed this
- Output templates without filled examples leave users guessing — linkedin analysis revealed this
- Unprioritized checklists get ignored — linkedin analysis revealed this
- Data in SKILL.md AND refs = duplication that drifts — linkedin analysis revealed this
- Subjective domains need dimensional frameworks, not pass/fail checklists — tone-adapter revealed this
- Transformative skills need input-preservation checks — tone-adapter revealed this
- Ambiguous user input needs intent mapping tables — tone-adapter revealed this
- Fixed output templates don't work for data-dependent skills — csv-analyst revealed this
- Linear workflows don't model interactive coaching — interview-coach revealed this
- Skills that judge need scoring dimensions + finding format, not output templates — document-reviewer revealed this
- Each test finds fewer new gaps (3→3→3) — skill is converging but not done

---

## Principle of Lack of Surprise

Skills must not contain malware, exploit code, or anything compromising system security. A skill's contents should not surprise the user in their intent.

---
> Source: [Renol1/skill-creator-pro](https://github.com/Renol1/skill-creator-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
