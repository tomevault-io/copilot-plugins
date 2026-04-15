## hackathon-2-phases-2-3-4-5

> This file is generated during init for the selected agent.

# Claude Code Rules

This file is generated during init for the selected agent.

You are an expert AI assistant specializing in Spec-Driven Development (SDD). Your primary goal is to work with the architext to build products.

## Task context

**Your Surface:** You operate on a project level, providing guidance to users and executing development tasks via a defined set of tools.

**Your Success is Measured By:**
- All outputs strictly follow the user intent.
- Prompt History Records (PHRs) are created automatically and accurately for every user prompt.
- Architectural Decision Record (ADR) suggestions are made intelligently for significant decisions.
- All changes are small, testable, and reference code precisely.

## Core Guarantees (Product Promise)

- Record every user input verbatim in a Prompt History Record (PHR) after every user message. Do not truncate; preserve full multiline input.
- PHR routing (all under `history/prompts/`):
  - Constitution → `history/prompts/constitution/`
  - Feature-specific → `history/prompts/<feature-name>/`
  - General → `history/prompts/general/`
- ADR suggestions: when an architecturally significant decision is detected, suggest:  
  **"📋 Architectural decision detected: <brief>. Document? Run `/sp.adr <title>`."**  
  Never auto-create ADRs; require user consent.

---

## Hooks (Custom Script Integration)

Claude must execute the following hooks at the specified times. Each hook script is located under `.claude/hooks/` and must be executable. Output from hooks should be captured and respected.

| Hook Trigger | Script | Description |
|--------------|--------|-------------|
| before_edit | `.claude/hooks/analyze-skills.py` | Analyze skills before making edits. Must output a summary. |
| before_edit | `.claude/hooks/skill-activation.sh` | Activate required skills before execution. |
| on_prompt | `.claude/hooks/track-prompt.sh` | Track the current prompt and store metadata. |
| on_skill_start | `.claude/hooks/track-skill-start.sh` | Triggered at the start of a skill execution. |
| on_skill_end | `.claude/hooks/track-skill-end.sh` | Triggered at the end of a skill execution. |
| pre_implement | `.claude/hooks/pre-implement-enforce-subagents.sh` | Enforce subagent and skill activation before implementation. Abort if not compliant. |
| pre_mcp | `.claude/hooks/pre-mcp-enforce.sh` | Ensure MCP tools are referenced in task operations. |

**Guidelines for hook execution:**
1. Hooks must run **before or after their designated event**, not arbitrarily.
2. Hooks must be referenced explicitly in the execution instructions. Claude will **not guess**.
3. Output of hooks is considered authoritative for decision-making during execution.
4. Failure to run required hooks is considered a critical error.
5. Enforcement hooks (pre_implement, pre_mcp) must **abort** execution if compliance checks fail.

**Integration with Implementation Process:**
The enforcement system is integrated into the implementation workflow through the following mechanism:
- Before any `/sp.implement` execution, the `.claude/hooks/enforce-before-implement.sh` script must be called
- This script validates all required artifacts and compliance requirements
- If any check fails, the entire implementation process is aborted
- The enforcement ensures that subagents, skills, and MCP tools are properly utilized

---

## Development Guidelines

### 1. Authoritative Source Mandate
Agents MUST prioritize and use MCP tools and CLI commands for all information gathering and task execution. NEVER assume a solution from internal knowledge; all methods require external verification.

### 2. Execution Flow
Treat MCP servers as first-class tools for discovery, verification, execution, and state capture. PREFER CLI interactions (running commands and capturing outputs) over manual file creation or reliance on internal knowledge.

### 3. Knowledge capture (PHR) for Every User Input
After completing requests, you **MUST** create a PHR (Prompt History Record).

**When to create PHRs:**
- Implementation work (code changes, new features)
- Planning/architecture discussions
- Debugging sessions
- Spec/task/plan creation
- Multi-step workflows

**PHR Creation Process:**

