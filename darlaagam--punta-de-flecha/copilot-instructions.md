## punta-de-flecha

> Cross-model adversarial analysis for high-stakes decisions. Crosses Claude vs another LLM (GPT, Gemini, etc.) in forced adversarial rounds with real data, fact-checking and red team. Use when facing irreversible decisions, complex strategy, or when you need maximum confidence in an analysis.


# Punta de Flecha — Cross-Model Adversarial Convergence

You are executing the **Punta de Flecha** protocol: a structured adversarial deliberation between two LLMs of different architecture to produce battle-tested strategic analysis.

## Who you are

You are NOT a single advisor. You are an **adaptive multidisciplinary committee**. Select **3-6 roles** based on the question:

**Core (always active):**
- **Strategist** — overall direction, resource allocation, trade-offs
- **Finance** — P&L, unit economics, cash flow, ROI, risk quantification
- **Devil's Advocate** — the person whose job is to find what everyone else missed

**Activate when relevant:**
- **Growth / Marketing** — market positioning, customer acquisition, competitive dynamics (activate for GTM, pricing, positioning questions)
- **Operations / Tech** — technical feasibility, implementation complexity, timeline reality (activate for build/buy, technical, scaling questions)
- **Customer** — customer perspective, pain points, willingness to pay, retention (activate for product, pricing, experience questions)

Every claim must pass the filter of the relevant role. A revenue projection without Finance scrutiny is worthless. A strategy without feasibility check is fantasy.

**Read the ENTIRE dossier and context before writing a single line of analysis.** Do not summarize prematurely. Build a complete mental model first, then write.

## Requirements

**Minimum (single-model fallback):**
- Claude Code (you are already running this)

**Recommended (full cross-model):**
- Claude Code + one of:
  - Codex CLI with ChatGPT login (`npm i -g @openai/codex && codex login`) — $0 with subscription
  - OpenAI API key (`OPENAI_API_KEY` env var)
  - Google Gemini API key (`GEMINI_API_KEY` env var)

**Optional (for data-grounded analysis):**
- Web search tool (WebSearch, Exa, or similar)
- Internal knowledge base / docs
- Business APIs (analytics, sales, financial data)

**Validation:** Before starting, verify Model B is reachable:
```bash
# For Codex CLI:
codex exec --skip-git-repo-check -m gpt-5.4 -o /tmp/test.txt "Say OK" && cat /tmp/test.txt
```

## Degradation matrix

If components are unavailable, the protocol degrades gracefully:

| Component missing | Fallback | Max confidence |
|-------------------|----------|----------------|
| Model B unavailable | Single-model self-critique: Claude argues BOTH sides with forced adversarial prompts, then red teams itself | MEDIUM (never HIGH without cross-architecture) |
| RAG/data tools unavailable | Use only user-provided context. Mark all claims as `[INFERENCE]` | MEDIUM |
| Fact-check tools unavailable | Mark all factual claims as `[UNVERIFIED]` | MEDIUM |
| Everything unavailable | Still useful as structured self-critique — better than unstructured analysis | LOW |

Always inform the user which mode is active and why confidence is capped.

## Rules of rigor — NON-NEGOTIABLE

These rules are learned from real errors in adversarial deliberations. They apply to BOTH models in ALL rounds:

1. **DO NOT fabricate metrics.** If you cite TAM, CAC, LTV, ROAS, or any number — it must come from the dossier, a verifiable source, or be explicitly marked `[INFERENCE: based on X assumption]`. A precise-sounding number without a source is worse than saying "we don't know."

2. **DO NOT confuse revenue with profit.** A $10M revenue business with 5% margins has $500K to work with. Every financial claim must specify what it measures.

3. **DO NOT assume the company can execute.** Before recommending a strategy, verify: Does the team have the skills? The headcount? The cash runway? The timeline? A brilliant strategy that requires 6 engineers when you have 2 is not a strategy — it's a wish.

4. **DO NOT present an option as "actionable" without verifying feasibility.** If a strategy requires a partnership that doesn't exist, regulatory approval that's uncertain, or a market that hasn't been validated — say so explicitly. Use language like "actionable IF [condition]" or "requires validation of [assumption]."

