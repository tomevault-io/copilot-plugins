## claude-debate-skill

> Multi-agent adversarial debate for product research. 5 agents research independently, debate with elimination rounds, produce structured recommendations.


# Multi-Agent Adversarial Debate System

You are orchestrating a 6-phase adversarial debate between 5 specialized research agents. Each agent has a unique persona, full web access, and the ability to search, analyze, and argue for product recommendations.

**Query**: $ARGUMENTS

## Architecture

⚠️ **MANDATORY RULE — SEPARATE TASKS PER AGENT**:
Every time this skill says "spawn agents", you MUST create **one Task() call per agent**.
NEVER write responses for multiple agents inside a single Task. Each agent is an independent
LLM instance with its own persona. One LLM roleplaying 5 agents defeats the entire system.
This applies to ALL phases: research, debate rounds, elimination votes, finals, and jury.

- **Communication**: Pure prompt relay via files — no direct agent-to-agent talk (from Mysti)
- **Topology**: Dense — all agents see all responses each round (from debate-or-vote)
- **State management**: Python scripts in `${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/` handle convergence detection, vote tallying, domain detection, and session state
- **Context passing**: `/tmp/debate-session/` symlink (always points to latest session). Each session gets a timestamped directory under `/tmp/debate-sessions/` (e.g. `debate-20260304-143022/`). Previous sessions are preserved.

### Mandatory Steps Checklist (DO NOT SKIP ANY)

These steps are marked with ⛔ throughout the document. If you skip any, the debate is invalid:

| Step | When | Script/Action |
|------|------|---------------|
| `init` | Before anything | `debate_orchestrator.py init "$ARGUMENTS"` — creates dated session folder |
| URL verification | After Phase 1 | WebFetch each product URL — exclude unverified products |
| `check-duplicates` | After Phase 2 | `debate_orchestrator.py check-duplicates` — verify distinct picks |
| `assess-convergence` | After each debate round | `debate_orchestrator.py assess-convergence {R}` — follow exit code |
| `vote_tallier.py` | After each elimination round | `vote_tallier.py /tmp/debate-session/phase4/` — official tally |

## Scripts Reference

| Script | Purpose | Key Functions |
|--------|---------|---------------|
| `scripts/debate_orchestrator.py` | Session state, domain detection, persona selection, context formatting | `init`, `detect-domain`, `select-personas`, `format-debate-context`, `format-judge-input`, `compile-synthesis` |
| `scripts/convergence_detector.py` | Convergence assessment (from Mysti _assessConvergence) | `assess_convergence()` — keyword agreement + Jaccard stability + Delphi facilitator override |
| `scripts/vote_tallier.py` | Vote collection + 4-step tie-break elimination (from elimination_game) | `run_elimination()` — plurality → confidence → cumulative → random |

## Setup

⛔ **BLOCKING — DO NOT SKIP**: You MUST run init before ANY other action. Do NOT `mkdir` manually.

```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py init "$ARGUMENTS"
```

This creates a timestamped session directory (e.g. `/tmp/debate-sessions/debate-20260304-143022/`) with phase subdirectories and `state.json`. A symlink at `/tmp/debate-session` always points to the latest session. Previous sessions are preserved — never overwritten.

**If you skip this step, sessions will overwrite each other and all previous debate data is lost.**

Then read prompt templates:
```bash
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/researcher.md
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/debater.md
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/judge.md
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/synthesizer.md
```

---

## Phase 1 — Research Sprint (Parallel)

**Goal**: 5 agents research independently, each producing 3-5 product picks with evidence.

### Step 1: Detect domain and select personas
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py detect-domain "$ARGUMENTS"
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py select-personas <detected_domain>
```

### Step 2: Spawn 5 research agents in parallel

⚠️ **CRITICAL**: You MUST make **5 separate Task() calls** in a single message — one per agent.
Each Task is a separate LLM instance. NEVER combine multiple agents into one Task.

```
Task(description="Agent 1 research", subagent_type="general-purpose", model="opus",
     prompt="You are {persona_1}... Write to /tmp/debate-session/phase1/agent-1.md ...")
Task(description="Agent 2 research", subagent_type="general-purpose", model="opus",
     prompt="You are {persona_2}... Write to /tmp/debate-session/phase1/agent-2.md ...")
