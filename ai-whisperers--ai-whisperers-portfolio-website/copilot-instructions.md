## ai-whisperers-portfolio-website

> Guidelines for identifying when a user story should be split into smaller, more manageable stories


# Story Splitting Rule

## Purpose & Scope

This rule defines criteria and patterns for recognizing when a user story is too large and should be split into multiple smaller stories. Proper story splitting enables better manageability, risk reduction, delivery predictability, and team velocity. It helps teams deliver value incrementally and maintain sustainable sprint commitments.

**Applies to**: All User Stories during backlog grooming, sprint planning, or mid-sprint refinement where story size, complexity, or risk is assessed.

**Does not apply to**: Epics (already designed to be split into Features/Stories), appropriately-sized stories (≤8 points), trivial tasks (<1 point), or stories that would lose cohesion if split further.

## When to Consider Story Splitting

### Size Indicators (Any of These)
- **Story Points > 8**: Stories larger than 8 points have high uncertainty and risk
- **Multiple Epics Worth**: Story spans scope that could justify multiple user stories
- **Cannot Complete in One Sprint**: Story cannot be fully implemented and tested within iteration timeframe
- **Multiple Major Components**: Story touches 3+ major components/systems
- **Conversation Explosion**: During planning, conversations balloon with "and also we need..." repeatedly

### Complexity Indicators (Any of These)
- **Multiple Critical Paths**: Story has several independent critical paths that could be delivered separately
- **Unclear Requirements**: After initial analysis, requirements remain vague or require extensive discovery
- **High Risk Areas**: Story contains multiple high-risk areas that should be tackled incrementally
- **Multiple Acceptance Criteria Sets**: Acceptance criteria naturally group into distinct functional areas
- **Implementation Spans Multiple Layers**: Frontend + Backend + Database + Integration all in one story

### Dependency Indicators (Any of These)
- **Sequential Dependencies**: Story has clear sequential phases that could be independent stories
- **Blocking Multiple Teams**: Story blocks work for multiple teams/developers who could work in parallel
- **External Dependencies**: Story depends on multiple external systems/teams with different timelines
- **Infrastructure + Feature**: Story mixes infrastructure setup with feature delivery

### Risk Indicators (Any of These)
- **Unknown Unknowns**: Significant technical uncertainty that needs investigation before commitment
- **Multiple Failure Modes**: Too many ways the story could fail or need rework
- **All-or-Nothing Delivery**: No way to deliver incremental value if story fails mid-implementation
- **Testing Complexity**: Testing strategy is unclear or requires extensive test scenario development

## Story Splitting Patterns

### Pattern 1: By User Journey Steps
Split based on user workflow stages:
- **Before**: "User can complete entire purchase flow"
- **After Split**:
  - Story 1: User can add items to cart
  - Story 2: User can proceed to checkout
  - Story 3: User can complete payment
  - Story 4: User receives order confirmation

### Pattern 2: By Technical Layer
Split based on system layers:
- **Before**: "Implement user authentication"
- **After Split**:
  - Story 1: Implement authentication data model and database
  - Story 2: Implement authentication API endpoints
  - Story 3: Implement authentication UI components
  - Story 4: Integrate authentication with existing features

### Pattern 3: By Business Rule Complexity
Split based on business rule complexity:
- **Before**: "Implement pricing calculation with all rules"
- **After Split**:
  - Story 1: Implement base pricing calculation
  - Story 2: Add volume discount rules
  - Story 3: Add promotional pricing rules
  - Story 4: Add customer-specific pricing rules

### Pattern 4: By Happy Path vs Edge Cases
Split based on scenario complexity:
- **Before**: "Implement file upload with all scenarios"
- **After Split**:
  - Story 1: Implement basic file upload (happy path)
  - Story 2: Handle file validation errors
  - Story 3: Handle large file uploads
  - Story 4: Handle network interruption recovery

### Pattern 5: By Infrastructure vs Feature
Split setup work from feature delivery:
- **Before**: "Implement notification system"
- **After Split**:
  - Story 1: Setup notification infrastructure and templates
  - Story 2: Implement email notifications
  - Story 3: Implement SMS notifications
  - Story 4: Implement in-app notifications

### Pattern 6: By Integration Points
Split based on external dependencies:
- **Before**: "Integrate with payment providers"
- **After Split**:
  - Story 1: Integrate with primary payment provider
  - Story 2: Integrate with backup payment provider
  - Story 3: Implement payment provider fallback logic
  - Story 4: Add payment provider monitoring

### Pattern 7: By Validation Phases
Split based on validation complexity:
- **Before**: "Implement temporary structure builder with validation"
- **After Split**:
  - Story 1: Design and implement bottom-up structure builder
  - Story 2: Implement LAYER field marker logic
  - Story 3: Implement atomic topnode swap mechanism
  - Story 4: Implement structure validation and cleanup

## Splitting Decision Framework

### Step 1: Initial Assessment
Ask these questions:
1. Can I clearly articulate all acceptance criteria in one sitting?
2. Do I understand all technical dependencies?
3. Can this be implemented AND tested in one sprint?
4. If something fails, can we still deliver partial value?
5. Does the team agree this is reasonably sized?