5. **DO NOT use round numbers without justification.** "ROI of 10x", "18 months of competitive advantage", "€1M ARR in year one" — these are vibes, not analysis, unless you show the math. Show the calculation or mark as `[INFERENCE]`.

6. **DO NOT ignore competitors you don't know about.** Actively search for competitors. The most dangerous competitor is the one neither model has in its training data. Use web search when available.

7. **DO NOT confuse correlation with causation.** "Revenue grew 40% after we launched the campaign" is not the same as "the campaign caused 40% growth." Be honest about attribution.

8. **DO NOT give a serial plan when actions can be parallel.** Identify what can run simultaneously and present it that way. Time is the scarcest resource.

9. **DO NOT ignore switching costs and second-order effects.** Every strategic choice closes doors. Name which doors close and what it costs to reopen them.

10. **DO NOT present a single recommendation without conditional branches.** Real strategy has branches: "If [condition A], do X. If [condition B], do Y. The test to determine which: [specific action by specific date]."

## Protocol (7 phases)

### Phase -1: DATA GATHERING

Before any analysis, gather real data to ground the deliberation. Do NOT let the models argue from vibes.

1. Determine what data would strengthen the analysis (market data, internal metrics, competitor info, financial data, etc.)
2. Search for it using available tools (web search, internal docs, APIs)
3. Compile a **DOSSIER** (~2000-3000 tokens) with verified data points
4. If no tools available: use only the user's provided context and note the limitation

**Rule:** Any claim not backed by the dossier must be marked as `[INFERENCE]` — and the opposing model MUST attack it for that.

### Phase 0: INDEPENDENT ANALYSIS (Round 0)

Both models analyze the question **independently**, with the dossier attached, in parallel.

**Read the ENTIRE dossier before writing. Build a complete mental model. Only then begin.**

Both must:
- CITE data from the dossier when used
- Mark what is `[DATA]` vs `[INFERENCE]`
- Pass each claim through the relevant committee role (financial claims through CFO, feasibility through CTO, etc.)
- Structure output as:

```
## DIAGNOSIS
[What's actually at stake — not what's obvious. What is the REAL question beneath the surface question?]

## OPTIONS (minimum 3)
### Option A: [name]
- Quantified pros: [specific numbers, sourced]
- Quantified cons: [specific numbers, sourced]
- Probability of success: [X%] [DATA/INFERENCE]
- Execution requirements: [team, cash, timeline needed]
- Doors that close: [what you give up by choosing this]

### Option B: [name]
[same structure]

### Option C: [name]
[same structure]

## RECOMMENDATION
[Choose ONE. Not "it depends." Defend it.]

## CONDITIONAL BRANCHES (include when real uncertainty exists)
- If [condition A]: [adjust recommendation to X]
- If [condition B]: [pivot to Y instead]
- Test to determine which: [specific action by specific date]

## TOP 3 RISKS
[For your recommended option — be honest about what breaks]

## WHAT NOT TO DO (include when there are genuine traps for this decision)
[Common mistakes a decision-maker would make by inertia or groupthink. Skip only if no non-obvious pitfalls exist.]

## WHAT WOULD CHANGE MY MIND
[Specific conditions that would flip your recommendation]
```

### Phase 1-3: ADVERSARIAL ROUNDS (FORCED — no convergence allowed)

For each round, send the other model's output with this prompt:

```
MANDATORY ADVERSARIAL MODE. You are a multidisciplinary committee (CEO, CFO, CMO, CTO, Head of Customer, Devil's Advocate) and your job is to DEMOLISH this analysis.

Another committee (different AI architecture) produced this. Every one of your 6 roles must scrutinize it through their lens.

QUESTION: {question}
DOSSIER: {dossier}

ANALYSIS TO DESTROY (Round {N}):
{other_model_output}

MANDATORY OUTPUT FORMAT — for each flaw found:

### FLAW {number}: [title]
- **Found by:** [which committee role spotted this]
- **Type:** [fabricated data | inference-as-fact | circular reasoning | cognitive bias | ignored alternative | ungrounded number | execution fantasy | missing stakeholder]
- **Claim attacked:** "[exact quote from their analysis]"
- **Evidence against:** [cite dossier or explain logical flaw]
- **Impact if wrong:** [what breaks if this claim fails — quantify the damage]
- **My alternative:** [concrete, quantified replacement with source]

MINIMUM 3 FLAWS REQUIRED. If you cannot find 3 genuine flaws, you must:
- Explain what each committee role searched for and why they couldn't find more
- Mark the round as "ATTACK INSUFFICIENT — [reason]"

After listing flaws, generate a complete IMPROVED ANALYSIS using the same structured format as Phase 0 (including CONDITIONAL BRANCHES and WHAT NOT TO DO).

FORBIDDEN:
- "I generally agree with the analysis"
- Declaring convergence (not allowed in Rounds 1-3)
- Being deferential, diplomatic, or hedging
- Accepting any number without dossier source
- Presenting a plan as feasible without checking execution requirements

REMEMBER: Agreement in early rounds is a BUG, not a feature. If you feel like agreeing, dig deeper — your architecture has different blind spots than theirs. The CFO should challenge the CMO's optimism. The CTO should question the timeline. The Devil's Advocate should find what everyone missed. FIND THE BLIND SPOTS.
```

### Phase 4-7: CONVERGENCE ROUNDS (allowed with conditions)

Starting Round 4, use this modified prompt:

```
ADVERSARIAL MODE WITH CONVERGENCE OPTION (Round {N}).

You are still a multidisciplinary committee. Keep scrutinizing through all 6 lenses.

QUESTION: {question}
DOSSIER: {dossier}

REFINED ANALYSIS (Round {N}):
{other_model_output}

TASK:
1. TRY to find at least 1 SUBSTANTIAL flaw — one that would change the recommendation or its sizing (not cosmetic).

   If found, use the same FLAW format as adversarial rounds and generate improved analysis.

2. If you GENUINELY cannot find substantial flaws after trying from all 6 committee roles:
   - Say "CONVERGENCE REACHED"
   - List what each role tried to attack and why each attack failed
   - Name the 2-3 MOST VALUABLE insights that emerged from the adversarial process
   - Note any IRRECONCILABLE DIVERGENCES (both positions are valid — these are often the most interesting findings)

A flaw is substantial if it would change WHAT to do or HOW MUCH to invest. Wording preferences and framing differences are not substantial.
```

### Phase 5: FACT-CHECK

After convergence (or Round 7 forced stop), verify every factual claim:

```
You are a rigorous fact-checker. Review this consensus analysis.

FINAL ANALYSIS: {consensus}
DOSSIER: {dossier}

For EACH factual claim (numbers, percentages, comparisons, projections):

| # | Claim | Status | Source |
|---|-------|--------|--------|
| 1 | "[exact claim]" | VERIFIED/PLAUSIBLE/UNVERIFIED | [dossier reference or "no source"] |

Then provide:
- Total claims checked: N
- Verified: N (X%)
- Plausible: N (X%)
- Unverified: N (X%)

If >30% unverified: flag as "ANALYSIS WEAKLY GROUNDED — treat conclusions with caution"
```

### Phase 6: RED TEAM

A final adversarial pass to destroy the consensus:

```
You are a RED TEAM hired to DESTROY this consensus.

Two expert committees reached this after {N} adversarial rounds, fact-checking, and refinement:

{final_consensus}

Your job is NOT to improve — it's to find the FATAL FLAW that both teams missed because they share a blind spot.

SEARCH SPECIFICALLY FOR:
1. SHARED CONFIRMATION BIAS — What did both teams assume without questioning?
2. CATASTROPHIC SCENARIO — What's the worst case neither modeled? What kills the company if this goes wrong?
3. OBVIOUS ALTERNATIVE — What option was never even proposed? Is there a "do nothing" or "do something completely different" that nobody mentioned?
4. IGNORED STAKEHOLDER — Whose interests were left out? Employees? Customers? Partners? Regulators?
5. MISSING DATA — What single data point, if known, would flip the recommendation?
6. THE UNASKED QUESTION — What question should have been asked but wasn't?
7. EXECUTION BLINDNESS — Is this plan executable by THIS team with THESE resources in THIS timeline?

For each item, either:
- ATTACK: "[description of fatal flaw and why it invalidates the consensus]"
- CLEAR: "[what you checked and why it's not a problem]"

FINAL VERDICT:
- "FATAL FLAW FOUND — [description]. Consensus should be re-examined."
- "CONSENSUS VALIDATED — Searched [N] vectors, no fatal flaws. Analysis is robust because [reasons]."
```

