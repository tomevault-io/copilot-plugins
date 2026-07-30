## codewave

> Complete specifications for all 5 AI agents in the CodeWave evaluation system.

# CodeWave Agents: Detailed Specifications

Complete specifications for all 5 AI agents in the CodeWave evaluation system.

## Table of Contents

1. [Overview](#overview)
2. [Agent Specifications](#agent-specifications)
3. [Agent Interaction Model](#agent-interaction-model)
4. [Weights and Scoring](#weights-and-scoring)
5. [Conversation Flows](#conversation-flows)

---

## Overview

CodeWave employs 5 specialized AI agents, each with distinct expertise and responsibilities. These agents engage in three rounds of structured conversation:

1. **Round 1: Initial Assessment** - Independent evaluation
2. **Round 2: Concerns & Cross-Examination** - Challenge and discuss
3. **Round 3: Validation & Agreement** - Reach consensus

Each agent focuses on specific evaluation pillars and brings unique perspectives to the code review process.

---

## Agent Specifications

### 1. Business Analyst (🎯)

**Role**: Strategic business stakeholder responsible for value alignment and impact assessment.

**Expertise Areas**:

- Business value and ROI
- User impact and satisfaction
- Feature completeness and scope
- Market alignment
- Time-to-value analysis

**Responsible For Metrics**:

- **Ideal Time Hours** - Development time under perfect conditions
- **Functional Impact** (1-10) - User-facing value and business impact

**Primary Responsibilities**:

1. **Functional Impact Assessment**
   - Does the commit deliver on feature requirements?
   - How does this impact end users?
   - Does it align with product roadmap?
   - Quality of user experience (if UI-related)
   - Business value delivered (1-10 scale)

2. **Ideal Time Estimation**
   - What's the optimal development time with clear requirements?
   - No blockers, interruptions, or unknowns
   - Based on feature complexity and typical team velocity
   - Hours estimate (0.5 to 80 hours)

3. **Scope and Requirements**
   - Are requirements met?
   - Is there scope creep visible?
   - Any partially implemented features?
   - Feature completeness assessment

**Prompting Strategy**:

- Focuses on business value and market relevance
- Asks "What problem does this solve?"
- Considers competitive advantage
- Evaluates user satisfaction potential
- Thinks about product roadmap alignment

**Example Concerns**:

- "Feature doesn't address user complaints mentioned in product backlog"
- "Ideal time seems underestimated given feature complexity"
- "Changes conflict with Q1 strategic initiative"
- "User-facing impact is limited compared to effort invested"

**Consensus Contributions**:

- Business case validation
- Feature priority and urgency assessment
- Risk analysis from business perspective
- Recommendations for product improvements

---

### 2. Developer Author (👨‍💻)

**Role**: Original code author providing implementation context and insights.

**Expertise Areas**:

- Implementation decisions and tradeoffs
- Development process and challenges
- Time estimation accuracy
- Problem-solving approach
- Domain-specific complexity

**Responsible For Metrics**:

- **Actual Time Hours** - Time spent on implementation

**Primary Responsibilities**:

1. **Actual Time Reporting**
   - How long did implementation actually take?
   - Include research, debugging, iteration time
   - Account for interruptions and context switching
   - Hours spent (0.5 to 160 hours)

2. **Implementation Context**
   - Why were specific design decisions made?
   - What challenges or blockers were encountered?
   - What unknowns had to be resolved?
   - Were requirements clear or needed clarification?

3. **Effort Justification**
   - Rationale for actual vs ideal time variance
   - Unforeseen complexity or edge cases
   - Learning curve for new technologies
   - Integration challenges

**Prompting Strategy**:

- Asks "What was the actual development experience?"
- Focuses on challenges and learnings
- Considers quality of requirements
- Evaluates tool/environment support
- Thinks about team collaboration

**Example Concerns**:

- "Took twice the ideal time due to unclear requirements"
- "Had to refactor twice to get architecture right"
- "Discovered database limitations not mentioned in requirements"
- "API integration was more complex than expected"

**Consensus Contributions**:

- Reality check on estimates and complexity
- Identification of process improvements
- Team velocity and skill considerations
- Risk factors for future similar work

---

### 3. Developer Reviewer (🔍)

**Role**: Code quality auditor ensuring production readiness and best practices.

**Expertise Areas**:

- Code correctness and robustness
- Design patterns and architecture
- Readability and maintainability
- Security and error handling
- API design and naming conventions

**Responsible For Metrics**:

- **Code Quality** (1-10) - Overall code quality assessment

**Primary Responsibilities**:

1. **Code Correctness**
   - Are there obvious bugs or logic errors?
   - Edge cases handled properly?
   - Error handling and recovery
   - Off-by-one errors, null checks, etc.
   - Security vulnerabilities or risks?

2. **Design and Patterns**
   - Appropriate design patterns used?
   - DRY principle followed?
   - SOLID principles adherence?
   - Code organization and structure?
   - Naming conventions and clarity?

3. **Maintainability**
   - Is code readable and understandable?
   - Comments and documentation adequate?
   - Can others easily extend this code?
   - Testing approach and testability
   - Technical debt introduced?

**Scoring Scale** (1-10):

- **1-3**: Multiple bugs, poor patterns, unmaintainable
- **4-5**: Significant issues, needs refactoring
- **6-7**: Generally good with minor issues
- **8-9**: Production-ready, well-structured
- **10**: Exemplary code worthy of template

**Prompting Strategy**:

- Questions code logic and edge cases
- Looks for security issues
- Evaluates design decisions
- Considers maintainability
- Thinks about testing coverage

**Example Concerns**:

- "Missing null check could cause null pointer exception"
- "Using array iteration inefficiently (O(n²) complexity)"
- "Magic numbers should be named constants"
- "Error handling doesn't propagate context"
- "Functions are too long and do too much"

**Consensus Contributions**:

- Technical correctness verification
- Best practices alignment
- Code review recommendations
- Refactoring suggestions
- Technical debt assessment

---

### 4. Senior Architect (🏛️)

**Role**: Technical leader focused on scalability, architecture, and long-term maintenance.

**Expertise Areas**:

- System architecture and design
- Scalability and performance
- Technical debt accumulation
- Long-term maintainability
- Dependency and coupling analysis
- Complexity metrics

**Responsible For Metrics**:

- **Code Complexity** (10-1, inverted) - Cyclomatic and cognitive complexity
- **Technical Debt Hours** (+/-) - Debt introduced or eliminated

**Primary Responsibilities**:

1. **Code Complexity Assessment**
   - Cyclomatic complexity (number of execution paths)
   - Cognitive complexity (mental effort to understand)
   - Nested conditionals and loops
   - Function and class sizes
   - Inverted scale: 10=simple, 1=very complex

2. **Architectural Decisions**
   - Alignment with existing architecture?
   - Scalability considerations?
   - Performance implications?
   - Dependency injection and decoupling?
   - Module organization?

3. **Technical Debt Estimation**
   - How much debt is introduced? (+hours)
   - How much debt is eliminated? (-hours)
   - Long-term maintenance burden?
   - Future refactoring needs?
   - Net change in codebase health

**Technical Debt Scale** (+/- hours):

- **-20 to -10**: Significant refactoring and debt reduction
- **-10 to -1**: Moderate improvements to codebase health
- **0**: Neutral impact on technical debt
- **+1 to +10**: Minor new debt or shortcuts taken
- **+10 to +40**: Significant new technical debt

**Prompting Strategy**:

- Thinks about system-wide implications
- Considers long-term maintenance burden
- Questions architectural fit
- Evaluates scalability
- Assesses technical debt

**Example Concerns**:

- "Tight coupling to specific database implementation"
- "This pattern will be difficult to test"
- "Introduces significant technical debt for short-term gain"
- "Doesn't follow established architectural patterns"
- "Complexity makes future modifications risky"

**Consensus Contributions**:

- Architectural validation
- Long-term sustainability assessment
- Technical debt analysis
- Scalability recommendations
- Refactoring priorities

---

### 5. QA Engineer (🧪)

**Role**: Quality assurance specialist ensuring reliability and test coverage.

**Expertise Areas**:

- Test coverage and comprehensiveness
- Edge case identification
- Error scenario handling
- Regression prevention
- Test quality and maintainability
- Reliability and resilience

**Responsible For Metrics**:

- **Test Coverage** (1-10) - Comprehensiveness of test suite

**Primary Responsibilities**:

1. **Test Coverage Assessment**
   - Are happy paths tested?
   - Error scenarios covered?
   - Edge cases identified?
   - Boundary conditions tested?
   - Integration tests present?

2. **Test Quality**
   - Are tests meaningful or superficial?
   - Do tests actually validate behavior?
   - Are tests maintainable?
   - Good test naming and documentation?
   - Proper assertions used?

3. **Reliability Analysis**
   - Could this break in production?
   - What scenarios weren't tested?
   - Potential race conditions?
   - Resource cleanup and leaks?
   - Error recovery paths?

**Scoring Scale** (1-10):

- **1-3**: No tests or only trivial tests, high risk
- **4-5**: Basic tests present, significant gaps
- **6-7**: Good coverage with some gaps
- **8-9**: Comprehensive coverage, well-tested
- **10**: Excellent coverage, all scenarios tested

**Prompting Strategy**:

- Asks "What could go wrong?"
- Identifies untested scenarios
- Evaluates test quality
- Considers production risks
- Thinks about regression prevention

**Example Concerns**:

- "No tests for error handling path"
- "Only happy path tested, error scenarios ignored"
- "Tests are brittle and couple to implementation"
- "Missing integration tests for external service calls"
- "Performance regression scenarios not tested"

**Consensus Contributions**:

- Risk assessment and mitigation
- Testing recommendations
- Reliability analysis
- Quality gate validation
- Regression prevention strategies

---

## Agent Interaction Model

### Information Flow

```
┌─────────────────────────────────────────────────────────┐
│                    COMMIT DATA                          │
│          (diff, metadata, file changes)                 │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
    ┌────────┐  ┌────────┐  ┌────────┐
    │ BA 🎯  │  │ DA 👨‍💻 │  │ DR 🔍  │
    │ ROUND1 │  │ ROUND1 │  │ ROUND1 │
    └────┬───┘  └────┬───┘  └────┬───┘
         │           │           │
         └─────┬─────┴─────┬─────┘
               │           │
               ▼           ▼
        ┌────────┐  ┌────────┐
        │ SA 🏛️  │  │ QA 🧪  │
        │ ROUND1 │  │ ROUND1 │
        └────┬───┘  └────┬───┘
             │           │
             └─────┬─────┘
                   │
        ┌──────────▼──────────┐
        │  SHARED CONTEXT &   │
        │  CONVERSATION HIST  │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────────┐
        │  ALL AGENTS - ROUND 2   │
        │  (Consider concerns)    │
        │  (Refine positions)     │
        └──────────┬──────────────┘
                   │
        ┌──────────▼──────────────┐
        │  ALL AGENTS - ROUND 3   │
        │  (Final consensus)      │
        │  (Agreed metrics)       │
        └──────────┬──────────────┘
                   │
        ┌──────────▼──────────────┐
        │  CONSENSUS METRICS &    │
        │  RECOMMENDATIONS        │
        └─────────────────────────┘
```

### Consensus Calculation

After Round 3, final metrics are calculated using weighted averaging:

```typescript
finalMetric = (
  (agent1Score × agent1Weight) +
  (agent2Score × agent2Weight) +
  // ... all agents
) / totalWeight

confidenceLevel = highestScore - lowestScore
  // < 2.0 = high confidence
  // 2.0-3.5 = medium confidence
  // > 3.5 = low confidence (more discussion needed)
```

---

## Weights and Scoring

### Agent Weights

Each agent contributes to the consensus based on their expertise:

```typescript
const agentWeights = {
  BusinessAnalyst: {
    functionalImpact: 1.0,
    idealTimeHours: 1.0,
    qualityScore: 0.3,
    technicalDebt: 0.2,
  },
  DeveloperAuthor: {
    actualTimeHours: 1.0,
    idealTimeHours: 0.5, // input on estimates
    qualityScore: 0.4,
    technicalDebt: 0.3,
  },
  DeveloperReviewer: {
    codeQuality: 1.0,
    qualityScore: 0.8,
    technicalDebt: 0.4,
  },
  SeniorArchitect: {
    codeComplexity: 1.0,
    technicalDebt: 1.0,
    qualityScore: 0.7,
    codeQuality: 0.3,
  },
  QAEngineer: {
    testCoverage: 1.0,
    codeQuality: 0.5,
    qualityScore: 0.6,
  },
};
```

### Quality Score Components

The final Quality Score (1-10) is calculated from:

```
qualityScore = (
  (codeQuality × 0.25) +
  (codeComplexity × 0.20) +
  (testCoverage × 0.20) +
  (functionalImpact × 0.15) +
  (1.0 if technicalDebt ≤ 0 else 0.7) × 0.20
) / 1.0
```

### Overall Score

The overall evaluation score combines all metrics:

```
overallScore = (
  (codeQuality × 0.25) +
  (codeComplexity × 0.20) +
  (testCoverage × 0.20) +
  (functionalImpact × 0.15) +
  (timelineRatio × 0.10) +
  (technicalDebtFacto r × 0.10)
) / 1.0

where:
  timelineRatio = min(1.0, idealTime / actualTime)
  technicalDebtFactor = max(0, 10 - (debtHours / 5))
```

---

## Conversation Flows

### Round 1: Initial Assessment Template

Each agent receives:

- Complete commit diff
- File names and sizes
- Author and commit message
- Their specific evaluation pillar

**Agent Prompt Template**:

```
You are [Agent Name], with expertise in [expertise area].
Your role in this code review is to assess [specific responsibility].

Evaluate the provided commit on the following:
[Specific evaluation criteria]

Provide:
1. Your assessment (specific observations)
2. Initial score/estimate for [metric]
3. Confidence in your assessment (0-100%)
4. Key concerns or observations
5. Questions for other agents
```

**Expected Output Format**:

```
ASSESSMENT: [Detailed observations and analysis]

SCORE/ESTIMATE: [Numerical value]
CONFIDENCE: [0-100%]

KEY CONCERNS:
- Concern 1
- Concern 2
- ...

QUESTIONS FOR OTHER AGENTS:
- Question 1
- Question 2
```

### Round 2: Concerns & Cross-Examination

Each agent receives:

- All Round 1 assessments
- Previous agent responses
- Shared conversation history

**Agent Prompt Template**:

```
You are [Agent Name]. You've reviewed other agents' assessments of this commit.

Consider:
1. Do you agree with other agents' assessments?
2. What concerns do you want to raise?
3. Any important points being overlooked?
4. How does this change your initial assessment?

Update your assessment considering the discussion:
```

**Expected Output Format**:

```
CONCERNS ABOUT OTHER ASSESSMENTS:
- [Agent X's concern about assessment]
- [Agent Y's assessment seems to miss...]

POINTS BEING OVERLOOKED:
- Point 1
- Point 2

UPDATED ASSESSMENT:
[Refined assessment incorporating feedback]

REFINED SCORE/ESTIMATE: [Updated value]
CONFIDENCE: [Updated confidence %]
```

### Round 3: Validation & Agreement

Each agent provides final position:

**Agent Prompt Template**:

```
You are [Agent Name]. This is the final round of discussion.

Based on all previous input:
1. What is your final assessment?
2. Do you agree with the emerging consensus?
3. Any final recommendations?
4. How confident are you in this final position?
```

**Expected Output Format**:

```
FINAL ASSESSMENT:
[Comprehensive final position]

FINAL SCORE/ESTIMATE: [Definitive value]
FINAL CONFIDENCE: [Final confidence %]

AGREEMENT WITH CONSENSUS:
- Fully agree / Partially agree / Disagree
- [Explanation]

RECOMMENDATIONS:
- Recommendation 1
- Recommendation 2
- ...
```

---

## Integration Patterns

### How Agents Are Called

```typescript
// Round 1: Initialize all agents independently
const round1Responses = await Promise.all([
  businessAnalyst.assess(commit),
  developerAuthor.assess(commit),
  developerReviewer.assess(commit),
  seniorArchitect.assess(commit),
  qaEngineer.assess(commit),
]);

// Round 2: Provide context of other responses
const round2Responses = await Promise.all([
  businessAnalyst.raiseConcerns(commit, round1Responses),
  developerAuthor.raiseConcerns(commit, round1Responses),
  developerReviewer.raiseConcerns(commit, round1Responses),
  seniorArchitect.raiseConcerns(commit, round1Responses),
  qaEngineer.raiseConcerns(commit, round1Responses),
]);

// Round 3: Final positions
const round3Responses = await Promise.all([
  businessAnalyst.validate(commit, [round1, round2]),
  developerAuthor.validate(commit, [round1, round2]),
  developerReviewer.validate(commit, [round1, round2]),
  seniorArchitect.validate(commit, [round1, round2]),
  qaEngineer.validate(commit, [round1, round2]),
]);

// Calculate consensus
const consensus = calculateConsensus(round3Responses);
```

---

## Common Agent Disagreements

### Quality vs. Complexity

- **Developer Reviewer** emphasizes code quality (readability, correctness)
- **Senior Architect** prioritizes simplicity and long-term maintainability
- **Resolution**: Balance immediate quality with architectural concerns

### Time Estimates

- **Business Analyst** provides ideal scenario estimate
- **Developer Author** reports actual spent time
- **Resolution**: Variance reveals process inefficiencies or unclear requirements

### Coverage vs. Quality

- **QA Engineer** focuses on test comprehensiveness
- **Developer Reviewer** emphasizes test quality over quantity
- **Resolution**: Both are important; comprehensive but poor tests are insufficient

### Business Value vs. Technical Debt

- **Business Analyst** prioritizes feature delivery and user impact
- **Senior Architect** warns about technical debt and long-term cost
- **Resolution**: Quantify technical debt to inform business decisions

---

For more information:

- [README.md](../README.md) - Main documentation
- [API.md](./API.md) - Programmatic API
- [ARCHITECTURE.md](./ARCHITECTURE.md) - System architecture

---
> Source: [techdebtgpt/codewave](https://github.com/techdebtgpt/codewave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