**If any answer is "No" → Consider splitting**

### Step 2: Identify Natural Boundaries
Look for natural split points:
- Distinct user workflows
- Independent technical components
- Separate data models
- Different integration points
- Sequential implementation phases
- Separate testing strategies

### Step 3: Validate Split Value
For each potential split, verify:
- ✅ Each resulting story delivers standalone value
- ✅ Each story has clear, testable acceptance criteria
- ✅ Each story can be independently deployed (or clearly marked as dependency)
- ✅ Split reduces risk and improves predictability
- ✅ Split enables parallel work or incremental delivery

### Step 4: Check for Over-Splitting
Avoid splitting too small:
- ❌ Stories that are trivial (< 1 point)
- ❌ Stories that create excessive coordination overhead
- ❌ Stories that cannot deliver value independently
- ❌ Stories that fragment cohesive functionality unnecessarily

## Implementation Guidance

### For Product Owners
When splitting stories:
1. Ensure each story delivers measurable business value
2. Prioritize stories based on risk and value
3. Consider dependencies between split stories
4. Communicate split rationale to stakeholders
5. Update story hierarchy and traceability

### For Developers
When requesting story splits:
1. Clearly articulate why split is needed
2. Propose specific split boundaries
3. Estimate each resulting story
4. Identify dependencies between splits
5. Highlight risk reduction benefits

### For Scrum Masters
When facilitating splits:
1. Ensure team consensus on split necessity
2. Facilitate split pattern discussion
3. Help identify natural boundaries
4. Ensure traceability is maintained
5. Update velocity tracking accordingly

## Red Flags: When NOT to Split

### Don't Split If...
- **Artificial Boundaries**: Split would create artificial boundaries with no value distinction
- **Excessive Coordination**: Split creates more coordination overhead than value
- **Cohesive Functionality**: Functionality is tightly cohesive and splitting breaks understanding
- **Already Right-Sized**: Story is genuinely simple but touches multiple areas
- **Split for Velocity Gaming**: Split only to make velocity look better

## Success Criteria

### Well-Split Stories Should
- ✅ Each story deliverable in < 1 week
- ✅ Each story independently testable
- ✅ Each story provides clear value
- ✅ Dependencies between stories are explicit
- ✅ Team confidence increases after split
- ✅ Risk is reduced through incremental delivery

### Warning Signs of Poor Split
- ❌ Stories depend on each other in complex ways
- ❌ No story delivers value independently
- ❌ Split creates confusion rather than clarity
- ❌ Testing requires all stories to be complete
- ❌ Team spends more time coordinating than delivering

## Splitting Examples

### Example 1: Complex Infrastructure Change
**Original Large Story**: "Implement new data processing pipeline with validation"
- Story Points: Would be 21+ (too large)
- Multiple critical paths
- High technical complexity
- Multiple validation phases

**Good Split**:
- Story 1 (5 pts): Design and implement core pipeline builder
- Story 2 (3 pts): Implement data marker and state logic
- Story 3 (5 pts): Implement atomic state transition mechanism
- Story 4 (3 pts): Implement old data cleanup logic
- Story 5 (5 pts): Enhance validation with multi-phase checks

**Value**: Each story delivers incremental capability, reduces risk, enables parallel work

### Example 2: Feature with Multiple Concerns
**Original Large Story**: "Implement complete validation and transition mechanism"
- Too many concerns in one story
- Mixed infrastructure and business logic
- Unclear testing strategy

**Good Split**:
- Story 1: Build temporary data structure (infrastructure)
- Story 2: Validate data structure (business rules)
- Story 3: Implement atomic transition (critical transaction)
- Story 4: Implement cleanup (housekeeping)

**Value**: Clear separation of concerns, independent testing, reduced risk

## Integration with Other Rules

### Complexity Assessment
- Stories marked as "Complex Implementation" in complexity assessment are prime candidates for splitting
- Use complexity-assessment-rule.mdc framework to evaluate split necessity

### Planning
- Split stories during planning phase before commitment
- Update plan.md to reflect split decisions and rationale
- Document dependencies between split stories

### Estimation
- Re-estimate all split stories
- Validate total effort is reasonable
- Consider coordination overhead in estimates

## Inputs (Contract)

- User story being evaluated for splitting
- Story points estimate or estimation uncertainty indication
- List of acceptance criteria
- Technical complexity assessment
- Team feedback on story size/feasibility
- Sprint capacity and timeline constraints

## Outputs (Contract)

- Splitting decision: Split or Keep Intact
- If split: Set of smaller user stories (typically 2-5 stories)
- Split pattern used (User Journey, Technical Layer, Business Rules, etc.)
- Each resulting story follows `rule.agile.user-story-documentation.v1` format
- Split rationale documented
- Dependencies between split stories identified
- Original story updated with references to split stories or marked as superseded

## Deterministic Steps