Task(description="Agent 3 research", ...)
Task(description="Agent 4 research", ...)
Task(description="Agent 5 research", ...)
```

Each agent gets:
- Their unique persona from personas.md injected into the researcher.md template
- Full tool access: WebSearch, WebFetch, perplexity_search_web, scrapling-fetch MCP (s_fetch_page, s_fetch_pattern)
- Instruction: "Find at least 2 independent sources per product. Use scrapling-fetch for pages that block basic fetching."

Each agent gets full tool access. The researcher prompt from `prompts/researcher.md` includes:
- Persona-specific system prompt (domain expert framing)
- Multi-query research protocol (direct, expert, comparison, reddit, problem-specific searches)
- Structured output format (product name, price, pros, cons, specs, sources)
- Evidence requirements (2+ independent sources, recent, specific data not vague praise)

### Step 3: Summarize and validate products

After all 5 complete:
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py summarize-research
```

### Step 4: Product URL validation

⛔ **BLOCKING — DO NOT SKIP**: Every product must be verified before entering Phase 2. Without this, agents debate phantom products with dead links, and the final "Buy here" URL is useless.

**Every product that enters the debate must be a real, existing product with a verified URL.** This prevents agents from hallucinating products or defending phantom items throughout the debate.

```
For each unique product found across all 5 agents' research:
  1. Extract the Product URL from the agent's output
  2. Use WebFetch(url=product_url, prompt="Does this page show a real product for sale? Return the exact product name shown on the page, or 'NOT FOUND' if the page is a 404, search results, category listing, or doesn't match the claimed product.")
  3. If NOT FOUND or wrong product:
     - Use WebSearch(query="buy [exact product name] site:amazon.com OR site:[manufacturer].com") to find the real URL
     - If still not found → flag this product as UNVERIFIED
  4. Record verified URL in state
```

```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py update-state '{"verified_products": {"Product A": "https://verified-url...", "Product B": "https://verified-url..."}, "unverified_products": ["Product C"]}'
```

**Rules:**
- Agents in Phase 2+ can ONLY pick from verified products
- Unverified products are excluded from the debate — tell agents they don't exist
- If a product has no working URL anywhere on the internet, it's not a real product

### Step 5: Gate

Present summary to user via **AskUserQuestion**:
> "5 agents researched independently. Here are the verified products: [list with URLs]. Excluded (unverified): [list if any]. Should I proceed to the debate phase, or adjust the research scope?"

---

## Phase 2 — Opening Statements (Sequential)

**Goal**: Each agent declares their single top pick. **Forced disagreement** (from MAD): no two agents can champion the same product.

### How forced disagreement works

From MAD's `negative_prompt` (config4all.json):
```
"##aff_ans##\n\nYou disagree with my answer. Provide your answer and reasons."
```

Adapted for product research: each agent sees all previous agents' picks and MUST choose differently. The opening statement prompt in `prompts/debater.md` enforces this.

### Execution

Spawn agents **sequentially** (each must see previous picks):

For agent N (1 through 5):
```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: <debater.md opening statement template> +
          "Previous agents' picks: [list]" +
          "Your research: [agent-N phase1 results]" +
          "Other agents' research summaries: [brief summaries]" +
          "You MUST pick a different product than: [previous picks]" +
          "IMPORTANT: You may ONLY pick from these verified products: [verified_products from state — product names with URLs]. Do NOT invent, hallucinate, or pick any product not on this list." +
          "Write to /tmp/debate-session/phase2/agent-{N}-opening.md"
)
```

### Verify distinctness

⛔ **BLOCKING — DO NOT SKIP**: Run duplicate check before proceeding to Phase 3.

```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py check-duplicates
```

If duplicates found, re-run the duplicate agent with explicit forced-disagreement instruction. Do NOT proceed to Phase 3 with duplicate picks — it breaks the adversarial structure.

### Save state after Phase 2
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py update-state '{"phase": "phase2-complete", "picks": {"1": "Product A", "2": "Product B", ...}}'
```
(Replace with actual agent picks extracted from opening statements.)

---

## Phase 3 — Debate Rounds (2-3 rounds)

**Goal**: Agents critique each other, update positions. Dense topology with agreement intensity level 9.

### Dense topology context format

Adapted from debate-or-vote's `get_new_message()` (analysis.md:326-343):
```
These are the recent positions from other agents:

One agent's position:
{agent_1_response}

One agent's position:
{agent_2_response}

[... all other agents ...]

This was your most recent position:
{this_agent_response}

You incorporate other agents' evidence 90% of the time and have almost no reservations.
Using their positions as additional perspective, provide your updated position on:
{query}
```

The orchestrator builds this automatically:
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py format-debate-context <round> <agent_id>
```

### Per-round execution

For each round R (1 to 3):

1. Create round directory:
   ```bash
   mkdir -p /tmp/debate-session/phase3/round-{R}
   ```

2. Spawn **5 SEPARATE Task calls** in parallel — one agent per Task:
   ```
   ⚠️ CRITICAL: You MUST make 5 independent Task() calls, NOT one Task that writes all 5.
   Each Task is a separate LLM instance with its own persona. One LLM writing
   all 5 responses defeats the entire purpose of multi-agent adversarial debate.

   Task(description="Agent 1 Round {R}", subagent_type="general-purpose", model="opus",
        prompt="You are {persona_1}... Write to /tmp/debate-session/phase3/round-{R}/agent-1.md")
   Task(description="Agent 2 Round {R}", subagent_type="general-purpose", model="opus",
        prompt="You are {persona_2}... Write to /tmp/debate-session/phase3/round-{R}/agent-2.md")
   Task(description="Agent 3 Round {R}", ...)
   Task(description="Agent 4 Round {R}", ...)
   Task(description="Agent 5 Round {R}", ...)
   ```

   Each agent gets:
   - Their unique persona from personas.md
   - The debater.md debate round template
   - Formatted context from orchestrator (`format-debate-context {R} {agent_id}`)
   - "Agreement intensity level 9: incorporate strong evidence 90% of the time"
   - "Maximum {word_limit} words"

   **Decreasing word limits** (from elimination_game): Round 1 = 500, Round 2 = 400, Round 3 = 300

3. **Assess convergence** — ⛔ **MANDATORY after every round. Follow its recommendation.**

   ```bash
   python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py assess-convergence {R}
   ```

   Exit codes:
   - `0` = **CONVERGED** → skip remaining rounds, move to Phase 4
   - `1` = **CONTINUE** → proceed to next round
   - `2` = **STALLED** → skip remaining rounds, move to Phase 4

   **You MUST follow the exit code.** Do NOT override with manual judgment ("it looks converged to me"). The script uses quantitative metrics (agreement ratio, Jaccard stability, facilitator override). If you disagree with the result, run the Delphi facilitator (step 4 below) to produce a facilitator score override — do NOT skip the script.

   The script uses:
   - 10 agreement patterns: `agree, concede, valid point, correct, accept, well-taken, convinced, you're right, strong evidence, well supported`
   - 10 disagreement patterns: `disagree, however, incorrect, wrong, reject, maintain, defend, overlooked, flawed, insufficient`
   - Jaccard-like text similarity for position stability (words > 3 chars, max-normalized)
   - Formula: `overallConvergence = (agreementRatio * 0.6) + (avgStability * 0.4)`
   - **Delphi facilitator override** (from Mysti BrainstormManager.ts:735-741): if a `facilitator-summary.md` file exists in the round directory with `Convergence Score: N/10`, it overrides the heuristic score

4. **(Optional) Delphi facilitator summary** — for more nuanced convergence after round 2+:
   ```
   Task(
     subagent_type: "general-purpose",
     model: "opus",
     prompt: "You are a Delphi facilitator. Review all agent positions from round {R}:" +
             "[all agent responses]" +
             "Produce: Consensus Points (Strong/Moderate/Tentative), Divergence Points (Position A/B + tension), Open Questions, and a Convergence Score: ?/10" +
             "(0=total disagreement, 5=split, 7=emerging consensus, 10=full agreement)" +
             "Write to /tmp/debate-session/phase3/round-{R}/facilitator-summary.md"
   )
   ```
   Then re-run `assess-convergence` — the facilitator score will override the heuristic.

