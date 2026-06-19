## deliberate

> Multi-agent deliberation skill. Forces structured disagreement to surface blind spots. Use when making complex decisions, evaluating trade-offs, or when a single confident answer might hide real risks. Invoke with /deliberate or @deliberate followed by your question.


# deliberate -- Multi-Agent Deliberation Protocol

**Agreement is a bug.** This skill forces multiple agents to disagree before they agree, surfacing blind spots that single-perspective answers hide.

## Invocation

```
/deliberate "should we migrate from REST to GraphQL?"
/deliberate --full "is this acquisition worth pursuing at 8x revenue?"
/deliberate --quick "monorepo or polyrepo?"
/deliberate --duo assumption-breaker,pragmatic-builder "should we rewrite the auth layer?"
/deliberate --triad architecture "should we split the monolith now?"
/deliberate --triad decision "build vs buy for the notification system"
/deliberate --members assumption-breaker,first-principles,bias-detector "why does our cache keep failing?"
/deliberate --brainstorm "how should we redesign the onboarding flow?"
/deliberate --profile exploration "what's the right approach to AI safety for our product?"
```

## Flags

| Flag | Effect |
|------|--------|
| (no flag) | Auto-detect domain from question, select matching triad |
| `--full` | Convene all 14 core agents. 3-round protocol. |
| `--quick` | Auto-detect triad. 2-round protocol (skip cross-examination). |
| `--duo agent1,agent2` | Dialectic mode. 2 agents, 2 rounds of exchange, then synthesis. |
| `--triad {domain}` | Use pre-defined triad for domain. 3-round protocol. |
| `--members a,b,c,...` | Custom agent selection (2-14 agents). 3-round protocol. |
| `--brainstorm` | Brainstorm mode. See BRAINSTORM.md for full protocol. |
| `--profile {name}` | Use named profile (full, lean, exploration, execution). |
| `--visual` | Launch visual companion for this session. |
| `--save {slug}` | Override auto-generated filename slug for output. |
| `--research` | Run a grounding phase before Round 1: scan the codebase and/or search the web. Agents reason from retrieved evidence, not parametric knowledge alone. **Opt-in only — never the default.** See `protocols/research-grounding.md`. |
| `--research=web` | Web search only (no codebase scan). |
| `--research=code` | Codebase scan only (no web search). |

## The 14 Core Agents

| # | Agent | Function | Tier |
|---|-------|----------|------|
| 1 | `assumption-breaker` | Destroys hidden premises, tests by contradiction, dialectical questioning | high |
| 2 | `first-principles` | Bottom-up derivation, refuses unexplained complexity | mid |
| 3 | `classifier` | Taxonomic structure, category errors, four-cause analysis | mid |
| 4 | `formal-verifier` | Computational skeleton, mechanization boundaries, abstraction | mid |
| 5 | `bias-detector` | Cognitive bias detection, pre-mortem, de-biasing interventions | high |
| 6 | `systems-thinker` | Feedback loops, leverage points, unintended consequences | mid |
| 7 | `resilience-anchor` | Control vs acceptance, moral clarity, anti-panic grounding | mid |
| 8 | `adversarial-strategist` | Terrain reading, competitive dynamics, strategic timing | mid |
| 9 | `emergence-reader` | Non-action, subtraction, intervention audit, minimum intervention | high |
| 10 | `incentive-mapper` | Power dynamics, actor incentives, principal-agent problems | mid |
| 11 | `pragmatic-builder` | Ship it, maintenance cost, over-engineering detection | mid |
| 12 | `reframer` | Dissolves false problems, frame audit, false dichotomies | high |
| 13 | `risk-analyst` | Antifragility, tail risk, fragility profile, barbell strategy | high |
| 14 | `inverter` | Multi-model reasoning, inversion, opportunity cost, cross-domain | mid |

## Optional Specialists

Activated only when their domain-specific triad is selected:

