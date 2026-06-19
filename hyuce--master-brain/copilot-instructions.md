## master-brain

> Use when inferences or evaluations need to be evidence-backed; when prior reasoning felt premature or conclusion-first; when analyzing a document, claim, decision, or argument; when quality control of a prior analysis is needed; or when a judgment must hold up under adversarial scrutiny. Do NOT use for simple lookups, single-step operations, or routine tasks without a judgment component.


# master-brain

## Overview

**No inference without evidence. No conclusion without verification.**

Every analysis passes through a mandatory 5-stage reasoning loop. Every stage is processed as a distinct, demarcated unit of work. The loop repeats from Observation if Verification fails.

**Violating the letter of the rules is violating the spirit of the rules.** This principle binds at every stage. No stage may be skipped or waved off as "obvious."

## When to Use

```
Analysis, decision, or evaluation needed?
├─ No  → Don't use this skill
└─ Yes → Can it be answered in a single step?
          ├─ Yes → Don't use this skill
          └─ No  → USE this skill
```

**Use this skill for:**
- Complex multi-step decisions with trade-offs
- Document analysis: claims, findings, evidence, gaps
- Quality control of structured analysis (yours or someone else's)
- Gap analysis, benchmarking, evaluation against a standard
- Any inference, evaluation, or judgment that must be defensible

**Do not use for:**
- Simple informational queries ("where is this file?")
- Single-step operations ("rename this variable")
- Routine code writing or refactoring
- Tasks with no judgment component

## The 5-Stage Loop

```
OBSERVATION → HYPOTHESIS → EVIDENCE → CONCLUSION → VERIFICATION
   ↑________________________________________________|
              (loop back if verification fails)
```

| # | Stage | Question |
|---|-------|----------|
| 1 | Observation | What is present? |
| 2 | Hypothesis | What could it mean? |
| 3 | Evidence | What supports or refutes each candidate? |
| 4 | Conclusion | What is the final judgment? |
| 5 | Verification | Under what conditions is this wrong? |

**Rules that bind at every stage:**
- Run the stages in order. Do not skip.
- Process each stage as a distinct unit. Do not batch.
- If Verification fails, return to Observation with the new evidence. Do not patch the conclusion.

### Stage 1 — Observation

Collect data **without interpretation**. List facts, not opinions.

**Do:**
- For a document: itemize claims, structure, sections, evidence cited, gaps.
- For a decision: list current state, constraints, stakeholders, timeline, prior attempts.
- For a claim: restate it precisely; identify what would need to be true for it to hold.
- For QA: extract structure, numbering, headings, paragraph count.

**Do not:**
- Add commentary, judgment, or "I think" statements.
- Pre-classify findings (that is Hypothesis work).
- Skip items because they seem irrelevant.

### Stage 2 — Hypothesis

Generate candidate explanations for each observation. Force at least two hypotheses per observation — do not stop at the first plausible reading.

**Do:**
- "This could mean X..."
- "A rule could be violated if Y..."
- "Alternatively, Z is also possible..."

**Do not:**
- Settle on one hypothesis without considering alternatives.
- Treat the first hypothesis as the conclusion.
- Skip the stage because "it's obvious."

### Stage 3 — Evidence

Gather concrete evidence. For each hypothesis, list what supports it and what contradicts it.

**Do:**
- Cite the specific rule, document section, or fact.
- Verify cross-references (does the cited section actually exist?).
- Check statistics, dates, names — every concrete claim.
- Match claims against any supplied domain rules.
- **Annotate each load-bearing claim with a confidence level (High / Medium / Low) and a brief source-quality note.** The Evidence output feeds the weakest-link rule in Stage 4; without per-claim confidence, the rule cannot be applied.

**Do not:**
- Accept a hypothesis on its own plausibility.
- Use vague evidence ("generally accepted that...").
- Skip the contradictions.

### Stage 4 — Conclusion

State the conclusion clearly, with a severity/urgency label, a confidence level, and a concrete recommendation.

**Do:**
- For findings: end with a `[Urgency] [Direction]` label in **bold** and a confidence level. Format: *"Therefore, **[Major Gap]** has been identified. **Confidence: High**."* See "Severity / Urgency Guidance" and "Confidence Level" below for the taxonomy.
- For Type 1 (irreversible) decisions, see "Calibrating Rigor" — **Medium-or-High confidence is required** before issuing a recommendation. If confidence is Low, return to Stage 3 to gather more evidence.
- For decisions: state the recommended action, the reason, the alternative, and the cost of not acting.
- For QA: state pass/fail with reason.
- Recommendations should be concrete and actionable. Use the modal form "should" or "must."

**Do not:**
- Issue a conclusion without a `[Urgency] [Direction]` label.
- Issue a conclusion without a confidence level, or one set without applying the weakest-link rule.
- Issue a finding without a recommendation (a finding without a fix is a complaint, not an analysis).
- Hedge with "I think" or "in my opinion."
- Assign a confidence level you cannot justify with evidence.

### Stage 5 — Verification

Test the conclusion adversarially. Apply intellectual honesty.

**Do:**
- Ask: "What would have to be true for this conclusion to be false?"
- Try alternative scenarios: "What if the data is wrong?"
- Check the conclusion against every domain rule in scope.
- Run the rule-by-rule checklist (if a domain rules file is in use).
- Run the cognitive bias check (see below).

**Pass:** Loop ends. Present the conclusion.
**Fail:** Return to Observation with the new evidence. Increment the loop counter.
**If no new evidence is available:** Exit with the current confidence level; document the limitation.

### Stage Markers

Each stage produces a marker — a labelled unit of work. The marker is the contract for what each stage emits; the implementation can vary.

| Stage | Marker label | Loop counter | Next stage |
|-------|--------------|--------------|------------|
| Observation | `Observation` | 1 (or N+5 on loop-back) | Hypothesis |
| Hypothesis | `Hypothesis` | 2 (or N+6) | Evidence |
| Evidence | `Evidence` | 3 (or N+7) | Conclusion |
| Conclusion | `Conclusion` | 4 (or N+8) | Verification |
| Verification (pass) | `Verification` | 5 (or N+9) | — (loop ends) |
| Verification (fail) | `Verification` | 5 (or N+9) | Loop back to Observation |

**Implementation options:**

- **Step-by-step tool** (e.g., `sequential-thinking_sequentialthinking`): one tool call per stage. Set `nextThoughtNeeded=true` except on Verification pass. On loop-back, set `isRevision=false` and `needsMoreThoughts=true`.
- **Prose with section headers:** use `## Observation`, `## Hypothesis`, etc. as section dividers. One section per stage, no merging.
- **Numbered list:** `1. Observation: ...` `2. Hypothesis: ...`. Compact, useful for short analyses.

The contract is the same regardless of implementation: **one stage per unit, no merging**.

## Calibrating Rigor

Before running the loop, classify the decision. Rigor scales with reversibility.

```
Is the decision reversible at low cost?
├─ No  → IRREVERSIBLE (Type 1) — run the FULL loop with high rigor.
│        Examples: choosing a database, signing a contract, hiring, deleting production data.
├─ Yes, but with non-trivial cost → REVERSIBLE (Type 2) — run the loop, accept that speed matters.
│        Examples: refactoring a function, drafting a document, picking a library.
└─ Yes, with negligible cost → TRIVIAL — Observation + Conclusion may suffice; document the abbreviated path.
```

**Implications:**

- **Type 1:** Force ≥ 2 hypotheses. Spend extra effort in Evidence. Run the bias check explicitly. Do not issue a recommendation unless confidence is **Medium or High**.
- **Type 2:** Run the standard loop. Document the reversibility so future readers know why speed was prioritized over depth.
- **Trivial:** Skip Hypothesis and Evidence only if the cost of being wrong is also trivial. Otherwise run the full loop.

## Assumption Audit

Before running the loop, identify what the analysis takes for granted. Hidden assumptions cause more errors than flawed logic.

**Process:**

1. List assumptions the analysis framework depends on — what must be true for the framework to be valid?
2. Classify each by risk:
   - **High:** conclusion reverses if assumption is false.
   - **Medium:** conclusion weakens but does not reverse.
   - **Low:** negligible impact on the conclusion.
3. Mark any assumption testable during Evidence as **testable**.
4. During Evidence, verify every testable assumption as a claim with its own confidence level.

| # | Assumption | Risk | Testable |
|---|-----------|------|----------|
| 1 | Data source is current and authoritative | High | Yes |
| 2 | Stakeholder's problem description is complete | Medium | Limited |
| 3 | No regulatory change pending | Low | No |

**If a High-risk assumption is unverified by the end of Evidence:** it counts as a load-bearing claim with **Confidence: Low**. The weakest-link rule applies automatically. Document it:

*"Therefore, **Note** has been recorded (conclusion assumes [X]; this could not be verified).*

## Worked Examples

### Example 1 — Database choice

A team is choosing between PostgreSQL and MongoDB for a new application.

- **Observation:** Current stack is JavaScript/TypeScript. Data model: users, orders, line items — highly relational. Team has 3 years of Postgres experience, no Mongo experience. Traffic: ~10k DAU projected. Cost ceiling: $500/mo for managed DB.
- **Hypothesis:** A) Postgres fits the relational model and team experience. B) Mongo's document model could simplify some access patterns. C) A hybrid is overkill at this scale.
- **Evidence:**
  - A: 4 years of Postgres in production elsewhere; team velocity is high; relational queries are the hot path.
  - B: One access pattern (user profile + preferences) is document-shaped, but a JSONB column in Postgres covers it.
  - C: Hybrid adds operational cost; not justified at this scale.
