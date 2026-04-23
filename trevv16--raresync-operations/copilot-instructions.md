## raresync-operations

> Defines validation board methodology and workflow - how validation boards work, how to structure hypotheses and tests, how to update metrics, track validation results, and use validated facts to inform business decisions


# Validation Board Methodology & Workflow

## Purpose
This rule defines how validation boards work using the Lean Startup Machine methodology. It explains the validation board process, how to structure hypotheses and tests, how to update metrics, track validation results, record customer discovery results, and use validated facts to inform business decisions.

## Required References

### Before Updating Validation Boards
**Always read these files first:**
1. `R&D/user-research/Problem: Unique Inventory/Start Pivot/Pivot Overview.md` - Main validation board
2. `R&D/user-research/Problem: Unique Inventory/Validation Board.md` - Validation tracking
3. `R&D/Validation Target Metrics.md` - Validation framework (Phase 1-4, gates, metrics)
4. `R&D/Customer Development Lifecycle.md` - Customer development stages (Discovery, Validation, Creation)
5. `R&D/user-research/Problem: Unique Inventory/Pivot Template.md` - Template structure reference
6. `R&D/user-research/Problem: Unique Inventory/Problem Overview.md` - Problem context
7. `.cursor/rules/raresync-company-context.mdc` - Company and customer context

## Core Methodology

### The Validation Board Process

**Validation boards turn hypotheses into facts through structured testing.**

The process:
1. **State Hypotheses** - What you believe about customers, problems, and solutions
2. **Design Tests** - Create experiments to validate or invalidate hypotheses
3. **Run Tests** - Execute experiments and collect data
4. **Document Results** - Record what actually happened
5. **Turn Facts into Decisions** - Use validated facts to inform business strategy

### Key Principle

> **Facts exist outside the building.** Your opinions don't matter, customer behavior does.

Validation boards force you to:
- Test assumptions before building
- Make decisions based on evidence, not guesses
- Fail fast and cheaply
- Pivot when hypotheses are invalidated

## Validation Board Structure

### Core Sections (from Pivot Overview.md and Pivot Template.md)
**Validation boards should include:**

1. **Core Hypotheses** - Customer, Problem, Solution hypotheses
2. **Core Assumptions** - List of assumptions being tested
3. **Riskiest Assumption** - The assumption that's most critical to validate
4. **Method** - How assumptions will be tested
5. **Minimum Success Criteria** - Specific metrics and thresholds
6. **Current Performance** - Actual results and metrics
7. **Go/No-Go Criteria** - Decision framework
8. **Next Steps Based on Results** - Actions based on validation outcomes

**Reference:** `R&D/user-research/Problem: Unique Inventory/Pivot Template.md` for complete template structure

### Validation Framework Alignment
**When updating validation boards:**
- Reference Validation Target Metrics.md for framework structure
- Align with Phase 1-4 validation phases when applicable
- Use consistent metric definitions
- Note which phase/gate is being validated
- Reference Customer Development Lifecycle.md to understand which lifecycle stage you're in

**Validation Phases (from Validation Target Metrics.md):**
- **Phase 1: Monopoly Opportunity Validation** - GO/NO-GO decision
  - Core Problem Validation (must hit ALL 4 metrics)
  - Monopoly Potential (must hit 3 of 4)
  - The Secret Test (must have ONE strong answer)
  - Power Law Potential (must hit 2 of 3)
  - Founder-Market Fit (must hit 2 of 3)
  - **GATE 1: Hit 13/15 metrics or STOP**
  - **Time allocation: Spend 80% of validation time here**

- **Phase 2: Solution Validation** - Define WHAT to build
  - MVP Scope Test
  - Build vs. Buy vs. Partner
  - Technical Moat Definition
  - **GATE 2: Clear solution with defensibility**

- **Phase 3: Market Entry Validation** - Define HOW to dominate
  - Distribution Strategy
  - Pricing Power Test
  - Competition Preemption
  - **GATE 3: Clear path to niche dominance**

- **Phase 4: Execution Validation** - Confirm you can build
  - Resource Reality Check
  - Speed to Monopoly
  - Expansion Roadmap
  - **GATE 4: Realistic execution plan**