| Agent | Function | Triads |
|-------|----------|--------|
| `ml-intuition` | Neural net intuition, training dynamics, jagged frontier | ai, ai-product |
| `safety-frontier` | Scaling dynamics, capability-safety frontier, phase transitions | ai |
| `design-lens` | User-centered design, honesty audit, "less but better" | design, ai-product |

## Polarity Pairs

These agents are structural counterweights. When both are present, genuine disagreement is almost guaranteed:

| Pair | Tension |
|------|---------|
| assumption-breaker vs first-principles | Top-down destruction vs bottom-up construction |
| classifier vs emergence-reader | Impose structure vs let it emerge |
| adversarial-strategist vs resilience-anchor | Win externally vs govern internally |
| formal-verifier vs incentive-mapper | Abstract purity vs messy human reality |
| pragmatic-builder vs reframer | Ship it vs does it need to exist? |
| pragmatic-builder vs systems-thinker | Fix the bug vs redesign the system |
| risk-analyst vs ml-intuition | Tail paranoia vs empirical iteration |

## Pre-defined Triads

| Domain | Agents | Reasoning Chain |
|--------|--------|-----------------|
| architecture | classifier + formal-verifier + first-principles | categorize -> formalize -> simplicity-test |
| strategy | adversarial-strategist + incentive-mapper + resilience-anchor | terrain -> incentives -> moral grounding |
| ethics | resilience-anchor + assumption-breaker + emergence-reader | duty -> questioning -> natural order |
| debugging | first-principles + assumption-breaker + formal-verifier | bottom-up -> assumptions -> formal verify |
| innovation | formal-verifier + emergence-reader + classifier | abstraction -> emergence -> classification |
| conflict | assumption-breaker + incentive-mapper + resilience-anchor | expose -> predict -> ground |
| complexity | emergence-reader + classifier + formal-verifier | emergence -> categories -> formalism |
| risk | adversarial-strategist + resilience-anchor + first-principles | threats -> resilience -> empirical verify |
| shipping | pragmatic-builder + adversarial-strategist + first-principles | pragmatism -> timing -> first-principles |
| product | pragmatic-builder + incentive-mapper + reframer | ship it -> incentives -> reframing |
| decision | inverter + bias-detector + risk-analyst | inversion -> biases -> tail risk |
| systems | systems-thinker + emergence-reader + classifier | feedback -> emergence -> structure |
| economics | adversarial-strategist + inverter + incentive-mapper | terrain -> models -> power |
| uncertainty | risk-analyst + adversarial-strategist + assumption-breaker | tails -> threats -> premises |
| bias | bias-detector + reframer + assumption-breaker | biases -> frame -> premises |
| ai | formal-verifier + ml-intuition + safety-frontier | formalism -> empirical ML -> safety |
| ai-product | pragmatic-builder + ml-intuition + design-lens | ship -> ML reality -> user |
| design | design-lens + reframer + pragmatic-builder | user -> frame -> ship |

## Profiles

| Name | Agents | When to Use |
|------|--------|-------------|
| full | All 14 core | Complex decisions with real trade-offs. Claude Code default. |
| lean | assumption-breaker, first-principles, bias-detector, pragmatic-builder, inverter | Fast decisions, limited context. Windsurf default. |
| exploration | assumption-breaker, classifier, emergence-reader, reframer, systems-thinker, inverter, risk-analyst | Discovery, open-ended investigation |
| execution | pragmatic-builder, first-principles, adversarial-strategist, bias-detector, formal-verifier | Shipping decisions, technical trade-offs |

---

## Coordinator Execution Sequence

The coordinator is the orchestration layer. It does NOT have its own opinion. It routes, enforces protocol, and synthesizes.

### Step 0: Research Grounding (optional — `--research` flag only)

If `--research`, `--research=web`, or `--research=code` is set (or natural language equivalents on Windsurf: "do research first", "search the web", "look at the codebase first", "with research"):