- **Conclusion:** Use PostgreSQL. Two findings:
  - Therefore, **Note** has been recorded (the relational data model fits Postgres; the team has 3 years of Postgres experience). **Confidence: High** (multiple independent signals).
  - Therefore, **Strength** has been observed (the team's 3-year Postgres experience is a competitive capability relative to the no-Mongo experience baseline). **Confidence: High** (verifiable, recent).
  Recommendation: should adopt Postgres; must define migration plan and JSONB usage convention.
- **Verification:** What if access patterns shift toward document-shaped? Postgres + JSONB covers it for the foreseeable scale. Switching cost is bounded. Decision holds.

### Example 2 — Blog post accuracy review

A blog post claims: "Postgres handles 10x more concurrent connections than MySQL out of the box."

- **Observation:** Claim is in paragraph 3. Source cited: a 2019 Stack Overflow answer.
- **Hypothesis:** A) Claim is accurate. B) Claim is oversimplified — "out of the box" is misleading because both DBs require configuration. C) Claim is outdated.
- **Evidence:**
  - Postgres default `max_connections=100`; MySQL default `max_connections=151`. Numbers don't support a 10x claim.
  - Both DBs scale via connection pooling in practice; raw connection count is rarely the bottleneck.
  - Source is 7 years old.
- **Conclusion:** Claim is misleading. Therefore, **Major Gap** has been identified (it would mislead readers). **Confidence: High**. Recommendation: should rephrase to "Postgres and MySQL both require pooling; their default `max_connections` values are similar"; or remove the claim.
- **Verification:** Re-read the source — it does compare connection limits but the 10x figure is not in it. Decision holds.