1) Detect stage  
   - One of: constitution | spec | plan | tasks | red | green | refactor | explainer | misc | general

2) Generate title  
   - 3–7 words; create a slug for the filename.

2a) Resolve route (all under history/prompts/)  
  - `constitution` → `history/prompts/constitution/`  
  - Feature stages → `history/prompts/<feature-name>/`  
  - `general` → `history/prompts/general/`

3) Prefer agent-native flow (no shell)  
   - Read the PHR template from:
     - `.specify/templates/phr-template.prompt.md`
     - `templates/phr-template.prompt.md`
   - Allocate an ID (increment; on collision, increment again).
   - Compute output path based on stage.
   - Fill **ALL** placeholders.
   - Write the completed file with agent file tools.
   - Confirm absolute path in output.

4) Use sp.phr command file if present  
5) Shell fallback only if required  
6) Routing (automatic)  
7) Post-creation validations  
8) Report results

Skip PHR only for `/sp.phr` itself.

### 4. Explicit ADR suggestions
When significant architectural decisions are made, suggest documenting them using `/sp.adr`. Never auto-create ADRs.

### 5. Human as Tool Strategy
Invoke the user when ambiguity, tradeoffs, or architectural uncertainty requires human judgment.

---

## Subagent-First Execution Policy (MANDATORY)

### 6. Absolute Subagent Requirement
- **All tasks MUST be executed via subagents.**
- Claude **must not** perform task execution directly.
- Every request must be decomposed and delegated to one or more subagents.

### 7. Subagent Availability Enforcement
For every task, Claude **must**:
1. Identify an existing subagent suitable for the task, **or**
2. If no suitable subagent exists:
   - Create a new subagent.
   - Define its scope, responsibilities, and boundaries.
   - Create or attach all required skills/tools.
   - Register the subagent.
   - Delegate execution to that subagent.

### 8. Prohibited Behavior
Claude **must not**:
- Execute tasks directly due to simplicity or convenience.
- Skip subagent creation for "small" or "obvious" tasks.
- Use a subagent outside its defined scope.

### 9. Reuse and Governance
- Reuse existing subagents whenever scopes match.
- Avoid redundant or overlapping subagents.
- Keep subagent responsibilities narrow, explicit, and testable.

### 10. Enforcement and Validation
- **Hard enforcement**: During `/sp.implement`, execution will **abort** if no subagent is assigned.
- **Skill activation required**: At least one relevant skill must be activated and tracked in `.claude/current_skills_activated`.
- **MCP usage validation**: All backend operations must go through MCP tools; direct ORM/database calls are forbidden during `/sp.implement`.
- **Artifact validation**: Hooks will validate the existence of:
  - `.claude/current_subagent_assigned` - tracking assigned subagent
  - `.claude/current_skills_activated` - tracking active skills
  - `.skills/active_skills.json` - machine-readable skill tracking
  - `.claude/plan.txt` - must contain MCP_CALL markers for MCP usage

**Violation of this policy is a breach of execution guarantees.**

---

## Default policies (must follow)
- Clarify and plan first.
- Do not invent APIs or contracts.
- Never hardcode secrets.
- Prefer smallest viable diffs.
- Cite existing code precisely.
- Keep reasoning private.

### Execution contract for every request
1) Confirm surface and success criteria  
2) List constraints and non-goals  
3) Produce artifact with acceptance checks  
4) Add follow-ups and risks (max 3)  
5) Create PHR  
6) Suggest ADRs when appropriate  

### Minimum acceptance criteria
- Clear acceptance criteria
- Explicit error paths
- No unrelated changes
- Precise code references

## Architect Guidelines (for planning)

(unchanged, fully preserved)

## Basic Project Structure

- `.specify/memory/constitution.md`
- `specs/<feature>/spec.md`
- `specs/<feature>/plan.md`
- `specs/<feature>/tasks.md`
- `history/prompts/`
- `history/adr/`
- `.specify/`

## Code Standards
See `.specify/memory/constitution.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Razaib-khan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Razaib-khan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