1. Read user story and collect story points, acceptance criteria, complexity indicators
2. Apply Splitting Decision Checklist (see below)
3. Count checklist indicators present
4. If <3 indicators: Keep story intact, proceed with original story
5. If 3-4 indicators: Flag as "Strong candidate for splitting", recommend review
6. If 5+ indicators: MUST split before commitment
7. If splitting required: Identify natural split boundaries using patterns (User Journey, Technical Layer, Business Rules, Happy Path vs Edge Cases, Infrastructure vs Feature, Integration Points, Validation Phases)
8. Apply selected splitting pattern(s)
9. Create 2-5 new user stories from split
10. Validate each resulting story:
    - Delivers standalone value
    - Has clear, testable acceptance criteria
    - Can be independently deployed or clearly marked as dependency
    - Reduces risk and improves predictability
11. Document split rationale and pattern used
12. Update original story with split references or mark as superseded
13. Establish dependencies between split stories if sequential

## Formatting Requirements

### Splitting Decision Documentation

When documenting a split decision, include:

```markdown
## Split Decision

**Original Story**: [STORY-ID]
**Decision**: Split into [N] stories
**Pattern Used**: [Pattern Name]
**Rationale**: [Why split was necessary]

### Resulting Stories

- [STORY-ID-1]: [Title] - [Focus]
- [STORY-ID-2]: [Title] - [Focus]
- [STORY-ID-N]: [Title] - [Focus]

### Dependencies

- [STORY-ID-1] must complete before [STORY-ID-2]
- [STORY-ID-2] and [STORY-ID-3] can be done in parallel
```

### Split Story References

In original story (if preserved):

```markdown
## Split History

This story was split into smaller stories on [DATE]:
- [STORY-ID-1](link)
- [STORY-ID-2](link)
- [STORY-ID-3](link)

**Status**: Superseded by split stories above
```

## OPSEC and Leak Control

- NO internal system names if story splitting reveals sensitive architecture
- NO customer names or identifying information in split examples
- NO actual story IDs from production if they reveal sensitive information
- Use generic placeholders in examples
- Redact organization-specific split rationales if sensitive

## Integration Points

### Related Rules
- `rule.agile.user-story-documentation.v1` - Each split story must follow documentation rule
- `rule.agile.business-feature-documentation.v1` - Feature may need update when stories split
- `rule.agile.epic-documentation.v1` - Epic may need story count update
- `rule.ticket.complexity-assessment.v1` - Complex stories are split candidates
- `rule.ticket.plan.update.v1` - Split decisions inform implementation planning

### Workflow Integration
- Story splitting typically occurs during backlog grooming
- Split stories may span multiple sprints
- Velocity calculations must account for split stories
- Sprint commitments may change if stories split mid-sprint

## Failure Modes and Recovery

**Over-Splitting (Stories Too Small)**:
- Detection: Resulting stories <1 point or trivial
- Recovery: Merge back into larger story, use different split pattern

**Loss of Cohesion**:
- Detection: Split stories don't deliver independent value
- Recovery: Regroup stories, use different split boundary

**Excessive Coordination Overhead**:
- Detection: Split creates more coordination than value
- Recovery: Keep story intact, manage complexity differently

**Missing Dependencies**:
- Detection: Split stories fail due to unidentified dependencies
- Recovery: Document dependencies, establish execution order

**Unclear Value Delivery**:
- Detection: No single split story delivers testable value
- Recovery: Revisit split pattern, ensure each story has clear value

## Provenance Footer Specification

When documenting split decisions in a separate file:

```
---
Produced-by: rule.agile.story-splitting.v1 | ts={{timestamp}} | original-story={{story-id}}
```

When updating original story with split references:

```
---
Updated: {{timestamp}} by rule.agile.story-splitting.v1 | split-into={{count}} stories
```

## Splitting Decision Checklist

Use this checklist during story grooming/planning:

- [ ] Story points > 8 or estimation very uncertain
- [ ] Story touches 3+ major components/systems  
- [ ] Cannot be completed and tested in one sprint
- [ ] Multiple critical paths that could be delivered separately
- [ ] Acceptance criteria naturally group into distinct areas
- [ ] High technical uncertainty requiring investigation
- [ ] Multiple failure modes or risk areas
- [ ] No way to deliver incremental value if story fails
- [ ] Team expresses concern about story size/complexity
- [ ] Planning conversation explodes with "and also we need..."

**If 3-4 boxes checked → Strong candidate for splitting**
**If 5+ boxes checked → MUST split before commitment**

## FINAL MUST-PASS CHECKLIST

- [ ] Splitting decision based on objective criteria (checklist with 3+ indicators)
- [ ] Split pattern identified and documented (User Journey, Technical Layer, Business Rules, etc.)
- [ ] Each resulting story delivers standalone value
- [ ] Each resulting story has clear, testable acceptance criteria
- [ ] Each resulting story independently deployable or dependencies explicitly documented
- [ ] Total story count after split is reasonable (2-5 stories typically)
- [ ] Original story updated with split references or marked as superseded
- [ ] Split rationale documented for team understanding
- [ ] No over-splitting (no trivial <1 point stories)
- [ ] Each split story follows `rule.agile.user-story-documentation.v1` format

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ai-Whisperers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