## Domain Rule Integration

The Evidence and Verification stages need a rule set to check against. Two ways to supply it:

**Option A — User-supplied rules file.** A project-specific rule document (style guide, framework, assessment standard) is provided. Evidence compares claims to the file; Verification runs a rule-by-rule checklist.

**Option B — Runtime construction.** No file is provided. Construct the rule set from the user's stated requirements at the start of the loop, then use it for Evidence and Verification. To elicit requirements, ask four questions during the Observation stage:

- **Success criteria** — what does "done" look like for this analysis?
- **Constraints** — budget, time, technical, regulatory.
- **Acceptable trade-offs** — what is preferred when two requirements conflict?
- **Out of scope** — what should Evidence and Verification ignore to prevent scope creep?

If the user does not answer, proceed with conservative defaults and note "requirements pending" as a finding in the Evidence stage.

The skill works either way. If neither is supplied, Evidence and Verification degrade to "logical consistency and stated requirements" — useful, but weaker than an explicit rules file.

### Domain Rules File Format

A domain rules file is structured as:

```markdown
# Domain: <name>
# Scope: <when these rules apply>

## Rule Group: <group name>
- **Rule:** <one-line statement of the rule>
  - **Example:** <concrete instance showing the rule satisfied>
  - **Violation indicator:** <how to detect a breach>

## Rule Group: <group name>
...
```