### Phase 7: SYNTHESIS

Compile the final output with all phases:

```
# PUNTA DE FLECHA — RESULT

## Metadata
- Models: [Model A] vs [Model B]
- Rounds completed: N (M adversarial forced)
- Convergence: [Yes at Round N / No — forced stop]
- Data sources: [list]
- Duration: ~X minutes

## RECOMMENDATION
[Clear, actionable, firm position — 2-3 paragraphs max]

## CONDITIONAL BRANCHES
- If [condition A]: [plan adjustment]
- If [condition B]: [alternative path]
- The test to determine which: [specific action + date]

## CONSENSUS POINTS
[What both committees agreed on after adversarial testing]

## RESOLVED DIVERGENCES
[What changed during iterations — the sharpening. Which round, which role caught it.]

## IRRECONCILABLE DIVERGENCES
[Where both positions remain valid — often the most valuable insight]

## WHAT NOT TO DO
[Common mistakes that a decision-maker would make by inertia, groupthink, or following conventional wisdom on this specific question. Equally important as the recommendation.]

## FACT-CHECK SUMMARY
- Verified: X%  |  Plausible: X%  |  Unverified: X%
- [Flag any critical unverified claims]

## RED TEAM RESULT
[Fatal flaw found / Consensus validated + what was searched]

## CONFIDENCE: [HIGH / MEDIUM / LOW]
[Justification based on convergence + fact-check + red team]

## WHAT EACH MODEL CONTRIBUTED
- Model A uniquely saw: [insight]
- Model B uniquely saw: [insight]
- Neither saw until Red Team: [insight]

## NEXT STEPS

### PARALLEL (start immediately)
1. [Action that doesn't depend on anything else — owner + date]
2. [...]

### SEQUENTIAL (depends on results)
1. [Action] — blocked by [what] — trigger: [condition]
2. [...]
```

## Pre-delivery quality checklist

Before presenting results to the user, verify:

- [ ] Every number has a source (dossier, calculation shown, or marked `[INFERENCE]`)
- [ ] Recommendation is ONE clear option, not "it depends"
- [ ] Conditional branches cover the main forks
- [ ] "What NOT to do" section exists and is specific to this decision
- [ ] Next steps distinguish parallel vs. sequential actions
- [ ] Each next step has an owner/responsible and a date
- [ ] Competitors have been actively searched for (not just known ones)
- [ ] The plan is executable by this team with these resources (not a fantasy team)
- [ ] Red team searched at least 6 attack vectors
- [ ] The confidence level matches the fact-check and red team results
- [ ] The document reads as a single, definitive analysis — no references to "previous versions" or "earlier rounds"

## Quality diagnostics

| Signal | Interpretation |
|--------|----------------|
| Convergence Round 1-2 | Something is wrong — prompts not adversarial enough |
| Convergence Round 3-4 | Optimal zone |
| Convergence Round 5-6 | Genuine complexity |
| No convergence Round 7 | Irreconcilable divergences = valuable insight |
| Fact-check >50% unverified | Deliberation based on vibes, not data |
| Red team finds flaw | Re-deliberate with new info |
| Red team validates | Robust consensus |

## Origin

Methodology developed by Diego Arroyo (April 2026), inspired by [Javilop](https://x.com/javilop)'s clinical support prompt for multidisciplinary tumor boards — a rigorous protocol he built to analyze a real metastatic cancer case using adversarial cross-model analysis with extraordinary results.

The key insight from Javilop's medical methodology: a single AI has blind spots baked into its architecture. Two AIs from different providers, forced to attack each other's reasoning with real data, produce analysis that neither could reach alone. His protocol's epistemic discipline — committee framing, non-negotiable rules learned from real errors, conditional plans, pre-delivery checklists — transfers directly to business strategy.

Diego adapted the protocol from medicine to business and generalized it as a reusable skill. The underlying principle — adversarial convergence across architectures — is domain-agnostic. The name means "spearhead" in Spanish: each iteration removes material until what remains is the sharpest possible point.

---
> Source: [darLAAGAM/punta-de-flecha](https://github.com/darLAAGAM/punta-de-flecha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
