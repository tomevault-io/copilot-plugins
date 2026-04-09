## 2jz

> This file provides guidance to Claude Code when working with code in this repository.

# claude.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview
Build the Flight Simulator of Engine Building. This is an educational tool where I can simulate taking apart and rebuilding a 2jz engine so many times that it becomes muscle memory for me.

# Software Design Philosophy Instructions

Follow the principles from "A Philosophy of Software Design" by John Ousterhout when writing software.

## Core Mindset

### Strategic Programming Over Tactical Programming
- **Working code is not enough.** The primary goal is producing a great design; working code is a side effect.
- Invest 10-20% of development time in design improvements - this pays back quickly through faster development.
- Take a "zero tolerance" approach to complexity. Never justify adding "just a little complexity" for short-term gains.
- Make continual small investments to improve system design rather than large upfront investments.

### Complexity is the Enemy
- **Complexity Definition**: Anything related to software structure that makes it hard to understand and modify.
- Complexity manifests as:
  - **Change amplification**: Simple changes require modifications in many places
  - **Cognitive load**: How much a developer must know to complete a task
  - **Unknown unknowns**: It's unclear what needs to be modified or what information is needed
- Complexity causes: **Dependencies** and **Obscurity** (莫名)
- Complexity accumulates incrementally - fight every small addition

## Module Design Principles

### 1. Modules Should Be Deep
- **Deep modules**: Simple interface, powerful functionality
- **Shallow modules**: Complex interface relative to functionality (RED FLAG)
- The best modules hide the most complexity behind the simplest interfaces
- Depth = functionality / interface complexity
- Example: Unix file I/O (5 simple methods, thousands of lines of implementation)

**When designing classes:**
- Aim for simple interfaces with powerful implementations
- Information hiding is the most important technique for deep modules
- Each module should encapsulate knowledge that doesn't appear in its interface
- Avoid "classitis" - resist pressure to make every class small
- It's often better to combine related functionality into one deeper class

### 2. Information Hiding and Leakage
- **Information Hiding** (GOOD): Design decisions embedded in implementation, not visible in interface
- **Information Leakage** (RED FLAG): Design decisions reflected in multiple modules

**Guidelines:**
- Hide complexity within modules
- When same knowledge appears in multiple places, consider:
  - Merging the modules
  - Creating a new module that encapsulates just that knowledge
  - Pulling information into a shared abstraction

**Avoid Temporal Decomposition** (RED FLAG):
- Don't structure code based on time order of operations
- Focus on knowledge needed for each task, not order of execution
- Example: Don't separate file reading and parsing into separate classes if both need to understand file format

### 3. General-Purpose Modules Are Deeper
- Implement modules in a "somewhat general-purpose" fashion
- Interface should be general enough to support multiple uses
- Implementation should reflect current needs
- Benefits:
  - Simpler interfaces
  - Better information hiding
  - Potential for reuse
- Questions to ask:
  - What is the simplest interface covering all current needs?
  - In how many situations will this method be used?
  - Is this API easy to use for current needs?

### 4. Different Layer, Different Abstraction
- Each layer should provide a different abstraction from layers above/below
- **Pass-through methods** (RED FLAG): Method does nothing but call another with similar signature
  - Indicates confused division of responsibility
  - Solutions: Expose lower layer directly, redistribute functionality, or merge classes
- **Decorators** - use sparingly, they're often shallow
  - Before creating decorator, consider adding to underlying class
  - Or merge with existing decorator
  - Or implement as standalone class

### 5. Pull Complexity Downward
- **Most modules have more users than developers** - better for developers to suffer than users
- Handle complexity internally rather than exposing it to users
- More important for module to have simple interface than simple implementation
- Configuration parameters are often complexity pushed upward (avoid when possible)
  - Only use when users can determine better values than the module
  - Provide sensible defaults

## Specific Design Techniques

### 6. Define Errors Out of Existence
- Best way to handle exceptions: eliminate them through better API design
- Examples:
  - Tcl `unset`: Should succeed whether variable exists or not
  - Unix file deletion: Allows deleting open files (delays actual deletion)
  - Java substring: Should handle out-of-range indices gracefully
- Other techniques when errors can't be eliminated:
  - **Exception masking**: Handle at low level so higher levels unaware
  - **Exception aggregation**: Handle many exceptions with single handler
  - **Just crash**: For errors that are rare and unrecoverable
- **Design special cases out of existence**:
  - Empty selection in text editor = selection with same start/end position
  - Eliminates if statements and special case code

### 7. Better Together or Better Apart?
Combine pieces when they:
- Share information (same knowledge used in both)
- Are used together bidirectionally
- Overlap conceptually under simple higher-level category
- Are hard to understand independently

Separate when:
- Bringing together creates duplication across layers
- Mix general-purpose and special-purpose code
- Create unnecessary dependencies

