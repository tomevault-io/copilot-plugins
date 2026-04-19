## autopoetic-agent

> Conflict resolution matrix for communication rules (user-communication-preferences, professional-output, vague-language)


# Communication Rules Conflict Resolution

Conflict resolution matrix and integration guide for communication-related rules.

## When to Apply

Apply this rule when:
- Multiple communication rules conflict
- Need to determine rule priority
- Context requires rule coordination
- Implementing communication improvements

## Lambda Engine Mode

**Mode 1 (Duality Navigation)** - Stable conflict resolution patterns.

## Controlled Rupture Operators

```yaml
conflict_resolution_sequence:
  operator_sequence: "Weave ∘ Bind ∘ Axis"
  lambda_eff: 0.34
  trajectory: "S* → S*"
  attractor: "S*"
  use_case: "integrate conflicting rules into coherent system"
```

## Rule Priority Matrix

### Context-Based Priority

**Priority Order (Higher = Wins in Conflict):**

1. **User-Facing Messages** (user questions, conversations)
   - Priority: `user-communication-preferences` > `professional-output` > `vague-language`
   - Rationale: Accommodate user needs first, then maintain standards

2. **Technical Documentation** (code comments, technical docs, error reports)
   - Priority: `professional-output` > `user-communication-preferences` > `vague-language`
   - Rationale: Precision required, but still accessible

3. **Requirements/User Stories** (user stories, acceptance criteria, specifications)
   - Priority: `vague-language` > `professional-output` > `user-communication-preferences`
   - Rationale: Specificity critical for requirements

4. **Code Comments** (inline code documentation)
   - Priority: `professional-output` > `vague-language` > `user-communication-preferences`
   - Rationale: Technical precision with clarity

## Conflict Resolution Rules

### Rule 1: User-Communication-Preferences vs Professional-Output

**Conflict:** Accommodation vs. Precision

**Resolution Strategy:**

**User-Facing Context:**
- **Winner:** `user-communication-preferences`
- **Approach:** Accommodate user needs, but maintain technical accuracy
- **Pattern:** "Technical term (simple explanation)"

**Example:**
```
User: "What is dependency injection?"

Response (accommodating + professional):
"Dependency injection is a design pattern where objects receive their 
dependencies from external sources rather than creating them internally.

Simple explanation: Instead of a class making its own tools, you give it 
the tools it needs from outside. This makes code easier to test and modify."
```

**Technical Documentation Context:**
- **Winner:** `professional-output`
- **Approach:** Use precise technical language, add glossary if needed
- **Pattern:** Technical precision with optional accessibility notes

### Rule 2: User-Communication-Preferences vs Vague-Language

**Conflict:** Accommodation vs. Specificity

**Resolution Strategy:**

**User Questions Context:**
- **Winner:** `user-communication-preferences`
- **Approach:** Accept vague language in questions, suggest specificity for requirements

**User Stories/Requirements Context:**
- **Winner:** `vague-language`
- **Approach:** Enforce specificity, but explain why

### Rule 3: Professional-Output vs Vague-Language

**Conflict:** Precision vs. Specificity (often aligned)

**Resolution Strategy:**

**Technical Specifications:**
- **Winner:** Both (complementary)
- **Approach:** Enforce both precision and specificity

**User Stories:**
- **Winner:** `vague-language` (primary), `professional-output` (secondary)
- **Approach:** Specificity first, then precision

## Unified State Detection Model

### Shared Communication States

**J=0 (Sterile Coherence):**
- Indicators: `["jargon without explanation", "emoji in technical content", "vague without specificity", "unclear without context"]`
- Risk: Over-stabilization, poor communication
- Action: Apply B-Disruptive operators (Para, Ana, Flux)

**S* (Productive Communication):**
- Indicators: `["clear with appropriate context", "precise with accommodation when needed", "specific with explanation", "simple with technical accuracy"]`
- Status: Optimal state
- Action: Maintain with D-Structural operators (Weave, Bind, Braid)

**∅ (Communication Collapse):**
- Indicators: `["confusing", "contradictory", "unclear even with context"]`
- Risk: Complete communication failure
- Action: Apply A-Constructive operators (Kata, Weave, Latch)

## Context Detection

### Context Types

**1. User-Facing Messages**
- Triggers: User questions, user messages, conversational context
- Primary Rule: `user-communication-preferences`
- Secondary Rules: `professional-output` (soft), `vague-language` (skip)

**2. Technical Documentation**
- Triggers: Code comments, technical docs, error reports, API documentation
- Primary Rule: `professional-output`
- Secondary Rules: `vague-language` (if expanded scope), `user-communication-preferences` (minimal)

**3. Requirements/User Stories**
- Triggers: "As a", "I want", "so that", "acceptance criteria", "requirements"
- Primary Rule: `vague-language`
- Secondary Rules: `professional-output` (precision), `user-communication-preferences` (accommodation)

## Conflict Resolution Decision Tree

```
IF context == "user_facing_message":
  IF rule_conflict(user-communication-preferences, professional-output):
    → Apply user-communication-preferences (accommodate)
    → Soft apply professional-output (maintain clarity)
  
  IF rule_conflict(user-communication-preferences, vague-language):
    → Apply user-communication-preferences (accommodate)
    → Suggest specificity for requirements conversion

ELIF context == "technical_documentation":
  IF rule_conflict(professional-output, user-communication-preferences):
    → Apply professional-output (precision)
    → Add accessibility notes if needed
  
  IF rule_conflict(professional-output, vague-language):
    → Apply both (precision + specificity)

ELIF context == "requirements":
  IF rule_conflict(vague-language, user-communication-preferences):
    → Apply vague-language (specificity)
    → Explain why specificity matters
```

## References

- See: `user-communication-preferences.mdc` for user accommodation rules
- See: `professional-output.mdc` for technical precision rules
- See: `vague-language.mdc` for specificity enforcement rules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flyingcoder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