5. **(Optional) Judge-based early stopping** (from debatellm Tsinghua MAD):

   After each debate round, optionally spawn a judge to check if a clear preference exists:
   ```
   Task(
     subagent_type: "general-purpose",
     model: "opus",
     prompt: "You are a debate judge. Review all agent positions from round {R} on: {query}" +
             "[all agent responses]" +
             "Is there a clear preference emerging? Output JSON:" +
             '{"has_preference": "Yes/No", "preferred_product": "", "confidence": "LOW/MEDIUM/HIGH"}' +
             "Only answer 'Yes' if one product is clearly stronger across multiple agents." +
             "Write to /tmp/debate-session/phase3/round-{R}/judge-early-stop.json"
   )
   ```
   If `has_preference == "Yes"` AND `confidence == "HIGH"`, skip remaining rounds.

6. **(Optional) Private deliberation channel** (from elimination_game dual channel):

   Before each public debate round, paired agents can exchange private critiques:
   ```bash
   python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py format-private-pairs {R}
   ```
   This outputs agent pairings (round-robin). For each pair:
   ```
   Task(
     subagent_type: "general-purpose",
     model: "haiku",
     prompt: "You are {{PERSONA_A}}. You and {{PERSONA_B}} are having a private exchange before the next public round." +
             "{{PERSONA_B}}'s current position: [their last response]" +
             "Your current position: [your last response]" +
             "In 100 words or less, share a private critique or observation with {{PERSONA_B}}. " +
             "Focus on evidence gaps, assumptions, or points you'd challenge in the public round." +
             "Write to /tmp/debate-session/phase3/round-{R}/private/agent-{A}-to-agent-{B}.md"
   )
   ```
   Private notes are automatically included in the agent's context via `format-debate-context`.

---

## Phase 4 — Elimination

**Goal**: Narrow from 5 to exactly **2 finalists**. Vote on **weakest** (from elimination_game), not strongest.

### Voting

⚠️ **CRITICAL**: Spawn **5 separate Task() calls** in parallel — one vote per agent.
NEVER combine multiple agents into one Task. Each vote must come from an independent LLM.

```
Task(description="Agent 1 elimination vote", subagent_type="general-purpose", model="opus",
     prompt="You are {persona_1}... Vote for WEAKEST... Write to /tmp/debate-session/phase4/vote-agent-1.md")
Task(description="Agent 2 elimination vote", subagent_type="general-purpose", model="opus",
     prompt="You are {persona_2}... Vote for WEAKEST... Write to /tmp/debate-session/phase4/vote-agent-2.md")
Task(description="Agent 3 elimination vote", ...)
Task(description="Agent 4 elimination vote", ...)
Task(description="Agent 5 elimination vote", ...)
```

Each agent gets:
- The debater.md elimination vote template with their persona
- All final positions from the last debate round
- "Vote for the WEAKEST recommendation to eliminate"
- "You CANNOT vote to eliminate your own pick"
- "Format: **ELIMINATE**: [name]\n**Reasoning**: [text]\n**Confidence**: [LOW/MEDIUM/HIGH]"

### Tallying with 4-step tie-break chain

⛔ **BLOCKING — DO NOT SKIP**: Run vote_tallier.py after EVERY elimination round. Do NOT manually count votes or declare eliminations without this script.

```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/vote_tallier.py /tmp/debate-session/phase4/
```

The script implements the full chain from elimination_game (analysis.md:811-884):
1. **Plurality** — most votes eliminated
2. **Confidence-weighted re-vote** — HIGH=1.5x, MEDIUM=1.0x, LOW=0.5x
3. **Cumulative vote check** — product with more historical votes-against eliminated
4. **Jury tie-breaker** — if still tied, script returns `method: "jury_tiebreak_required"` (exit code 2)

**The script's output is the official result.** Do NOT interpret vote files yourself — the script handles fuzzy matching, confidence weighting, and tie-breaks that manual reading cannot replicate.

### Handling jury tie-breaker (exit code 2)

If `vote_tallier.py` exits with code 2, a tie persists after all algorithmic steps. **Do NOT eliminate randomly.** Instead:

1. Read the `tied_products` from the JSON output
2. Spawn champion agents for each tied product **in parallel** — each writes a 150-word defense
3. Spawn a fresh **jury agent** that reads all defenses, scores them on evidence/relevance/persuasiveness, and eliminates the weakest
4. See `prompts/judge.md` → "Elimination Tie-Breaker Jury" for the exact prompt templates
5. Read the jury verdict from `/tmp/debate-session/phase4/tiebreak-verdict.md` and continue

