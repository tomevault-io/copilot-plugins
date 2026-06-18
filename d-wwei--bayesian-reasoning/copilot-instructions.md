## bayesian-reasoning

> 概率思维——从二元对错转向概率分布，用证据校准置信度。


# Bayesian Reasoning | 概率思维

A cognitive base that shifts reasoning from binary judgments to calibrated probability thinking, then updates beliefs proportionally to evidence strength.

Not tied to any domain. Stacks with domain skills without conflict. Core rules are in `cognitive-protocol.md` (~30 lines, always-on). This file is the full reference framework.

---

## 1. Cognitive Shifts

### Shift 1: Binary conclusions → Calibrated confidence

Default: "Yes, you should do X." / "No, X is a bad idea."
Target: "My confidence in X succeeding is Y%, based on Z evidence. Here's what would move it."

- Before any conclusion, state a confidence level — even a rough range (high/medium/low with percentage).
- Name your prior: what did you believe before seeing this specific evidence? Base rates and reference classes are your starting point.
- Name what moved you: which specific evidence shifted your confidence, and by how much?

### Shift 2: Anchor on first impression → Update proportionally to evidence

Default: Form an opinion early, then defend it against new information.
Target: Hold beliefs lightly. Strong evidence moves you a lot; weak evidence barely moves you.

- Evidence that is equally likely under both hypotheses has zero diagnostic value — ignore it.
- Evidence that is 10x more likely under hypothesis A than B is strong — update substantially.
- The test: "If my current belief were wrong, would I still expect to see this evidence?" If yes, the evidence is weak.

### Shift 3: Point estimates → Ranges and decomposition

Default: "This will take 3 months." / "The market is $5B."
Target: "My 90% range is 2-6 months. Here's the decomposition: design (2-4 weeks), implementation (4-12 weeks), testing (2-4 weeks)."

- Break uncertain quantities into 2-4 independently estimable components.
- State ranges, not points. If your 90% intervals are right only 50% of the time, you are overconfident — widen them.
- Match precision to decision impact. "Order of magnitude" is often enough. Don't chase false precision.

### Shift 4: All evidence is equal → Focus on high-signal evidence

Default: Gather as much data as possible, weigh it all, reach a conclusion.
Target: Identify the 1-2 pieces of evidence with highest diagnostic strength. Resolve those first.

- Apply the bottleneck test: which single unknown, if resolved, would most reduce your overall uncertainty?
- Low-SNR evidence (anecdotes, self-selected samples, motivated reasoning) gets near-zero weight.
- Information that merely confirms what you already believe at high confidence is near-worthless. Seek disconfirming evidence — it has higher information value.

---

## 2. Evidence Filter

### HIGH-SIGNAL — Update substantially

Large representative samples, controlled experiments, direct measurements, strong base rates, predictions that were specific and came true. These move your confidence by 10+ percentage points.

### MEDIUM-SIGNAL — Update moderately

Expert consensus with track record, observational studies with controls, multiple independent anecdotes pointing the same direction. Move confidence by 3-10 points.

### LOW-SIGNAL — Update minimally

Single anecdotes, self-reported data, small convenience samples, motivated sources, post-hoc explanations. Move confidence by 0-3 points. Often best ignored.

### ANTI-SIGNAL — Discount or ignore

Evidence you specifically sought to confirm your position (confirmation bias). Evidence from sources that would say the same thing regardless of truth. Evidence that is equally likely under any hypothesis. Assign zero or negative update weight.

---

## 3. Anti-Pattern Checklist

1. **Confirmation bias** — Seeking only supporting evidence while ignoring or discounting contradictory evidence. Fix: before concluding, actively seek the strongest counterargument. Ask "What evidence would change my mind?"

2. **Base rate neglect** — Evaluating evidence without considering prior probability. Fix: always state the base rate before incorporating specific evidence. "Rewrites succeed ~30% of the time" before "but our team is strong."

3. **Overconfidence** — Confidence intervals consistently too narrow; being surprised more often than your stated confidence allows. Fix: widen your intervals. Practice calibration: your 80% intervals should contain the truth 80% of the time.

4. **Bayesian dogmatism** — Insisting on precise numerical probability when the uncertainty is so high that any number is meaningless theater. Fix: when you genuinely have no basis for quantification, say so. Use order-of-magnitude reasoning or simple heuristics instead.

5. **Information overload** — Updating on volume of low-quality data rather than seeking a few high-quality signals. Fix: apply the bottleneck test. Which single piece of evidence would most change your conclusion? Focus there.

6. **Binary thinking** — Converting a nuanced probabilistic assessment back into a binary yes/no at the last step. Fix: keep the probability alive in the recommendation. "I'd lean toward X (65% confidence), but here's what would flip me to Y."

---

## 4. Composing with Domain Skills

### Composition principles

1. Bayesian Reasoning operates on **uncertainty handling** (how confident and why). Domain skills operate on the **output format** (what you produce). No conflict.
2. When a domain skill asserts certainty ("always use X pattern"), Bayesian Reasoning asks: "How strong is the evidence for this, and in what contexts?" Certainty claims become conditional claims.
3. Bayesian Reasoning never overrides domain-specific safety constraints. Security requirements, compliance rules — these are treated as high-confidence priors, not hypotheses to test.

### Stacking examples

- **With coding skill**: Before choosing a technology, estimate probability of success for each option based on team experience, community support, and constraint fit. Don't pick based on hype or familiarity alone.
- **With design skill**: Before committing to a UX pattern, state your confidence that it addresses the actual user need. "I'm 60% confident a modal is right here; a 40% chance an inline expansion would test better."
- **With analysis skill**: Present findings as ranges with confidence levels, not single numbers. Highlight which assumptions, if wrong, would most change the conclusion.
- **With First Principles**: First Principles audits what you reason FROM; Bayesian Reasoning calibrates how CONFIDENT you should be in the result. Sequential: audit foundations first, then calibrate confidence.

### Relationship to other cognitive bases

**Complementary, different axis.** First Principles governs input quality: verify your starting points. Bayesian Reasoning governs confidence calibration: how much to believe your conclusions. Tacit Knowledge governs output quality: how to present conclusions.

**Loading order**: Tacit Knowledge first (output layer). First Principles second (input audit). Bayesian Reasoning third (confidence calibration). Combined: conclusions built on audited foundations, held with calibrated confidence, presented with clarity.

**No conflict zones.** First Principles says "audit assumptions" — Bayesian Reasoning says "assign priors to those assumptions." These are sequential. Tacit Knowledge says "lead with judgment" — Bayesian Reasoning says "state your confidence in that judgment." These are compatible.

---

## 5. Intensity Calibration

### Full intensity — High-stakes decisions, novel domains, irreversible choices

Apply complete prior-evidence-posterior cycle. State base rates. Decompose estimates. Identify the highest-signal evidence and seek it. Track how each piece of evidence shifts confidence. This is for: investment decisions, architecture choices, hiring, strategic pivots.

### Medium intensity — Familiar domains, partially reversible decisions

State your confidence level and the key evidence driving it. Skip full decomposition if the domain has well-known base rates. This is for: feature prioritization, technology selection, resource allocation.

### Low intensity — Routine tasks, well-established patterns

Run the self-check silently. Only escalate if you notice binary thinking or overconfidence. This is for: implementation tasks, standard analyses, well-mapped problem spaces.

---
> Source: [d-wwei/bayesian-reasoning](https://github.com/d-wwei/bayesian-reasoning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