1. Read the full protocol at `protocols/research-grounding.md`
2. Execute Steps R1–R4 from that protocol before dispatching any agents
3. Package the grounding context (Codebase Context Summary and/or Web Research Summary) for injection into all agent prompts in Step 5
4. After Round 1, run Step R5 (evidence quality check)

If `--research` is NOT set, skip this step entirely. Do not spontaneously research unless the user explicitly requests it.

---

### Step 0.5: Model Selection (Claude Code only)

**This step runs only on Claude Code.** On Windsurf and Cursor, agents use the active model in the current context — no selection needed.

Before doing anything else, ask the user:

```
⚙️ Model configuration for this deliberation:

High-tier agents (assumption-breaker, bias-detector, emergence-reader,
reframer, risk-analyst, safety-frontier) will use:

  A) Opus + Sonnet  — opus for high-tier, sonnet for mid-tier [DEFAULT]
                      Best quality. Higher token cost.
  B) Sonnet only    — sonnet for all agents
                      Faster. Lower cost. Still strong.

Which would you prefer? (A/B, or press Enter for default A)
```

**Wait for the user's response before proceeding.**

- If the user selects **A** or presses Enter: resolve `high → opus`, `mid → sonnet` from `configs/defaults.yaml`
- If the user selects **B**: resolve both `high` and `mid` → `sonnet`

Store the resolved model map for use in Step 4. This selection applies for the entire session.

If `configs/provider-model-slots.yaml` exists in the project root, skip this prompt entirely and use manual overrides from that file.

---

### Step 1: Platform Detection

Read `configs/defaults.yaml` to determine:
- **Claude Code**: Use parallel subagents. Each agent gets its own context window.
- **Windsurf**: Use sequential role-prompting within single context. Default to "lean" profile unless user overrides.
- **Other**: Fall back to sequential mode.

### Step 2: Problem Restatement

Before any agent speaks, the coordinator restates the user's question in neutral, precise terms. This prevents framing bias from the original question.

```
## Problem Restatement
{Neutral restatement of the user's question, stripped of loaded language}
```

### Step 3: Agent Selection

Based on flags:
- `--full`: All 14 core agents
- `--quick` or no flag: Auto-detect domain from question keywords, select matching triad
- `--triad {domain}`: Select named triad
- `--duo a,b`: Select the two named agents
- `--members a,b,c`: Select named agents
- `--profile {name}`: Select agents from named profile

If auto-detection is ambiguous, present the top 2-3 triad matches and let the user choose.

### Step 4: Model Routing

Use the resolved model map from Step 0.5 (Claude Code) or the active context model (Windsurf/Cursor):

**Claude Code:**
- Agents with `model_tier: high` → pass `model: opus` (or `model: sonnet` if user chose B) to the Agent tool
- Agents with `model_tier: mid` → pass `model: sonnet` to the Agent tool
- `opus` and `sonnet` are Claude Code's accepted shorthands for the current claude-opus-4-6 and claude-sonnet-4-6
- If `configs/provider-model-slots.yaml` exists, use manual overrides instead

**Windsurf / Cursor:**
- All agents run sequentially within the current context window using the model already active in the session
- `model_tier` in agent frontmatter is treated as metadata only — no model switching occurs
- No model selection prompt is shown

### Step 5: Visual Companion Offer

If `--visual` flag is set, skip directly to launching the server (step 5c below).

Otherwise, **always offer the visual companion** before starting the deliberation:

```
Would you like to follow the deliberation in real time in your browser?
The visual companion shows agent positions, agreement/disagreement maps,
and verdict formation as it happens. (Requires opening a local URL)

Open visual companion? (y/N)
```

**This offer MUST be its own message.** Wait for the user's response before continuing.

- If **yes**: proceed to launch (5c)
- If **no** or Enter: proceed without visual companion

**5c — Launch:**
1. Run `scripts/start-server.sh --project-dir {project_root}`
2. Save `screen_dir` and `state_dir` from the response
3. Tell the user to open the URL
4. Visual companion will be updated after each round

### Step 6: Round 1 -- Independent Analysis

