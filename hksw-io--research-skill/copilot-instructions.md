## research-skill

> >


# Multi-Agent Research

Spawn a team of investigators to research a topic from multiple facets in parallel. Inspired by [Anthropic's multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system).

## Arguments

Parse `$ARGUMENTS`:
- `size:N` -> team size override (3-7)
- `depth:X` -> focused (1-2 rounds), standard (2-3), comprehensive (4+)
- Remaining text -> research topic

## Flow

```
CONTEXT_GATHERING -> SCOPE_FRAMING -> PLAN_TO_MEMORY -> INVESTIGATOR_SELECTION -> PARALLEL_INVESTIGATION -> [EVALUATE: more needed?] -> CROSS_POLLINATION -> [EVALUATE: more needed?] -> VERIFICATION -> CONSOLIDATION -> REPORT -> MEMORY_SUGGESTION -> CLEANUP
```

The research loop is adaptive — after each round, the lead agent evaluates whether more investigation is needed rather than committing to a fixed number of rounds upfront.

## Context Gathering

Before framing scope or selecting investigators, the lead agent must build understanding of the project and topic.

### Codebase exploration

If the topic is not about code (e.g., market research, policy questions, product strategy), skip codebase exploration. Proceed directly to high-impact questions.

Use Glob, Grep, and Read to understand the project:
- **Platform detection**: Look for .xcodeproj/.xcworkspace (Apple), .csproj/.sln (Microsoft), AndroidManifest.xml (Android), package.json (Web).
- **Tech stack**: Read project config files, dependency manifests, and top-level source directories.
- **Relevant code**: If the topic references specific features or systems, read the relevant source files.
- **Project docs**: Read CLAUDE.md, README, or architecture docs if they exist.

### High-impact questions

After codebase exploration, evaluate whether there is enough context. If not, ask the user via AskUserQuestion:

- **Research scope**: "Are you looking for a go/no-go assessment, a risk map, or a full implementation plan?"
- **Known constraints**: "What do you already know? What have you already ruled out?"
- **Decision timeline**: "Is this informing an imminent decision or building long-term understanding?"
- **Acceptable depth**: "Should we investigate implementation details or stay at the architectural level?"
- **Prior work**: "Has this been investigated before? What was found?"

Pick the 2-4 most impactful questions. Do not ask generic questions the codebase answers. Do not ask more than 4.

### Context summary

Produce a brief internal context summary (not shown to user) capturing: platform, tech stack, relevant files, constraints, what the user wants to learn. This is included in every investigator's prompt.

## Scope Framing

Score the topic on 4 dimensions (1-3 each):

| Dimension | 1 (Narrow) | 2 (Medium) | 3 (Broad) |
|-----------|-----------|------------|-----------|
| Breadth | Single system/component | 2-3 systems | Cross-cutting concern |
| Depth | Surface-level assessment | Implementation detail needed | Deep technical investigation |
| Uncertainty | Well-understood area | Some unknowns | Mostly unknown territory |
| Stakeholder impact | Single team | 2-3 teams | Organization-wide |

Total = sum (range 4-12).

| Score | Scale | Investigators | Tool call budget/agent |
|-------|-------|--------------|----------------------|
| 4-6 | Focused | 3 | 3-10 calls |
| 7-9 | Standard | 3-5 | 10-15 calls |
| 10-12 | Comprehensive | 5-7 | 15+ calls |

The tool call budget is embedded in each investigator's prompt. Per Anthropic: "agents struggle to judge appropriate effort, so we embedded scaling rules in the prompts."

Override team size with `size:N`.

### Save plan to memory

After scope framing and before spawning investigators, the lead agent saves its investigation plan (topic, facet decomposition, investigator assignments) to persist context. This prevents loss if the context window is truncated during long investigations.

## Investigator Selection

Read [references/perspectives.md](references/perspectives.md) for detailed guidance.

Decompose the research topic into **facets** — distinct aspects that can be investigated independently. Then assign one investigator per facet.

### Facet decomposition

- What technical systems are involved? -> one investigator per major system
- What non-technical dimensions matter? -> (compliance, UX, operations, cost)
- What is the riskiest unknown? -> assign a dedicated investigator

### Structural roles (always present)

1. **Integrator** — maps cross-facet dependencies, identifies cascading effects, produces draft consolidation, and performs the verification step. Domain-labeled (e.g., "Data Pipeline Engineer" who sees how encoding affects every stage).
2. **Scope Sentinel** — monitors whether the investigation is covering the right territory, flags blind spots, asks "what are we not looking at?" Domain-labeled (e.g., "QA Infrastructure Lead" who checks test coverage).

### Facet investigators (1-5 additional)

Each investigator gets clear delegation boundaries:
- **Objective**: What to investigate (concrete, bounded)
- **Output format**: Findings tagged [grounded]/[informed]/[speculative] with confidence levels
- **Tool guidance**: Which tools to prefer, search strategy hints
- **Task boundaries**: What is IN scope and what is explicitly OUT of scope (prevents overlap)

### Codex assignment

Codex-powered investigators scale with team size. Research benefits more from model diversity than debate — different models genuinely find different facts and access different knowledge.

| Team size | Codex investigators | Assignment strategy |
|-----------|-------------------|-------------------|
| 3 | 1 | Highest-uncertainty facet |
| 4-5 | 2 | Two highest-uncertainty facets |
| 6-7 | 2-3 | Up to 3, never more than half the team |

Assign Codex to facets where cross-model disagreement is most valuable: high uncertainty, factual questions, or code analysis where different models may find different patterns. Do not assign Codex to the Integrator or Scope Sentinel — they need full context awareness that the stateless relay pattern cannot provide.

No fictional names. No fictional backgrounds. Label by role only.

## Agent Questions (structured format)

Investigators communicate questions by including a `# Questions for User` section at the end of their output. Every agent MUST include this section, even when they have no questions.

**Format when agent has questions:**
```
# Questions for User

1. [QUESTION] Can the database schema be modified, or is it shared with other services?
   [WHY] My investigation of the storage layer depends on whether we can add encoding columns.
   [ASSUME_IF_NO_ANSWER] Schema is shared and cannot be modified.

2. [QUESTION] What percentage of current filenames contain non-ASCII characters?
   [WHY] This determines whether the impact is theoretical or already causing failures.
   [ASSUME_IF_NO_ANSWER] Less than 1% based on the English-language test fixtures I found.
```

**Format when agent has no questions:**
```
# Questions for User

NONE
```

## Codex Consultation

During Round 2+, any Claude investigator can request an ad-hoc Codex consultation to cross-check a specific finding. This is separate from the Codex-powered investigators on the team.

- Max 2 ad-hoc consultations per investigation.
- The requesting agent must state what finding it wants to verify and why.
- Invoke via the Skill tool: `skill: "codex", args: "reasoning:EFFORT PROMPT"`
- Set reasoning effort based on the scope score (see table below).
- Results are shared with all investigators.

## Team Execution

Every investigation spawns a team. All investigators — including the Codex-powered one — are first-class team members.

### Step 1: Create team

Use TeamCreate with name `research-{topic-slug}`.

### Step 2: Create tasks

Use TaskCreate to create round tasks:
- `round-1-investigation`: each investigator explores their assigned facet
- `round-2-cross-pollination`: each investigator reacts to all findings
- Additional tasks as needed based on the research loop

### Step 3: Spawn investigator agents

Spawn one Agent per investigator role:
- `team_name`: the research team name
- `subagent_type`: `"general-purpose"` (needs Bash for Codex, Read/Grep for codebase)
- `name`: slug of the role (e.g., `storage-engineer`, `filesystem-specialist`, `scope-sentinel`)

**Claude investigator prompt template:**

> You are the [ROLE] investigating: [TOPIC]
>
> Your facet: [FACET DESCRIPTION]
>
> Project context: [CONTEXT SUMMARY — platform, tech stack, relevant files, key constraints, what the user is trying to learn]
>
> Objective: [CONCRETE OBJECTIVE — what to investigate]
> Task boundaries: [IN SCOPE: ... / OUT OF SCOPE: ...]
> Tool call budget: [N calls for this investigation scale]
>
> Round [N] instructions: [INVESTIGATE YOUR FACET / CROSS-POLLINATE WITH OTHER FINDINGS]
>
> [For Round 2+: Full cumulative investigation history follows — all prior rounds, not just the last one]
> [Round 1 findings: all investigators' reports]
> [Round 1 cross-pollination brief: connections, uncertainty hotspots, blind spots]
> [User clarifications: any answers from the user between rounds]
> [For Round 3+: Summarize Round 1 findings to 2-3 sentences each. Include Round 2+ in full.]
> [Key gaps to address this round: ...]
>
> Quality rules:
> 1. **Ground and tag.** Search the codebase for relevant code, configs, tests, or docs using Read and Grep. Cite specific files and lines. Tag each finding: [grounded] if verified in code/docs, [informed] if based on domain knowledge, [speculative] if uncertain — state what would verify it. Report absence of evidence as a finding.
> 2. **Start wide, then narrow.** Begin with broad queries to survey the landscape. Evaluate what's available. Then progressively narrow focus based on what you find. Do not start with hyper-specific searches.
> 3. **Separate findings from implications.** State what you observed (finding), then what it means for the research question (implication). Do not conflate the two.
> 4. **Report unknowns explicitly.** For each aspect of your facet, classify as: KNOWN (verified), PARTIALLY KNOWN (some evidence, gaps remain), or UNKNOWN (no evidence found, needs investigation). An investigation that reports "everything is fine" without evidence is a failed investigation.
> 5. **Ask if unsure.** If you lack context for a thorough investigation, you MUST add a `# Questions for User` section using the structured format (see Agent Questions section). State what you need and what you will assume if not answered. If you have no questions, end with `# Questions for User` followed by `NONE`. **Do NOT call AskUserQuestion yourself — only the lead agent does. Calling it from a teammate will deadlock.**
>
> Source quality: Prefer authoritative sources (official docs, RFCs, primary sources, academic papers) over SEO-optimized content. Flag when you can only find low-quality sources for a finding. For software engineering topics, prefer current APIs and modern approaches over deprecated or legacy ones — search results often surface outdated patterns that are no longer recommended.
>
> Be direct and analytical. No conversational filler.
> Label your output with your role name.
> Check TaskList for your assigned task and mark it complete when done.

**Codex reasoning effort table:**

| Scope Score | Reasoning Effort |
|-------------|-----------------|
| 4-6 (focused) | medium |
| 7-9 (standard) | high |
| 10-12 (comprehensive) | xhigh |

**Codex investigator prompt template:**

Same role/topic/round information and quality rules as any other agent, plus:

> You do not reason about this topic yourself. Instead, for each round:
> 1. Take your facet assignment, the research question, the quality rules, and the full investigation history
> 2. Construct a single prompt that includes: (a) your role and facet, (b) the topic, (c) the 5 quality rules, (d) the full investigation history from prior rounds, and (e) the specific gaps to address this round. One comprehensive prompt per round.
> 3. Invoke via the Skill tool: `skill: "codex", args: "reasoning:EFFORT PROMPT"`
>    where EFFORT is set by the lead agent based on the scope score (see table above).
> 4. Return the Codex output as your investigation findings
> 5. Your output must end with a `# Questions for User` section. Parse Codex's output for any questions and include them in the structured format. If Codex asked none, output `# Questions for User` followed by `NONE`.
>
> You are a relay between the investigation and Codex CLI. Participate in the task list and messaging like any other agent.

### Step 4: Execute rounds

**Round 1 (Independent Investigation):** All investigators explore their assigned facet in parallel. Each operates in an isolated context window, returning findings to the lead agent. After collecting outputs, follow the **mandatory questions check**.

**Research loop evaluation (after each round):** The lead agent evaluates:
- Are there blocking unknowns that need deeper investigation?
- Did investigators discover new facets not in the original plan?
- Is the evidence quality sufficient for the user's needs?

If more investigation is needed: spawn additional targeted tasks (not full re-runs). If sufficient: proceed to the next phase.

**Round 2 (Cross-Pollination):** Lead agent sends each investigator the **full cumulative investigation history** via SendMessage — all prior findings, the cross-pollination brief, all user clarifications — plus specific gaps to address. Investigators are directed to:
- Review all findings — how do other investigators' discoveries affect their facet?
- Validate or challenge connection points identified by the Integrator
- Address blind spots flagged by the Scope Sentinel
- Refine findings and upgrade/downgrade confidence levels

The Integrator gets special instruction: "Produce a dependency map: which findings depend on which others? Where does a change in one facet cascade?"

The Scope Sentinel gets special instruction: "Based on all Round 1 findings, what facets are still underexplored? What questions has nobody asked? What assumptions are all investigators sharing without examination?"

All agents work in parallel. After collecting outputs, follow the **mandatory questions check**.

**Round 3+ (Deep-Dive):** Only if the research loop evaluation identifies high-uncertainty areas. The lead agent assigns specific deep-dive tasks to individual investigators rather than broad re-investigation. End when the uncertainty map stabilizes or remaining unknowns require action outside the codebase.

**Agent failure:** If an agent fails to respond or produces an error, note the gap and proceed with remaining investigators. Do not retry more than once.

### Mandatory questions check (after EVERY round)

**This step is NOT optional.** After collecting all agent outputs for a round, the lead agent MUST execute this protocol:

1. **Scan every agent output** for the `# Questions for User` section.
   - If any agent's output lacks this section entirely, send that agent a follow-up message: "Your output is missing the `# Questions for User` section. Reply with your questions or `NONE`." Wait for the response before proceeding.
2. **Collect all non-NONE questions** into a single numbered list with attribution (which agent asked).
3. **Apply materiality filter.** For each question, apply the counterfactual test: "If the user gave the opposite answer, would it change the investigation's conclusions?" Drop questions that fail this test.
4. **If material questions remain:**
   a. Batch into AskUserQuestion calls (up to 4 questions per call, multiple calls if needed). **MUST use AskUserQuestion tool — do not ask questions via text output.**
   b. **STOP and WAIT for the user's answer.** Do not proceed to the next round.
   c. After receiving answers, propagate to all agents via SendMessage before continuing.
   d. If the user declines to answer a specific question, use the agent's [ASSUME_IF_NO_ANSWER] and propagate that assumption.
5. **If no material questions remain:** Proceed to the next phase.

**Cross-pollination brief (after Round 1):**

After collecting Round 1 outputs, the lead agent produces an internal cross-pollination brief:
- **Findings map**: What each investigator discovered, organized by facet
- **Connection points**: Where one investigator's findings affect another's facet
- **Uncertainty hotspots**: Facets where evidence is thin or speculative
- **Blind spots flagged**: Anything the Scope Sentinel identified as uncovered

### Step 5: Verification

After the final investigation round and before consolidation, the lead agent asks the Integrator to perform a verification pass:

1. Check every [grounded] finding — does the cited source actually support the claim?
2. Check every [informed] finding — is the domain knowledge current and applicable?
3. Flag any [speculative] findings that could be resolved with one more search
4. Downgrade findings where evidence doesn't hold up
5. Upgrade findings where additional evidence was found during cross-pollination

The lead agent reviews the Integrator's verification before producing the final report.

### Step 6: Consolidation

The lead agent asks the Integrator to produce a draft consolidation:

> "Produce a draft consolidation of all investigation findings. Structure it as:
> 1. A findings summary organized by facet
> 2. A cross-facet dependency map (what affects what)
> 3. A risk matrix (likelihood x impact for identified risks)
> 4. An unknowns map (what remains unknown and how to resolve each)
> 5. Your assessment of overall investigation quality (how much is grounded vs speculative)"

The lead agent refines the Integrator's draft into the final report.

### Step 7: Save key findings to memory

After delivering the final report, save the most important findings to memory so the same research doesn't need to be repeated. This step is automatic — do not ask the user for approval.

**What to save:**
- Non-obvious findings that would be expensive to re-derive (e.g., platform constraints, irreversible decisions, verified gotchas)
- Findings likely to inform future work in this project
- Actionable constraints or recommendations that affect ongoing decisions

**What NOT to save:**
- Ephemeral or one-off facts (e.g., "current test coverage is 73%")
- Anything easily re-derived by reading code or running a command
- Anything already documented in CLAUDE.md or project docs

**How to save:** Write a single memory file covering the research topic's key conclusions. Use the `project` type for technical constraints and decisions, or `reference` for external resource pointers. Keep it concise — capture the conclusions and rationale, not the full investigation. Update MEMORY.md index accordingly.

### Step 8: Cleanup

Send shutdown messages to all agents via SendMessage, then TeamDelete.

## Consolidation Rules

Read [references/investigation-patterns.md](references/investigation-patterns.md) for detailed patterns.

Confidence levels for findings:
- **HIGH**: Multiple grounded sources confirm. No contradicting evidence.
- **MEDIUM**: Some grounded evidence, some informed. Minor gaps.
- **LOW**: Mostly informed or speculative. Significant gaps remain.

Weight findings by evidence quality and investigator domain relevance. A storage engineer's findings about database encoding carry more weight than a UX researcher's speculation about the same topic.

Never produce: "No issues found" without demonstrating thorough investigation. The absence of evidence is not evidence of absence — report what was checked and what was not.

## Final Report

Plain text with markdown headers where helpful. No box-drawing characters, no emoji, no decorative formatting.

**Core sections (always present):**

### Summary
1-3 sentences answering the research question at the executive level. Note which investigators participated and which were Codex-powered.

### Key Findings
Numbered findings, each with:
- Finding statement
- Evidence tag ([grounded], [informed], [speculative])
- Confidence level (HIGH/MEDIUM/LOW)
- Supporting evidence (file paths, URLs, code snippets, documentation references)

### Risk Matrix
Table: Risk | Likelihood (HIGH/MEDIUM/LOW) | Impact (HIGH/MEDIUM/LOW) | Mitigation | Evidence quality

### Unknowns Map
Table: Unknown | Why it matters | How to resolve | Priority

Priority levels:
- **BLOCKING**: Must be resolved before proceeding
- **IMPORTANT**: Should be resolved but work can begin in parallel
- **NICE-TO-KNOW**: Can be deferred

### Recommendations
Concrete next steps grouped by:
- **Immediate**: things that can be done now with high confidence
- **Requires further investigation**: things that depend on resolving unknowns
- **Long-term considerations**: strategic implications

**Conditional sections (include only when substantive):**

### Cross-Facet Dependencies
Which findings interact. "If X is true, then Y is affected because Z." Include only when the investigation revealed non-obvious dependencies between facets.

### Cross-Model Divergence
Include if Codex-powered investigators' findings diverged meaningfully from Claude investigators on the same or related facets. What does cross-model disagreement reveal about certainty? Omit if they agreed.

### Surprise Finding
Include if the investigation surfaced something genuinely non-obvious. Omit if nothing surprising — do not manufacture one.

---
> Source: [hksw-io/research-skill](https://github.com/hksw-io/research-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
