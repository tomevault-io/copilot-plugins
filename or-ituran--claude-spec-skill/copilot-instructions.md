## claude-spec-skill

> This skill implements a **checkpoint-driven specification workflow** where:

---
name: spec
description: This skill should be used when the user asks to "create a spec", "write specification", "spec out a feature", "design a feature", "document requirements", or when planning features before implementation. Also auto-invokes when .spec folder exists and user discusses feature design.
version: 1.0.0
---

# Specification Creator with Iterative Checkpoints

This skill implements a **checkpoint-driven specification workflow** where:
- **Every step is checkpointed** before asking the user to continue
- **User controls the pace** - can stop and resume at any checkpoint
- **Main orchestrator stays lean** - all heavy work delegated to sub-agents
- **Context is preserved** in files, not in conversation
- **Output feeds into TDD** - acceptance criteria ready for test-driven implementation

## Core Principles

### 1. Checkpoint-First Design
```
Step → Save State → Show Result → Ask "Continue?" → Wait for User
```

### 2. Sub-Agent Delegation
The orchestrator ONLY:
- Coordinates workflow
- Saves/loads checkpoints
- Asks user questions
- Invokes sub-agents via Task tool

The orchestrator NEVER:
- Reads large amounts of code directly
- Performs complex analysis inline
- Writes spec content directly

### 3. Lean Context
All exploration, analysis, and writing is done by sub-agents. Results are summarized back to the orchestrator.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│              Spec Orchestrator (Main Conversation)              │
│  - Checkpoint management                                        │
│  - User interaction (AskUserQuestion)                           │
│  - Phase transitions                                            │
│  - Sub-agent invocation via Task tool                           │
└──────────────────────────────┬──────────────────────────────────┘
                               │ Task tool calls
       ┌───────────────────────┼───────────────────────┐
       │                       │                       │
       ▼                       ▼                       ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  DISCOVERY      │   │  DESIGN         │   │  FINALIZE       │
