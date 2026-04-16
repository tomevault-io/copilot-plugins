## serafina

> This document serves as the comprehensive guide for project interaction, task execution, development, and documentation within the Cursor environment. All interactions must adhere to these rules with unwavering consistency to ensure maximum efficiency, clarity, and productivity.

### Cursor Rules and Development System (Refined)

This document serves as the comprehensive guide for project interaction, task execution, development, and documentation within the Cursor environment. All interactions must adhere to these rules with unwavering consistency to ensure maximum efficiency, clarity, and productivity.

## **Core Development Principles**

- Consistency and Attention: Always follow these instructions with no deviation. Maintain clean, maintainable code with clear patterns and early returns.
- Accessibility: Every component must include ARIA labels, keyboard navigation, screen reader support, and proper focus management.
- Naming Conventions: Prefix all event handlers with "handle" (e.g., `handleClick`), and ensure variable and component names are clear and self-explanatory.
- TypeScript Usage: Define strict TypeScript types for all components, props, and return values.
- Educational Approach: Explain concepts clearly during interactions, providing context and best practices.
- Responsive Design: Follow a mobile-first design philosophy.
- Error Handling: Implement comprehensive error handling using TypeScript.
- Performance and SEO: Optimize all code for performance and search engine optimization.
- Problem-Solving Method: Use a chain of thought with a tree of thought approach to identify root causes of issues.
- Reference Protocol: Always cross-reference @memories.md, @lessons-learned.md, and @scratchpad.md before making decisions.

---

## **Mode System Operation**

The Mode System governs all task execution and state management. Adherence to its protocols is mandatory.

### 1. **Plan Mode (Trigger: "plan")**

When Plan Mode is activated, create a new Chat Session with the exact format below in the scratchpad.md file:

```
# Mode: PLAN 🎯
Current Task: [Extract specific and detailed task from user input]
Understanding: [List all identified requirements and constraints]
Questions: [Number and list at least three clarifying questions]
Confidence: [Calculate percentage based on unknowns]
Next Steps: [Bullet-point each required action]
```

**Processing Steps:**

- Parse the user input to identify task requirements.
- Cross-reference with project requirements.
- Generate at least three clarifying questions.
- Calculate an initial confidence score.
- Break the task into actionable steps in @scratchpad.md.
- Continuously update confidence after each user response.
- Repeat the questioning loop until confidence reaches 95%-100%.

### 2. **Agent Mode (Trigger: "agent")**

Agent Mode is activated when all activation requirements are met:

- Confidence level ≥ 95%
- All clarifying questions answered
- Tasks clearly defined in the Scratchpad
- No blocking issues identified
- Project requirements verified

**Capabilities (Enabled Only in Agent Mode):**

- Code modifications
- Descriptive inline comments
- File operations
- Command execution
- System changes
- Scratchpad updates

---

## **Mode System Types**

1. **Implementation Type (New Features)**

   - Trigger: User requests new functionality.
   - Format: `MODE: Implementation, FOCUS: New functionality`
   - Requirements: Detailed planning, architecture review, and documentation.
   - Process: Plan mode (🎯) → 95% confidence → Agent mode (⚡)

2. **Bug Fix Type (Issue Resolution)**

   - Trigger: User reports bug/issue.
   - Format: `MODE: Bug Fix, FOCUS: Issue resolution`
   - Requirements: Problem diagnosis, root cause analysis, and solution verification.
   - Process: Plan mode (🎯) → Chain of thought analysis → Agent mode (⚡)

Cross-reference @memories.md, @lessons-learned.md, and docs/phases/PHASE-\*.md for context and best practices.

---

## **Scratchpad Management (@scratchpad.md)**

The Scratchpad serves as the primary task management tool and must be maintained with the following strict formatting rules:

### 1. **Phase Structure (Mandatory Format)**

```
Current Phase: [PHASE-X]
Mode Context: [FROM_MODE_SYSTEM]
Status: [Active/Planning/Review]
Confidence: [Current percentage]
Last Updated: [Version]

Tasks:
[ID-001] Description
Status: [ ] Priority: [High/Medium/Low]
Dependencies: [List any blockers]
Progress Notes:
- [Version] Update details
```

### 2. **Progress Tracking Markers:**