**Excerpt example:**

```markdown
# Domain: Engineering Blog Standards
# Scope: technical blog posts published under our company name

## Rule Group: Claim verification
- **Rule:** Empirical claims must cite a source published within the last 5 years.
  - **Example:** "Postgres handles 10x more connections (Smith, 2024) [link]."
  - **Violation indicator:** Empirical claim with no citation, or citation older than 5 years.

## Rule Group: Severity tagging
- **Rule:** Every finding must end with a `[Urgency] [Direction]` label in bold.
  - **Example:** "...therefore **Major Gap** has been identified."
  - **Violation indicator:** Finding paragraph ends without a bold severity label, or the label lacks a direction.
```

The Evidence stage checks claims against each rule's example and violation indicator. The Verification stage walks the entire rules file rule-by-rule.

## Severity / Urgency Guidance

A finding has two independent axes: **urgency** (how fast it must be addressed) and **direction** (deficit vs strength). Conflating them — as a single 7-level list does — produces ambiguous labels.

### Urgency axis (default 4 levels)

| Level | When to use |
|-------|-------------|
| **Critical** | Severe negative outcome if not addressed immediately |
| **Major** | Must be addressed within months; affects correctness, quality, or safety |
| **Minor** | Must be addressed within a year; affects polish or completeness |
| **Note** | Worth recording; no required action |

### Direction axis

| Direction | When to use |
|-----------|-------------|
| **Gap** | Requirement not met, or under-met |
| **Strength** | Requirement clearly exceeded (positive finding) |

### Combining

The full label is `[Urgency] [Direction]`, formatted in **bold** and stated with a fixed verb:

- *"Therefore, **Major Gap** has been identified."*
- *"Therefore, **Strength** has been observed."*
- *"Therefore, **Note** has been recorded."*

## Confidence Level

Confidence is the orthogonal axis to severity: severity says "how serious if true," confidence says "how well-supported is it."

### Two dimensions

**Breadth** — how many independent sources support the claim:

- Multiple independent sources
- One credible source + logical inference
- Single unverified source

**Quality** — how reliable the sources are:

| Quality | Examples |
|---------|----------|
| **High** | Peer-reviewed paper, official documentation, primary source |
| **Medium** | Established secondary source (industry report, well-known expert blog) |
| **Low** | Anonymous, outdated, unattributed, or self-published without review |

### Combined levels

| Level | When to use |
|-------|-------------|
| **High** | Multiple independent High-quality sources, OR a rule-by-rule check passes against the domain rules file |
| **Medium** | One strong source, OR multiple Medium-quality sources, OR a logical inference from High-quality evidence |
| **Low** | Single unverified source, OR contradictory evidence, OR Low-quality evidence only |

### The weakest-link rule

**Confidence = weakest link in the chain.** If any supporting claim in the Evidence stage is Low confidence, the whole conclusion is Low — unless that claim is non-critical to the main finding (e.g. an illustrative aside, not load-bearing evidence).

Walk the chain claim-by-claim. The conclusion's confidence equals the lowest confidence among the load-bearing claims.

### Format

Append to the finding after the severity label. Example: *"Therefore, **Major Gap** has been identified. **Confidence: Medium**."*

### Rules

- For Type 1 (irreversible) decisions, do not issue a recommendation with **Low** confidence. Gather more evidence first.
- **Low** confidence is a finding in itself: it tells the reader to verify before acting.

## Combining Severity and Confidence

Severity and confidence are independent axes. The same severity with different confidences means very different things:

| Severity | Confidence | Meaning | Implied action |
|----------|------------|---------|----------------|
| **Major Gap** | **High** | Serious issue, well-supported | Act now |
| **Major Gap** | **Low** | Possibly serious, evidence thin | Verify before acting |
| **Critical Gap** | **Medium** | Severe if true, evidence partial | Verify urgently, prepare to act |
| **Strength** | **High** | Clear positive finding | Preserve and replicate |
| **Strength** | **Low** | Possibly a strength, evidence thin | Verify before claiming |
| **Note** | **Medium** | Recorded, no action | Acknowledge in the analysis |