Each selected agent receives:
```
## Your Role
You are {agent_name}. Read your full agent definition at agents/{agent_name}.md.

## Problem
{Problem restatement from Step 2}

## Grounding Evidence [only present if --research is active]
{Codebase Context Summary and/or Web Research Summary from Step 0}
Use this evidence in your analysis. You may cite specific findings.
If the evidence is insufficient, note what additional information would change your position.

## Instructions
Produce your independent analysis following your Analytical Method.
400-word maximum. Do NOT reference other agents (you haven't seen their output yet).
Follow your Standalone Output Format.
```

**Claude Code execution**: Launch all agents as parallel subagents. Wait for all to complete.
**Windsurf execution**: Prompt each agent role sequentially. Collect all outputs before proceeding.

### Step 7: Round 2 -- Cross-Examination

Each agent receives ALL Round 1 outputs and must respond:

```
## Your Role
You are {agent_name}. Read your full agent definition at agents/{agent_name}.md.

## Problem
{Problem restatement}

## Round 1 Outputs
{All agent outputs from Round 1}

## Instructions
Follow your Round 2 Output Format:
1. Name which agent you MOST disagree with, and why
2. Name which agent's insight STRENGTHENS your position
3. State what, if anything, changed your view
4. Restate your position

300-word maximum. You MUST engage at least 2 other agents by name.
You MUST disagree with at least one position.
```

**Claude Code execution**: Sequential (each agent needs to see prior cross-examinations for richer engagement).
**Windsurf execution**: Sequential.

### Step 8: Enforcement Scans

After Round 2, the coordinator checks:

1. **Hemlock rule**: Did `assumption-breaker` re-ask a question already answered with evidence? If yes, force 50-word position statement.
2. **3-level depth**: Did any questioning chain exceed 3 levels? If yes, force position commitment.
3. **2-message cutoff**: Did any pair exchange more than 2 messages? If yes, force Round 3.
4. **Dissent quota**: Did at least 30% of agents disagree with something? If not, the coordinator explicitly flags: "Low dissent detected. Consider whether groupthink is present."
5. **Novelty gate**: Did Round 2 introduce at least one idea not present in Round 1? If not, flag stale deliberation.
6. **Groupthink flag**: If ALL agents agree on the core position, flag: "Unanimous agreement may indicate groupthink. Consider invoking `risk-analyst` or `assumption-breaker` standalone for a second opinion."

### Step 9: Round 3 -- Crystallization

Each agent states their FINAL position:

```
## Instructions
State your final position in 100 words or fewer.
No new arguments. No new questions. Crystallization only.
If you changed your mind during cross-examination, state the new position and what changed it.
```

**Execution**: Parallel (Claude Code) or sequential (Windsurf). No interaction needed.

### Step 10: Verdict Synthesis

The coordinator synthesizes all three rounds into a structured verdict:

```markdown
## Deliberation Verdict

### Problem
{Original question}

### Agents Present
{List of agents and their functions}

### Mode
{Full / Quick / Duo / Triad: {domain}}

### Consensus Position
{The position held by 2/3+ of agents, if one exists}

### Key Insights by Agent
{2-3 sentence summary per agent of their most valuable contribution}

### Points of Agreement
{Bullet list of positions where agents converged}

### Points of Disagreement
{Bullet list of positions where agents diverged, with the specific tension}

### Minority Report
{If any agent held a position that the majority rejected, state it here with full reasoning. Sometimes the minority is right.}

### Verdict Type
{consensus | majority | split | dilemma}
- consensus: 2/3+ agree, minority report recorded
- majority: simple majority, significant dissent recorded
- split: no majority, all positions presented equally
- dilemma: the agents surfaced a genuine dilemma with no clear resolution

### Recommended Next Steps
{1-3 concrete actions based on the verdict}

### Unresolved Questions
{Questions raised during deliberation that were not resolved}
```

### Step 11: Save Output

Save the full deliberation record to `deliberations/YYYY-MM-DD-HH-MM-{mode}-{slug}.md`.