**Customer Development Lifecycle Alignment (from Customer Development Lifecycle.md):**
- **Stage 1: Customer Discovery** - Maps to Phase 1 validation
- **Stage 2: Customer Validation** - Maps to Phase 2-3 validation
- **Stage 3: Customer Creation** - Maps to Phase 4 validation and beyond

## How to Structure Validation Tests

### Test Design Principles

**Good tests:**
- ✅ Test one assumption at a time
- ✅ Have clear success/failure criteria
- ✅ Are cheap and fast to run
- ✅ Produce measurable results
- ✅ Test with real customers (not friends/family unless they're the target)

**Bad tests:**
- ❌ Test multiple assumptions simultaneously
- ❌ Have vague success criteria
- ❌ Are expensive or time-consuming
- ❌ Produce subjective results
- ❌ Test with wrong audience

### Test Methods

**Customer Discovery Methods:**
- Reddit research (natural questions, pattern identification)
- Problem interviews (understand pain points, quantify costs)
- Workflow observation (shadow customers, see actual process)

**Customer Validation Methods:**
- Solution interviews (show mockups, gauge excitement)
- Pricing tests (ask about willingness to pay, test price points)
- Prototype demos (show working prototype, measure engagement)
- Beta testing (real usage, retention metrics)

**When to Use Each Method:**
- **Reddit:** Early discovery, validate problem exists, identify patterns
- **Interviews:** Validate specific assumptions, understand workflows, test pricing
- **Prototypes:** Validate solution, test features, measure engagement
- **Beta:** Validate retention, usage patterns, willingness to pay ongoing

## Updating Metrics

### Current Performance Section
**When updating current performance:**
- Use specific counts and percentages
- Update both quantitative and qualitative metrics
- Maintain format from Pivot Overview.md
- Group by validation method (Reddit, Interviews, etc.)

**Format:**
```markdown
## Current Performance

**Customer Discovery (Reddit):**
- Reddit posts made: [Number]
- Total comments received: [Number]
- % validating inventory management is painful: [X]%
- % mentioning cataloging as pain: [X]%
- % mentioning cascade symptoms: [X]%

**Customer Validation (Interviews):**
- Interviews completed: [X]/[Y]
- Identify cataloging as top 2 pain: [X]/[Y] ([Z]%)
- Acknowledge cascade problems: [X]/[Y] ([Z]%)
- Would pay $99/month: [X]/[Y] ([Z]%)
```

### Minimum Success Criteria Assessment
**When assessing against success criteria:**
- Check each criterion against actual results
- Mark criteria as met/not met
- Note if threshold is close but not met
- Document any criteria that need adjustment

**Format:**
```markdown
### Minimum Success Criteria

**Must hit ALL (Core Problem Validation):**
- [X] [Criterion 1] - [Status: Met/Not Met/Close]
- [X] [Criterion 2] - [Status: Met/Not Met/Close]

**Must hit 3 of 4 (Monopoly Potential):**
- [X] [Criterion 1] - [Status]
- [X] [Criterion 2] - [Status]
- [X] [Criterion 3] - [Status]
- [X] [Criterion 4] - [Status]
```

### Go/No-Go Criteria Updates
**When updating Go/No-Go criteria:**
- Check each green flag/red flag against results
- Mark criteria as met/not met
- Note pivot signals if applicable
- Update based on actual validation outcomes

**Format:**
```markdown
## Go/No-Go Criteria

**GREEN FLAGS (Build AI-powered inventory for unique products):**
- [X] Lighthouse customer enthusiastically commits to $99/month
- [ ] 7+ retailers identify cataloging as top pain
- [ ] 70%+ would pay $99/month

**RED FLAGS (Pivot to different problem/customer):**
- [ ] Lighthouse customer won't commit at $99/month
- [ ] <7 out of 10 identify cataloging as top pain
- [ ] Nobody will pay above $50/month
```

## Recording Customer Discovery Results

### Reddit Discovery
**When documenting Reddit results:**
- Count total posts and comments
- Identify patterns in responses
- Calculate percentages for key metrics
- Note cascade symptoms mentioned
- Document which pain points emerged

**Format:**
```markdown
## Reddit Discovery Results

**Posts Made:** [Number] across [Subreddits]
**Total Comments:** [Number]

### Quantitative Results
- Comments validating inventory management is painful: [X] ([Y]%)
- Comments mentioning cataloging as pain: [X] ([Y]%)
- Comments mentioning cascade symptoms: [X] ([Y]%)

### Qualitative Patterns
- [Pattern 1]: [Description]
- [Pattern 2]: [Description]
```

### Interview Results
**When documenting interview results:**
- Update interview completion count
- Record quantitative metrics for each interview
- Aggregate metrics across all interviews
- Document qualitative patterns
- Note which assumptions were validated/invalidated

**Format:**
```markdown
## Interview Results Summary

**Interviews Completed:** [X]/[Y]

### Aggregate Quantitative Results
- Identify cataloging as top 2 pain: [X]/[Y] ([Z]%)
- Acknowledge cascade problems: [X]/[Y] ([Z]%)
- Would pay $99/month: [X]/[Y] ([Z]%)
- [Other metrics...]

### Qualitative Patterns
- [Pattern 1]: [Description with examples]
- [Pattern 2]: [Description with examples]

### Assumptions Validated
- [X] [Assumption 1]
- [X] [Assumption 2]

### Assumptions Invalidated
- [X] [Assumption 1]
- [X] [Assumption 2]
```

## Maintaining Validation Structure

### Hypothesis Updates
**When hypotheses change:**
- Document what changed and why
- Update all sections that reference the hypothesis
- Note if change is based on validation results
- Update success criteria if hypothesis changed

### Assumption Tracking
**When assumptions are validated/invalidated:**
- Mark assumption as validated/invalidated
- Document evidence for validation/invalidation
- Note impact on other assumptions
- Update next steps based on results

### Method Updates
**When validation method changes:**
- Document why method changed
- Update method section with new approach
- Adjust success criteria if method changed
- Note any impact on existing results

## Workflow Guidelines

### Before Updating Validation Board
1. Read current Pivot Overview.md fully
2. Review Pivot Template.md for structure reference
3. Review Validation Target Metrics.md for Phase 1-4 framework
4. Review Customer Development Lifecycle.md to understand current lifecycle stage
5. Understand current validation status
6. Identify what needs updating

### During Validation Board Update
1. Update Current Performance section with new data
2. Assess against Minimum Success Criteria
3. Update Go/No-Go Criteria checkboxes
4. Document qualitative patterns
5. Note which assumptions validated/invalidated
6. Update Next Steps Based on Results

### After Validation Board Update
1. Verify metrics are consistent across sections
2. Check that next steps align with results
3. Ensure cross-references are accurate
4. Note any significant changes or insights

## Quality Standards

### Validation Board Quality
- [ ] All metrics updated with latest data
- [ ] Success criteria assessed against results
- [ ] Go/No-Go criteria updated
- [ ] Qualitative patterns documented
- [ ] Assumptions marked as validated/invalidated
- [ ] Next steps align with validation outcomes
- [ ] Metrics are consistent across sections

### Consistency Checks
- Metrics match interview/documentation results
- Success criteria thresholds are clear
- Go/No-Go criteria align with results
- Next steps are actionable based on outcomes
- Framework alignment with Validation Target Metrics.md

## Examples

### ❌ Don't Do This
**Validation Board Update:**
"Did some interviews. Most people liked the idea. Updated the board."

**Why this is wrong:**
- No specific metrics
- Doesn't update Current Performance section
- Doesn't assess against success criteria
- Doesn't update Go/No-Go criteria
- Not actionable

### ✅ Do This Instead
**Validation Board Update:**
"## Validation Board Update - 2025-01-15

**New Data:**
- Completed 3 additional interviews (now 4/10 total)
- All 3 identified cataloging as top pain point
- All 3 acknowledged cascade problems
- 2/3 would pay $99/month, 1/3 hesitant at $150/month

**Updated Current Performance:**
- Interviews completed: 4/10 (40%)
- Identify cataloging as top 2 pain: 4/4 (100%)
- Acknowledge cascade problems: 4/4 (100%)
- Would pay $99/month: 3/4 (75%)

**Success Criteria Assessment:**
- ✅ 7+ identify cataloging as top pain: 4/4 so far (need 3 more)
- ✅ 5+ acknowledge cascade problems: 4/4 (met)
- ⚠️ 70%+ would pay $99/month: 75% (met, but small sample)

**Go/No-Go Criteria:**
- ✅ Lighthouse customer committed: Yes
- ⚠️ 7+ retailers identify cataloging: 4/4 so far (need 3 more)
- ✅ 70%+ would pay $99/month: 75% (met)

**Next Steps:**
- Continue interviews to reach 10 total
- If 7+ identify cataloging and 70%+ pay, proceed to build MVP
- Monitor pricing sensitivity (1/4 hesitant at $150/month)"

**Why this is better:**
- Specific metrics with counts and percentages
- Updates Current Performance section
- Assesses against success criteria
- Updates Go/No-Go criteria
- Documents next steps
- Actionable and clear

## Using Facts to Inform Business Decisions

### How Validated Facts Inform Strategy

**Once you have validated facts, use them to update:**

1. **Product Strategy**
   - `R&D/product/MVP Definition.md` - What to build based on validated features
   - `R&D/product/Evolution Plan.md` - Evolution path based on validated needs

2. **Pricing Strategy**
   - `R&D/pricing/Pricing Strategy.md` - Pricing based on validated willingness to pay
   - `R&D/pricing/Pricing Table.md` - Pricing structure based on validated value

3. **Marketing Strategy**
   - `Marketing/Content/Content Marketing Strategy.md` - Messaging based on validated pain points
   - `Marketing/Keyword Research.md` - Keywords based on validated customer language

4. **Customer Development**
   - `R&D/Customer Dev Lifecycle Status.md` - Progress based on validation gates
   - `R&D/user-research/Problem: Unique Inventory/Start Pivot/Pivot Overview.md` - Updated with new facts

### Decision Framework

**When hypothesis is VALIDATED:**
1. Document the fact in validation board
2. Update relevant strategy files with validated information
3. Proceed to next validation test or build phase
4. Use validated fact to inform next hypothesis

**When hypothesis is INVALIDATED:**
1. Document the invalidation in validation board
2. Identify what you learned (why was it wrong?)
3. Pivot hypothesis or test different assumption
4. Update strategy files if significant change needed

**When results are INCONCLUSIVE:**
1. Document what you learned (partial validation)
2. Identify what additional data is needed
3. Design follow-up test to get conclusive answer
4. Don't proceed until you have clear answer

### Example: Using Validated Facts

**Validated Fact:** "7+ out of 10 retailers identify cataloging as top pain point"

**Actions:**
1. ✅ Update MVP Definition to prioritize cataloging features
2. ✅ Update Content Marketing Strategy to lead with cataloging pain
3. ✅ Update Pricing Strategy (cataloging is core value, can price accordingly)
4. ✅ Proceed to solution validation (test if AI cataloging solves it)

**Invalidated Fact:** "Only 2 out of 10 identify cataloging as top pain"

**Actions:**
1. ❌ Don't build cataloging-focused solution
2. ✅ Pivot to whatever IS the top pain (stock counts, purchasing, etc.)
3. ✅ Update Problem Hypothesis
4. ✅ Update MVP Definition to focus on validated pain point

## Related Rules
- [R&D User Research](r-and-d-user-research.mdc) - User research documentation standards
- [RareSync Company Context](raresync-company-context.mdc) - Company and customer context
- [Document Maintenance](document-maintenance.mdc) - Documentation consistency
- [Reddit Research Methodology](reddit-research-methodology.mdc) - Reddit research process
- [Strategy File Integration](strategy-file-integration.mdc) - How validated facts update strategy files

## Key Documents to Reference
- `R&D/user-research/Problem: Unique Inventory/Start Pivot/Pivot Overview.md` - Main validation board
- `R&D/user-research/Problem: Unique Inventory/Validation Board.md` - Validation tracking
- `R&D/Validation Target Metrics.md` - Validation framework (Phase 1-4, gates, metrics, time allocation)
- `R&D/Customer Development Lifecycle.md` - Customer development stages (Discovery, Validation, Creation)
- `R&D/user-research/Problem: Unique Inventory/Pivot Template.md` - Template structure for pivot documentation
- `R&D/user-research/Problem: Unique Inventory/Problem Overview.md` - Problem context

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trevv16) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