### Repeat until exactly 2 remain

Keep eliminating until exactly **2 finalists** remain. Phase 5 requires exactly 2 — never send 3+.

```
Round 1: 5 products → eliminate 1 → 4 remain
Round 2: 4 products → eliminate 1 → 3 remain
Round 3: 3 products → eliminate 1 → 2 remain ✓
```

For each additional round, pass cumulative votes for tie-break step 3:
```bash
mkdir -p /tmp/debate-session/phase4/round-{N}
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/vote_tallier.py /tmp/debate-session/phase4/round-{N}/ --cumulative '{"ProductA": 2, "ProductB": 1}'
```
(Use the `cumulative_votes` from prior tally outputs to feed into the next round.)

### Save state after Phase 4
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py update-state '{"phase": "phase4-complete", "finalists": ["Winner Pick", "Runner Pick"], "eliminated": ["Elim1", "Elim2", "Elim3"], "cumulative_votes": {"Elim1": 3, "Elim2": 2, ...}}'
```

### User gate

**AskUserQuestion**:
> "Finalists: [list with brief rationale]. Eliminated: [list with vote counts]. Proceed to final evaluation?"

---

## Phase 5 — Finals

**Goal**: Paired debate, 2-step judge (from MAD), forced revision (from agent-for-debate), jury validation.

### Step 1: Final debate

Spawn finalist champion agents **in parallel**. Each defends their pick against the other finalist(s) specifically.

**Forced revision** (from agent-for-debate, analysis.md:627-643): each agent must produce at least 1 revision cycle — first write their defense, then read the opponent's defense and write a revised version.

```
For each finalist agent (champion of finalist product):
  Task(
    subagent_type: "general-purpose",
    model: "opus",
    prompt: <debater.md debate round template> +
            "You are the champion of [your finalist product]." +
            "Your opponent champions: [other finalist product(s)]" +
            "Their opening statement evidence: [Phase 2 opening for opponent]" +
            "Their latest debate position: [last round position for opponent]" +
            "Your opening statement evidence: [Phase 2 opening for you]" +
            "Your latest debate position: [last round position for you]" +
            "Write a 400-word maximum defense. Be specific about why YOUR pick beats the opponent on the query's requirements." +
            "Write to /tmp/debate-session/phase5/finals-agent-{N}.md"
  )
```

Then run a **revision round** — each finalist reads the other's defense and writes a revised response:
```
For each finalist agent:
  Task(
    subagent_type: "general-purpose",
    model: "opus",
    prompt: "You previously defended [your product]. Your opponent has now responded:" +
            "[opponent's finals-agent-{M}.md content]" +
            "Produce a revised defense (300 words max). Address their strongest points directly." +
            "Write to /tmp/debate-session/phase5/finals-agent-{N}-revised.md"
  )
```

### Step 2: Two-step judge

Adapted from MAD's "ultimate deadly technique" (interactive.py:199-221):

**Step 1 — Strip to bare candidates** (fresh judge, no debate context):

From MAD `judge_prompt_last1`:
```
Affirmative side arguing: ##aff_ans##
Negative side arguing: ##neg_ans##
Now, what answer candidates do we have? Present them without reasons.
```

Adapted: Judge sees ONLY product names + factual specs. No persuasion, no adjectives.

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: <judge.md Step 1 template> +
          "Finalist products with bare facts only" +
          "Write to /tmp/debate-session/phase5/judge-step1.md"
)
```

**Step 2 — Fresh evaluation with evidence**:

Key insight from MAD: use `memory_lst[2]` (first-round response, not latest) to avoid recency bias.

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: <judge.md Step 2 template> +
          "Stripped candidates: [Step 1 output]" +
          "Opening statement evidence (FIRST ROUND — before groupthink): [Phase 2 openings]" +
          "Final round evidence: [Phase 5 finals]" +
          "Write to /tmp/debate-session/phase5/judge-step2.md"
)
```

Get judge input data:
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py format-judge-input
```

### Step 3: Forced revision of judgment

From agent-for-debate's mandatory revision rule (analysis.md:975-990):
> "至少要让他们修改1次" (Must make them revise at least once)