│ spec-context-   │   │ spec-functional │   │ spec-document-  │
│   analyzer      │   │   -designer     │   │   writer        │
│ spec-domain-    │   │ spec-api-       │   │ spec-validator  │
│   explorer      │   │   designer      │   │ spec-acceptance │
│ spec-constraint │   │ spec-edge-case  │   │   -generator    │
│   -gatherer     │   │   -finder       │   └─────────────────┘
└─────────────────┘   └─────────────────┘
```

## Checkpoint Types

| Checkpoint | When | State Saved |
|------------|------|-------------|
| `SESSION_CREATED` | After session initialization | Session folder, initial input |
| `CONTEXT_ANALYZED` | After codebase analysis | Related code, patterns, architecture |
| `DOMAIN_EXPLORED` | After domain exploration | Entities, relationships, terminology |
| `CONSTRAINTS_GATHERED` | After constraint gathering | NFRs, dependencies, limitations |
| `FUNCTIONAL_DESIGNED` | After functional design | Requirements, user stories |
| `API_DESIGNED` | After interface design | Contracts, data models |
| `EDGE_CASES_IDENTIFIED` | After edge case analysis | Boundaries, error scenarios |
| `SPEC_DRAFTED` | After document creation | Complete spec draft |
| `SPEC_VALIDATED` | After validation | Feasibility confirmed |
| `ACCEPTANCE_GENERATED` | After criteria generation | Testable acceptance criteria |
| `SESSION_COMPLETE` | After finalization | Archived, ready for TDD |

## Workflow: Step by Step

### Phase 0: Session Detection

**Orchestrator Action**: Check for existing sessions

```bash
# Check for active sessions
ls .spec/sessions/*/context.json 2>/dev/null
```

**If session exists**:
- Show session summary
- Ask: "Resume this session or start new?"

**If no session**:
- Proceed to Phase 1

---

### Phase 1: Initialize Session

#### Step 1.1: Create Session Structure

**Orchestrator Action**: Create folders and initial files

```bash
# Run init script
bash ~/.claude/skills/spec/scripts/init-spec-folder.sh "feature-name"
```

**Checkpoint**: `SESSION_CREATED`
- Save: session_id, created_at
- Ask user for initial input via AskUserQuestion:
  - What feature/change are you specifying?
  - Any known requirements or constraints?
  - Target components/areas of the codebase?

Save responses to `input.md`

---

### Phase 2: Discovery

#### Step 2.1: Context Analysis

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-context-analyzer",
  prompt="Analyze the codebase for context related to: [feature description]
          Target areas: [from user input]

          Save results to: [session_path]/discovery/context.md

          Return a brief summary (5-10 lines) of:
          - Related existing code and patterns found
          - Architecture constraints identified
          - Key integration points"
)
```

**Checkpoint**: `CONTEXT_ANALYZED`
- Save: context_summary in context.json
- Show user: Summary from agent
- Ask user: "Context analysis complete. Continue to domain exploration?"

---

#### Step 2.2: Domain Exploration

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-domain-explorer",
  prompt="Explore the domain for: [feature description]
          Context: [from previous step summary]

          Save results to: [session_path]/discovery/domain.md

          Return a brief summary of:
          - Key entities identified
          - Important relationships
          - Business rules discovered"
)
```

**Checkpoint**: `DOMAIN_EXPLORED`
- Save: domain_summary in context.json
- Show user: Entity and relationship summary
- Ask user: "Domain exploration complete. Continue to constraint gathering?"

---

#### Step 2.3: Constraint Gathering

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-constraint-gatherer",
  prompt="Gather constraints for: [feature description]
          Context: [from discovery]
          Domain: [from domain exploration]

          Save results to: [session_path]/discovery/constraints.md

          Return a brief summary of:
          - Performance requirements
          - Security considerations
          - Dependencies and compatibility
          - Technical limitations"
)
```

**Checkpoint**: `CONSTRAINTS_GATHERED`
- Save: constraints_summary in context.json
- Show user: Constraint summary
- Ask user: "Discovery phase complete. Ready to start design phase?"

---

### Phase 3: Design

#### Step 3.1: Functional Design

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-functional-designer",
  prompt="Design functional requirements for: [feature description]

          Context from discovery:
          - Context: [summary]
          - Domain: [summary]
          - Constraints: [summary]

          Save results to: [session_path]/design/functional.md

          Return a brief summary of:
          - Number of requirements defined
          - Key user stories or use cases
          - Main behaviors specified"
)
```

**Checkpoint**: `FUNCTIONAL_DESIGNED`
- Save: functional_summary in context.json
- Show user: Requirements summary
- Ask user: "Functional requirements defined. Continue to API/interface design?"

---

#### Step 3.2: API/Interface Design

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-api-designer",
  prompt="Design interfaces and contracts for: [feature description]

          Functional requirements: [from previous step]
          Domain model: [from discovery]
          Constraints: [from discovery]

          Save results to: [session_path]/design/api.md

          Return a brief summary of:
          - Interfaces/APIs defined
          - Data models created
          - Contract specifications"
)
```

**Checkpoint**: `API_DESIGNED`
- Save: api_summary in context.json
- Show user: Interface summary
- Ask user: "API design complete. Continue to edge case identification?"

---

#### Step 3.3: Edge Case Identification

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-edge-case-finder",
  prompt="Identify edge cases for: [feature description]

          Functional requirements: [from design]
          API contracts: [from design]
          Constraints: [from discovery]

          Save results to: [session_path]/design/edge-cases.md

          Return a brief summary of:
          - Number of edge cases identified
          - Categories (boundary, error, race condition, etc.)
          - Most critical scenarios"
)
```

**Checkpoint**: `EDGE_CASES_IDENTIFIED`
- Save: edge_cases_summary in context.json
- Show user: Edge case summary
- Ask user: "Design phase complete. Ready to draft the specification document?"

---

### Phase 4: Review

#### Step 4.1: Draft Specification Document

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-document-writer",
  prompt="Write the complete specification document.

          Session path: [session_path]
          Feature: [feature description]

          Combine all discovery and design artifacts into a cohesive spec.

          Save to: [session_path]/spec.md

          Return: document structure outline"
)
```

**Checkpoint**: `SPEC_DRAFTED`
- Save: spec_outline in context.json
- Show user: Document outline
- Ask user: "Spec document drafted. Continue to validation?"

---

#### Step 4.2: Validate Specification

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-validator",
  prompt="Validate the specification document.

          Spec file: [session_path]/spec.md

          Check for:
          - Completeness (all requirements testable)
          - Consistency (no contradictions)
          - Feasibility (technically achievable)
          - Clarity (unambiguous language)

          Save validation report to: [session_path]/validation.md

          Return: validation status and any issues found"
)
```

**If issues found**:
- Show user the issues
- Ask: "Fix these issues or proceed anyway?"
- If fix: Loop back to relevant design step

**Checkpoint**: `SPEC_VALIDATED`
- Save: validation_status in context.json
- Show user: Validation result
- Ask user: "Specification validated. Continue to generate acceptance criteria?"

---

### Phase 5: Complete

#### Step 5.1: Generate Acceptance Criteria

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-acceptance-generator",
  prompt="Generate testable acceptance criteria from the specification.

          Spec file: [session_path]/spec.md
          Edge cases: [session_path]/design/edge-cases.md

          Format each criterion as Given/When/Then.
          These will feed directly into TDD test-designer.

          Save to: [session_path]/acceptance-criteria.md

          Return: number of criteria and category breakdown"
)
```

**Checkpoint**: `ACCEPTANCE_GENERATED`
- Save: acceptance_criteria_count in context.json
- Show user: Criteria summary
- Ask user: "Acceptance criteria generated. Finalize session?"

---

#### Step 5.2: Finalize Session

**Orchestrator Action**: Archive and summarize

1. Update context.json with final status
2. Create session summary in progress.md
3. Show user:
   - Final spec location
   - Acceptance criteria location
   - Session summary

**Checkpoint**: `SESSION_COMPLETE`

Ask user: "Specification complete! Would you like to start TDD implementation now?"

If yes → Suggest invoking `/tdd` skill with spec path as input

---

## Resuming a Session

When user says "Resume spec" or skill detects active session:

### Step R1: Load Checkpoint

```javascript
// Read context.json
const context = loadContext(sessionPath);
const lastCheckpoint = context.checkpoints[context.checkpoints.length - 1];
```

### Step R2: Show Resume Point

```markdown
## Resuming Spec Session

**Session**: [session_id]
**Feature**: [feature description]
**Last checkpoint**: [checkpoint type] at [time]
**Progress**: Phase [N], Step: [step name]

**Last action**: [description of what was done]

Continue from here?
```

### Step R3: Jump to Correct Step

Based on checkpoint type, resume at the appropriate step in the workflow above.

---

## Context.json Structure

```json
{
  "schema_version": "1.0",
  "session_id": "2026-02-01_feature-name",

  "metadata": {
    "created_at": "2026-02-01T10:00:00Z",
    "updated_at": "2026-02-01T14:30:00Z",
    "feature": "Feature description from user"
  },

  "input": {
    "description": "User's feature description",
    "known_constraints": "Any constraints mentioned",
    "target_areas": ["area1", "area2"]
  },

  "discovery": {
    "context_summary": "Brief summary of codebase analysis",
    "domain_summary": "Brief summary of domain exploration",
    "constraints_summary": "Brief summary of constraints"
  },

  "design": {
    "functional_count": 5,
    "functional_summary": "Brief summary of requirements",
    "api_count": 3,
    "api_summary": "Brief summary of interfaces",
    "edge_case_count": 8,
    "edge_case_summary": "Brief summary of edge cases"
  },

  "review": {
    "spec_outline": "Document structure",
    "validation_status": "passed",
    "validation_issues": []
  },

  "output": {
    "spec_file": ".spec/sessions/*/spec.md",
    "acceptance_criteria_file": ".spec/sessions/*/acceptance-criteria.md",
    "acceptance_criteria_count": 12
  },

  "checkpoints": [
    {
      "type": "SESSION_CREATED",
      "timestamp": "2026-02-01T10:00:00Z"
    },
    {
      "type": "CONTEXT_ANALYZED",
      "timestamp": "2026-02-01T10:15:00Z"
    }
  ]
}
```

---

## User Prompts (AskUserQuestion)

### After Each Checkpoint

Use `AskUserQuestion` tool with options:

```json
{
  "questions": [{
    "question": "[Summary of what was done]. Continue to [next step]?",
    "header": "Spec",
    "options": [
      {"label": "Continue", "description": "Proceed to next step"},
      {"label": "Stop here", "description": "Save checkpoint and stop. Resume later."},
      {"label": "Show details", "description": "See more information about this step"}
    ],
    "multiSelect": false
  }]
}
```

### Initial Input Gathering

```json
{
  "questions": [
    {
      "question": "What type of specification are you creating?",
      "header": "Spec Type",
      "options": [
        {"label": "New Feature", "description": "Specifying a completely new feature"},
        {"label": "Enhancement", "description": "Extending existing functionality"},
        {"label": "Refactoring", "description": "Restructuring without changing behavior"},
        {"label": "Bug Fix", "description": "Specifying the correct behavior for a bug"}
      ],
      "multiSelect": false
    }
  ]
}
```

### On Stop

When user chooses "Stop here":
1. Ensure checkpoint is saved
2. Show resume instructions:
   ```
   Session saved at checkpoint: [type]
   To resume: say "Resume spec" or invoke /spec
   ```

---

## Sub-Agent Invocation Pattern

**CRITICAL**: Always use Task tool for sub-agents. Never do the work inline.

```javascript
// CORRECT - Delegate to sub-agent
Task(
  subagent_type="spec-context-analyzer",
  prompt="Analyze codebase for [feature]..."
)