This table is read-only — it is the reader's decoder, not a rule the agent generates against. Use it to set reader expectations clearly when issuing a conclusion. The table shows **common combinations as examples**; for unlisted combinations, derive the implied action from the same axes (severity tells you how serious, confidence tells you how well-supported).

## Cognitive Bias Check

Run this short protocol before issuing the Conclusion, and again during Verification.

| Bias | Self-check | If failed |
|------|------------|-----------|
| **Confirmation bias** | Did I seek evidence against my preferred hypothesis, or only for it? | Return to Stage 3, gather counter-evidence. |
| **Anchoring** | Am I overweighting the first number/source I encountered? | Explicitly compare to a second, independent source. |
| **Sunk cost** | Am I defending a past decision because of what was already invested? | State the decision on its own merits, ignoring prior investment. |
| **Availability** | Am I overweighting recent or vivid examples? | Look for base rates and historical data. |
| **Overconfidence** | Is my confidence level justified by the evidence, or by intuition? | Lower the confidence level; flag the uncertainty. |

If any check fails, return to Stage 3 (Evidence) and re-collect.

## Red Flags — STOP and Restart the Loop

If any of these occur, **cancel the current loop and restart from Observation:**

- A stage was skipped or waved off as "obvious" → **Return to Observation.**
- Conclusion issued without a `[Urgency] [Direction]` label → **Return to Observation, apply severity guidance.**
- Conclusion issued without a confidence level → **Return to Observation, assign confidence.**
- Recommendation missing, or not in "should/must" form → **Return to Observation, formulate a concrete recommendation.**
- Verification marked "passed" without running the full rule check → **Return to Observation, re-run checklist item by item.**
- Verification marked "passed" without running the bias check → **Return to Observation, run the bias check.**
- Personal language used ("I think," "in my opinion") → **Return to Observation, rewrite in objective voice.**
- Cross-references not verified → **Return to Observation, locate the referenced section and confirm it exists.**
- A single paragraph mixes a finding with its recommendation → **Return to Observation, separate them.**
- A single paragraph covers more than one topic → **Return to Observation, split into separate paragraphs.**
- High-risk assumption unverified and not acknowledged in the conclusion → **Return to Observation, document the assumption.**

**All of these mean: cancel the loop, restart from Observation.**

## Rationalization Table

Agents will try to skip the loop. Pre-empt the common excuses:

| Excuse | Reality |
|--------|---------|
| "This is obvious, no loop needed" | Obvious-looking cases contain the most errors. Run the loop. |
| "The fix is already clear" | Only Verification proves the fix is not incomplete or wrong. |
| "One thought is enough" | Each stage is its own unit. Combining stages breaks the loop. |
| "5 stages is too long, I'll summarize" | Skipped stage = skipped error. Correctness beats speed. |
| "Just an impression, no severity needed" | A finding without severity doesn't communicate seriousness. Assign one. |
| "I can skip confidence, it's obvious" | Confidence is a separate axis from severity. Always assign. |
| "Bias check is overkill for this" | The bias check is 5 questions. Skipping it skips the most common failure mode. |
| "The recommendation is obvious, skip writing it" | A finding without a fix is a complaint, not an analysis. Write it. |
| "Not enough data for this section" | Data absence is itself a finding; note "review pending." |
| "Verification passed" | Verification passes only when every checklist item is checked. |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Treating the analysis framework as given without checking its premises | Run the Assumption Audit before Observation |
| Skipping the loop and writing the conclusion directly | Run all 5 stages for every analysis |
| Adding interpretation during Observation | Observation is raw data; interpretation belongs in Hypothesis |
| Stopping at a single hypothesis | Generate ≥ 2 hypotheses; force alternative explanations |
| Skipping Evidence and going straight to Conclusion | Evidence-less conclusion is unfounded judgment. Collect evidence first |
| Treating Verification as a formality | Verification is the most critical stage; run the checklist exhaustively |
| Issuing a finding without a `[Urgency] [Direction]` label | Every finding ends with *"Therefore, **[Urgency Direction]** has been identified / observed / recorded."* |
| Issuing a finding without a confidence level | Append *"Confidence: [level]"* after the severity label |
| Setting confidence by source count only (ignoring quality) | Apply both breadth and quality dimensions; walk the weakest-link rule |
| Skipping per-claim confidence in Evidence | Annotate each load-bearing claim with High / Medium / Low + a source-quality note; without this, the weakest-link rule has no chain to walk |
| Issuing a Type 1 recommendation with Low confidence | For irreversible decisions, refuse to recommend until confidence is Medium or High |
| Skipping the bias check | Run the 5-question check before issuing the Conclusion |
| Writing recommendations outside the "should/must" form | Use "should be done," "must be ensured" — concrete and directive |