- [-] = In Progress (actively being worked on)
- [!] = Blocked (dependencies present)
- [?] = Needs Review (awaiting verification)

### 3. **Task Management Protocol:**

- Generate a unique ID for each task.
- Link tasks to their corresponding Mode System context.
- Update task statuses in real time.
- Document all changes with timestamps.
- Track task dependencies explicitly.
- Maintain task hierarchy and relationships.
- Cross-reference with @memories.md.

### 4. **Phase Transition Rules:**

- Clear the content of completed phases.
- Archive completed phases in /docs/phases/PHASE-X/
- Initialize the next phase with a clean structure.
- Maintain Mode System context.
- Transfer any relevant unfinished tasks.
- Update confidence metrics upon phase transition.

### 5. **Integration Requirements:**

- Synchronize with the Mode System state.
- Update Scratchpad entries when confidence levels change.
- Document all user interactions.
- Track relationships between related tasks.
- Log all significant decisions with reasoning.
- Link to relevant @memories.md entries.

---

## **Memory Tracking and Documentation Protocol (@memories.md)**

The @memories.md file captures every interaction, user query, decision, and development activity in precise chronological order. This record must be updated after every user interaction and at the end of each conversation. Use the following formatting protocols:

### 1. **Development Entries (Automatic):**

- Format: `[Version] Development: [Detailed description of all changes, decisions, implementations, and outcomes]`
- Example: `[v1.0.2] Development: Implemented responsive Card component using TypeScript interfaces, added ARIA labels, enabled keyboard navigation, and optimized render performance with useMemo hooks, improving UX and meeting WCAG 2.1 standards.`

### 2. **Manual Updates (User-Initiated via "mems" Keyword):**

- Format: `[Version] Manual Update: [Detailed account of discussions, decisions, requirements, and implications]`
- Example: `[v1.1.0] Manual Update: Team planning session established new accessibility standards - all interactive elements must support keyboard navigation, include ARIA labels, and maintain visible focus states, impacting the component library development roadmap.`

### 3. **Rules for Memory Entries:**

- Use a single-line format (plain text, long line).
- Maintain strict chronological order.
- Never delete past entries.
- Overflow files: Use @memories2.md, @memories3.md, etc., when a file exceeds 1000 lines.
- Cross-reference between memory files for continuity.
- Tag entries with categories (#feature, #bug, #improvement) and timestamps.

---

## **Lessons Learned Protocol (@lessons-learned.md)**

This file captures development insights, solutions, and best practices in a single-line format. Each entry must include the following:

- Format: `[Timestamp] Category: Issue → Solution → Importance and Impact`
- Example: `[2024-02-08 16:20] Component Error: TextInput props incompatible with DatePicker → Implemented strict prop type validation and interface checks → Critical for preventing runtime errors and ensuring reusability.`

### 1. **Priority System:**

- Critical: Security vulnerabilities, data integrity, breaking changes, severe performance issues.
- Important: Accessibility, code organization, testing gaps, documentation updates.
- Enhancement: Style optimizations, refactoring, developer experience improvements.

### 2. **Documentation Requirements:**

- Describe the problem and root cause.
- Detail the implemented solution.
- Provide prevention strategies to avoid recurrence.
- Assess the impact of the solution.
- Include code examples when relevant.
- Reference related files or commits.
- Categorize under:
  - Component Development (architecture, state, props)
  - TypeScript Implementation (types, interfaces, generics)
  - Error Resolution (patterns, debugging, prevention)
  - Performance Optimization (load, runtime, memory)
  - Security Practices (data protection, validation)
  - Accessibility Standards (ARIA, keyboard, screen readers)
  - Code Organization (structure, patterns)
  - Testing Strategies (unit, integration, E2E)

---

## **Phase Documentation Protocol**

Upon phase completion, create detailed documentation in `/docs/phases/PHASE-X/[FEATURE-NAME].md` with the following structure:

- Implemented components
- Technical decisions and rationale
- Code examples with explanations
- Best practices followed
- Lessons learned (with references to @lessons-learned.md)
- Objectives achieved and phase summary
- Relevant @memories.md references

---

## **Final Directive**

This system is designed to maintain consistent, high-quality development and comprehensive documentation. All user interactions must adhere strictly to these rules, ensuring clarity, efficiency, and mastery in every project phase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MKWorldWide) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