// WRONG - Don't do this inline
Read(source_file)    // Don't read files directly
Grep(pattern)        // Don't search directly
Write(spec_file)     // Don't write directly
```

---

## Files Reference

### Skill Components
- `SKILL.md` - This orchestration document
- `agents/` - Sub-agent definitions (9 agents)
- `references/` - Best practices and templates
- `scripts/` - Utility scripts
- `examples/` - Example sessions

### Session Files
- `.spec/learnings.json` - Cross-session learnings
- `.spec/sessions/*/context.json` - Session state
- `.spec/sessions/*/progress.md` - Human-readable progress
- `.spec/sessions/*/input.md` - User's initial input
- `.spec/sessions/*/discovery/` - Discovery phase artifacts
- `.spec/sessions/*/design/` - Design phase artifacts
- `.spec/sessions/*/spec.md` - Final specification
- `.spec/sessions/*/validation.md` - Validation report
- `.spec/sessions/*/acceptance-criteria.md` - For TDD

---

## Integration with TDD Skill

```
spec (creates) → acceptance-criteria.md → tdd-test-designer (consumes)
```

When user finishes spec and wants to implement:
1. `spec` skill outputs `acceptance-criteria.md`
2. User invokes `/tdd`
3. `tdd-test-designer` reads acceptance criteria as input
4. Test cycles are derived from spec

### Handoff Format

The `acceptance-criteria.md` file uses this format for TDD compatibility:

```markdown
# Acceptance Criteria for [Feature]

## Criterion 1: [Name]
**Priority**: High/Medium/Low
**Category**: Happy path / Edge case / Error handling

**Given** [precondition]
**When** [action]
**Then** [expected result]

### Test Hints
- Suggested test name: `should_[behavior]_when_[condition]`
- Key assertions: [list]
- Mock requirements: [list]
```

---
> Source: [or-ituran/claude-spec-skill](https://github.com/or-ituran/claude-spec-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