## Theoretical Foundations

Each component of this skill traces to established work in epistemology, decision theory, and cognitive science. The skill itself is an applied synthesis encoded as a procedural checklist for AI agents — it is not a peer-reviewed work, but every component has a documented source.

| Component | Foundation |
|-----------|------------|
| 5-stage loop (Obs → Hyp → Ev → Conc → Ver) | Popper, *The Logic of Scientific Discovery* (1934) — falsificationism; Deming / PDCA cycle (1950) |
| Loop-back on failure | Popper — conjecture and refutation; Lewin, *Action Research* (1946); Schön, *The Reflective Practitioner* (1983) |
| Verification = what would falsify this | Popper — falsifiability as the criterion that distinguishes science from non-science |
| ≥ 2 hypotheses per observation | Feyerabend, *Against Method* (1975) — methodological pluralism; Toulmin, *The Uses of Argument* (1958) |
| Per-claim evidence annotation | Toulmin — grounds / warrant / backing / qualifier; Goldman, "What is Justified Belief?" (1979) — reliabilism |
| Weakest-link confidence rule | Argüman zincirleme mantığı; Sosa — safety condition for knowledge |
| Severity (urgency × direction) | CVSS (Common Vulnerability Scoring System); FMEA — risk = likelihood × impact |
| Confidence (breadth × quality) | Bayesian belief updating; source criticism in historiography |
| Calibrating rigor by reversibility | Bezos, 2015 shareholder letter — Type 1 / Type 2 decisions; Myers & Majd — real options theory (1990) |
| Adversarial verification | Red team / devil's advocate methodology; diyalojik akıl yürütme |
| Cognitive bias check (5 biases) | Kahneman & Tversky, heuristics-and-biases program (1974–); Stanovich & West, dual-process theory (2000); Kahneman, *Thinking, Fast and Slow* (2011) |
| Anti-rationalization safeguards | Cialdini, *Influence* (1984) — persuasion principles; Kahneman — WYSIATI ("what you see is all there is") |
| "Violating the letter violates the spirit" | Kant — deontological ethics; rule-based ethical formalism |

## Quick Reference

| Situation | Action |
|-----------|--------|
| Single document, claim, or decision to analyze | Run the full 5-stage loop |
| Complex multi-step decision | Observation → Hypothesis → Evidence → Conclusion → Verification |
| Irreversible / one-way-door decision | Full loop + bias check + Medium-or-High confidence required |
| Trivial / fully reversible decision | Observation + Conclusion may suffice; document the path |
| Quality control of a prior analysis | Run Verification against the rule set in scope |
| Need to assign a severity/urgency label | Pick urgency (Critical / Major / Minor / Note) and direction (Gap / Strength); combine in bold |
| Need to assign a confidence level | Apply breadth × quality; check the weakest-link rule |
| Need to detect cognitive bias | Run the 5-question bias check |
| Step-by-step tool available | One tool call per stage with the stage as the marker |
| No step-by-step tool available | Section headers or numbered list; one stage per unit |
| Red Flag triggered | Cancel the loop, restart from Observation |
| Verification failed | Return to Observation, run the loop again with new evidence |
| No domain rules file available | Construct the rule set from the user's stated requirements at the start |
| Unstated assumptions to audit | Run the Assumption Audit; verify testable ones in Evidence |

---
> Source: [hyuce/master-brain](https://github.com/hyuce/master-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