Generate 2-3 devil's-advocate concerns, run revision prompt from `judge.md`. Write to `/tmp/debate-session/phase5/judge-revision.md`

### Step 4: Jury validation

From elimination_game's jury mechanism (analysis.md:604-675): eliminated agents form the jury and vote on finalists using their accumulated debate context.

Spawn **eliminated agents as jurors** — they bring their debate history and private notes:
```
For each eliminated agent (from state["eliminated"]):
  Task(
    subagent_type: "general-purpose",
    model: "opus",
    prompt: <judge.md jury validation template> +
            "You are {eliminated_agent_persona}." +
            "You were eliminated in Phase 4 but now serve as a juror." +
            "Your debate history:" +
            [read their phase2/agent-{N}-opening.md] +
            [read their phase3/round-{R}/agent-{N}.md for each round they participated in] +
            [read their phase3/round-{R}/private/ notes if any] +
            "Winner: [X], Runner-up: [Y]" +
            "Key evidence summary from finals" +
            "Verdict: VALIDATED or CHALLENGED" +
            "Write to /tmp/debate-session/phase5/jury-{N}.md"
  )
```

Typically 3 eliminated agents serve as jurors (the 3 eliminated in Phase 4).

**If majority of jurors challenge**: runner-up becomes winner. Write final verdict to `/tmp/debate-session/phase5/final-verdict.md`

### Save state after Phase 5
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py update-state '{"phase": "phase5-complete", "winner": "Winner Product", "runner_up": "Runner-up Product"}'
```

---

## Phase 6 — Synthesis

**Goal**: Structured recommendation with comparison table, narrative, and sources.

### Compile all evidence
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py compile-synthesis
```

### 3-level synthesis fallback (from Mysti, analysis.md:912-921)

**Level 1 — Primary synthesis**:
```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: <synthesizer.md primary template> +
          "Evidence archive: [compiled evidence from orchestrator]" +
          "Write to /tmp/debate-session/phase6/synthesis.md"
)
```

Required output sections:
1. Recommendation (winner + one-sentence verdict + **"Buy here" link**)
2. Comparison table (ALL products, not just finalists — name, price, key specs, best for, worst for, verdict)
3. Narrative (2-3 paragraphs reasoning chain, referencing debate moments)
4. Case for runner-up (when it might be better + **"Buy here" link**)
5. What the debate missed (evidence gaps, honest limitations)
6. Sources (deduplicated URLs organized by product)

**Level 2 — Backup** (if primary fails or is missing sections):
Run a second agent with simplified prompt asking just for table + verdict + sources.

**Level 3 — Concatenate raw** (if both fail):
```
*Full synthesis unavailable. Individual agent findings below:*

## Judge's Verdict
[judge-step2.md content]

## Agent Research Summaries
[each agent's final position]
```

### Link validation (mandatory)

Before presenting, **verify all "Buy here" links actually work**:

```
For each "Buy here" link in the synthesis:
  Use WebFetch(url=link, prompt="Does this page exist and show the correct product? Return YES or NO with the product name shown.")

  If link is broken or wrong product:
    Use WebSearch(query="buy [product name] [retailer]") to find the correct link
    Replace the broken link in the synthesis
```

Both the winner and runner-up **must** have working direct product links. Do not present the synthesis until links are validated.

### Present to user

Read and display `/tmp/debate-session/phase6/synthesis.md` as the final output.

---

## Verbatim Prompt Configs (from research)

### MAD Config (adapted from config4all.json)
```json
{
  "debate_topic": "",
  "player_meta_prompt": "You are a product research debater. It's not necessary to fully agree with each other's perspectives, as our objective is to find the best recommendation.\nThe research query is:\n##query##",
  "moderator_meta_prompt": "You are a moderator evaluating product recommendations. Two or more agents will present their picks and argue their merits for:\n\"##query##\"\nAt the end of each round, evaluate which recommendation is strongest.",
  "negative_prompt": "##aff_ans##\n\nYou disagree with this recommendation. Provide your alternative pick and reasons.",
  "debate_prompt": "##oppo_ans##\n\nDo you agree with this perspective? Provide your reasons and updated recommendation.",
  "judge_prompt_last1": "Agent A recommends: ##aff_ans##\n\nAgent B recommends: ##neg_ans##\n\nWhat product candidates do we have? Present them without reasons — facts and specs only.",
  "judge_prompt_last2": "For the query: ##query##\nSummarize your reasons and give the final recommendation. Output as JSON: {\"Reason\": \"\", \"winner\": \"\", \"runner_up\": \"\"}"
}
```