**Splitting Methods:**
- Don't split by length alone
- Good reasons to split:
  - Extract cleanly separable subtask (child understands parent, parent doesn't need child details)
  - Original method has overly complex interface doing unrelated things
- If you must flip between parent/child to understand them: **Conjoined Methods** (RED FLAG)

### 8. Design It Twice
- **Always consider multiple design alternatives** for significant decisions
- Try radically different approaches
- Compare pros/cons focusing on:
  - Simplicity of interface
  - General-purpose vs special-purpose
  - Efficiency
- The best design may combine features from multiple alternatives
- Apply at multiple levels: interfaces, implementations, system architecture
- Takes minimal extra time compared to implementation
- Even smart people can't get it right on hard problems

## Documentation Principles

### 9. Write Comments First
- **Write comments at the beginning**, not the end
- Process:
  1. Write class interface comment
  2. Write interface comments and signatures for key public methods (empty bodies)
  3. Iterate until structure feels right
  4. Write declarations and comments for instance variables
  5. Fill in method bodies, adding implementation comments as needed
- Benefits:
  - Better comments (design fresh in mind)
  - Improves system design (comments force thinking about abstractions)
  - More enjoyable (part of creative design process)

### 10. Comments Should Describe Things Not Obvious from Code
- Comments at different level of detail than code:
  - **Lower-level comments**: Add precision (units, boundary conditions, ownership, invariants)
  - **Higher-level comments**: Enhance intuition (overall purpose, simplified view)

**Interface Comments:**
- Describe abstraction, not implementation
- For classes: Overall abstraction and capabilities
- For methods: Behavior from caller perspective, arguments, return values, side effects, exceptions, preconditions
- Should be complete enough that users never read implementation

**Implementation Comments:**
- Describe what and why, not how
- Most short methods need no implementation comments
- For longer methods: Comment before each major block
- Document why code exists, especially for non-obvious bug fixes
- Focus on what variables represent, not how they're manipulated

**Red Flags:**
- Comment repeats code (RED FLAG)
- Implementation documentation in interface (RED FLAG)
- Hard to describe (RED FLAG) - suggests design problem

### 11. Comment Conventions
- Use different words in comment than in the name
- Pick conventions and follow them consistently
- Comment types:
  - Interface comments (before declarations)
  - Data structure member comments
  - Implementation comments (inside methods)
  - Cross-module comments (in central designNotes file)

## Red Flags Summary

Watch for these warning signs:

1. **Shallow Module**: Interface not much simpler than implementation
2. **Information Leakage**: Design decision reflected in multiple modules
3. **Temporal Decomposition**: Structure based on execution order, not information hiding
4. **Overexposure**: Common features require awareness of rare features
5. **Pass-Through Method**: Does nothing but call another method
6. **Repetition**: Same code appears multiple times
7. **Special-General Mixture**: Special-purpose not cleanly separated from general-purpose
8. **Conjoined Methods**: Can't understand one without understanding the other
9. **Comment Repeats Code**: No information beyond the code
10. **Implementation Documentation Contaminates Interface**: Interface docs describe implementation
11. **Hard to Describe**: Requires long comment to be complete
12. **Hard to Pick Name**: Difficult to find precise, intuitive name

## Implementation Checklist

When writing code:

**Before Starting:**
- [ ] Design it twice - consider at least 2 approaches
- [ ] Write interface comments first
- [ ] Think about what to hide vs expose

**During Design:**
- [ ] Create deep modules with simple interfaces
- [ ] Minimize information leakage
- [ ] Pull complexity downward
- [ ] Make common cases simple
- [ ] Define errors out of existence where possible
- [ ] Separate general-purpose from special-purpose code

**During Implementation:**
- [ ] Write comments before code
- [ ] Watch for red flags
- [ ] Invest time in simplification (10-20% overhead)
- [ ] Refactor when discovering design problems
- [ ] Comments describe non-obvious information at different level than code

**During Review:**
- [ ] Can interface be simplified?
- [ ] Is module deep enough?
- [ ] Any information leakage?
- [ ] Are comments complete and clear?
- [ ] Any red flags present?

## Key Quotes to Remember

- "Working code isn't enough"
- "Complexity is incremental - sweat the small stuff"
- "Define errors out of existence"
- "Different layer, different abstraction"
- "Pull complexity downward"
- "It's more important for a module to have a simple interface than a simple implementation"
- "General-purpose modules are deeper"
- "The best way to reduce bugs is to make software simpler"
- "Most modules have more users than developers, so it is better for the developers to suffer than the users"
- "Comments should describe things that aren't obvious from the code"

---

**Remember**: Strategic programming with continuous small investments in design quality produces better systems faster than tactical "just make it work" programming. The goal is to minimize complexity through deep modules, information hiding, and thoughtful abstraction.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Brendan-Brummer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Brendan-Brummer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