If visual companion is active, push the verdict formation view to the browser.

### Step 12: Visual Companion Shutdown (if active)

If the visual companion server was started during this session, ask the user:

```
The visual companion server is still running at http://localhost:{port}.
Stop it now? (Y/n)
```

- If **Y** or Enter: run `scripts/stop-server.sh {session_dir}` where `{session_dir}` is the `session_dir` value returned by `start-server.sh` in Step 5c. This stops the exact session that was started.
- If **N**: leave it running. Remind the user it will auto-shutdown after 30 minutes of inactivity.

If the visual companion was NOT active during this session, skip this step.

---

## Quick Mode Execution

Quick mode skips Round 2 (cross-examination):

1. Steps 0-6 (same as full, including research grounding if `--research` is set)
2. Skip Steps 7-8
3. Step 9: Crystallization (agents state final position based only on seeing Round 1 outputs)
4. Steps 10-11 (same as full)

Quick mode is faster and cheaper but produces less refined disagreement. Use for time-sensitive decisions where the diversity of initial perspectives is more valuable than deep cross-examination.

---

## Duo Mode Execution

Duo mode runs two agents in dialectic:

1. Steps 0-5 (same as full, but only 2 agents, including research grounding if `--research` is set)
2. **Exchange Round 1**: Agent A states position (400 words). Agent B states position (400 words).
3. **Exchange Round 2**: Agent A responds to Agent B (300 words). Agent B responds to Agent A (300 words).
4. **Synthesis**: Coordinator synthesizes the exchange into a verdict highlighting:
   - Where the two agents agree
   - Where they disagree and why
   - What the user should weigh when deciding
5. Step 11 (save output)

Duo mode is ideal for binary decisions ("should we do X or not?"). The polarity pairs table above lists the most productive pairings.

---

## Auto-Routing (no flag)

When the user invokes `/deliberate` without a mode flag:

1. Parse the question for domain keywords
2. Match against triad `duo_keywords` from agent frontmatter
3. Select the best-matching triad
4. If match confidence is low (keywords match multiple triads equally), present top 2-3 options:
   ```
   Your question touches multiple domains. Which triad fits best?
   A) decision (inverter + bias-detector + risk-analyst) -- for evaluating trade-offs
   B) architecture (classifier + formal-verifier + first-principles) -- for structural decisions
   C) strategy (adversarial-strategist + incentive-mapper + resilience-anchor) -- for competitive positioning
   ```
5. Run 3-round protocol with the selected triad

---

## Tie-Breaking and Consensus Rules

- **2/3 majority** = consensus. Dissenting position recorded in Minority Report.
- **Simple majority but less than 2/3** = majority. Significant dissent recorded.
- **No majority** = the dilemma is presented to the user with each position clearly stated. The coordinator does NOT force artificial consensus.
- **Domain expert weighting**: The agent whose function most directly matches the problem domain gets 1.5x weight in determining majority. (E.g., `formal-verifier` gets 1.5x weight on a formalization question.)

---

## When to Use and When Not To

**Use `/deliberate` for:**
- Complex decisions where trade-offs are real
- Architecture choices, strategic pivots, build-vs-buy
- Decisions where you already have an opinion but suspect you're missing something
- Any situation where a single confident answer might hide real trade-offs

**Do NOT use `/deliberate` for:**
- Questions with clear, correct answers
- Do not convene 14 agents to debate tabs vs spaces
- Do not use `--full` when a triad covers the domain (14 agents consume significant context and API cost)

**The sweet spot:** Decisions where a single confident answer hides real trade-offs. `/deliberate` surfaces what you're not seeing, structured, with the disagreements visible.

---

## Brainstorm Mode

For brainstorming, read the full protocol at `BRAINSTORM.md`. Brainstorm mode is a separate process optimized for creative exploration rather than decision-making. It uses the same agents but in a divergent-convergent flow with visual companion support.

---
> Source: [FavioVazquez/deliberate](https://github.com/FavioVazquez/deliberate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