### Agreement Intensity (from DebateLLM google_ma_debate.yaml)

Configurable 0-10 scale. Default is level 9 (empirically optimal at 90% deference).
Set during init or via `update-state`:
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py update-state '{"agreement_intensity": 7}'
```

| Level | Behavior |
|-------|----------|
| 0 | Completely disagree with all other agents |
| 1-3 | Skeptical, strong reservations |
| 4-5 | Consider with reservations (50/50) |
| 6-8 | Incorporate evidence 60-80% |
| **9** | **Incorporate 90%, almost no reservations (default, empirically optimal)** |
| 10 | Fully incorporate all evidence |

The full table is in `debate_orchestrator.py` `AGREEMENT_INTENSITY` dict. The suffix is automatically appended to debate context by `format-debate-context`.

### Convergence Thresholds (from Mysti)
```
CONVERGED: agreementRatio >= 0.7 AND avgStability >= 0.8
STALLED:   prevConvergence >= currentConvergence AND avgStability < 0.3
FORMULA:   overallConvergence = (agreementRatio * 0.6) + (avgStability * 0.4)
```

---

## Design Rules with Sources

| Rule | Implementation | Source (repo + line range) |
|------|---------------|---------------------------|
| Prompt relay (no agent-to-agent) | Files in /tmp/debate-session/ | Mysti BrainstormManager.ts:69-89 |
| Dense topology | All see all per round | debate-or-vote main.py:get_new_message, L147-165 |
| Agreement intensity 9 | 90% deference suffix | DebateLLM google_ma_debate.yaml, L549 |
| Vote on weakest | Elimination targets worst | elimination_game game_engine.py (reconstructed), L758-767 |
| 2-step judge | Strip → evaluate | MAD interactive.py:199-221 (judge_prompt_last1 + last2) |
| First-round evidence for judge | memory_lst[2] not latest | MAD interactive.py:620-621 |
| Forced disagreement | negative_prompt injection | MAD config4all.json negative_prompt |
| Forced revision (1+ cycle) | Reviewer mandates revision | agent-for-debate reviewer prompts ("至少修改1次") |
| Convergence: keyword + stability | 10 agree + 10 disagree patterns | Mysti BrainstormManager.ts:1462-1527 |
| Text similarity (Jaccard-like) | Words > 3 chars, max-normalized | Mysti BrainstormManager.ts:1532-1545 |
| 4-step tie-break | Plurality → weight → cumulative → jury tiebreak | elimination_game elimination.py (reconstructed), L811-884 |
| Decreasing word limits | 500 → 400 → 300 | elimination_game word_limits [70,50,30] adapted |
| 3-level synthesis fallback | Primary → backup → concatenate | Mysti BrainstormManager.ts:912-921 |
| Jury validation | Eliminated agents serve as jurors with debate history; majority vote | elimination_game jury mechanism, L604-675 |
| Domain-specific personas | 4 domain sets × 5 personas | Concept from debate-or-vote DyLAN (model_utils.py:71-86); personas custom-written for product research, not ported from original 13 academic DyLAN personas |
| Per-agent memory (not shared) | Each agent has own context | MAD Innovation 2, per-agent memory_lst |
| Position-aware prompt switching | First/middle/last speaker variants | agent-for-debate Innovation 4, position prompts |
| Delphi facilitator scoring | LLM-based convergence override (N/10) | Mysti BrainstormManager.ts:735-741 |
| Context window overflow mgmt | Progressive truncation preserving first/last ¶ | Mysti BrainstormManager.ts:580-615 |
| Judge-based early stopping | Judge checks for clear winner after each round | DebateLLM Tsinghua MAD early termination |
| Private deliberation channel | Paired agents exchange private critiques | elimination_game dual channel mechanism |
| Consumer domain personas | Product Analyst, Domain Expert, UX Researcher | debate-or-vote DyLAN, domain detection |

---
> Source: [miiraheart/claude-debate-skill](https://github.com/miiraheart/claude-debate-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
