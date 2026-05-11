## toolkami

> - Run program: `uv run main.py`

# Trinket


## Build & Test

- Run program: `uv run main.py`

## Architecture Overview

- Basic UV Python project

## Conventions & Patterns

### Organize things
Organizing code well makes it easy to navigate, maintain, and extend.

- Organize code logically
  - Group related functions and classes together
  - Place high-level abstraction before low-level details
- Function
  - Keep function interfaces simple by limiting parameters and prefer returning simple types
  - If a function is only called from a single place, consider in-lining it
  - If a function is called from multiple places, see if it is possible to arrange for the work to be done in a single place, perhaps with flags, and in-line that
  - If there are multiple versions of a function, consider making a single function with more, possibly defaulted, parameters
  - If the work is close to purely functional, with few references to global state, try to make it completely functional
- Object
  - Initialize large structures or objects directly where they are declared. In-place construction avoids unnecessary copying or moving of data.
- Variable
  - Declare variables close to their usage and within the smallest necessary scope

### Control and limits
Predictable control flow and bounded system resources are essential for safe execution.

- Centralize control flow
  - Parent function controls flow by containing all switch and if statements, while maintaining state
    - Practice single assignment. Avoid reassigning or update a variable outside of true iterative calculations in loops.
  - Leaf functions should be purely functional with non-branching logic
    - In C/C++, making almost every variable `const` at initialization is good practice.
- Don't exceed more than 3 levels of indentation. It can be reduced with:
  - Extraction - pull out part of the function into its own function
  - Inversion: flipping conditions and switching to an early return
- Set fixed limits: Set explicit upper bounds on loops, queues, and other data structures. It prevents infinite loops and uncontrolled resource use, following the **fail-fast** principle. This approach helps catch issues early and keeps the system stable.

### Naming things
Get the nouns and verbs right. Great names capture what something is or does and create a clear, intuitive model.

- Use descriptive and meaningful names for variables, functions, and files.
- Avoid abbreviations unless it is widely accepted and clear (e.g., ID, URL)
- Append units or qualifiers to variable names, placing them in descending order of significance (e.g., "latency_ms_max" instead of "max_latency_ms")
- Use comments to explain why decisions were made, not just what the code does.
- Write comments as complete sentences with correct punctuation and grammar.

---
> Source: [aperoc/toolkami](https://github.com/aperoc/toolkami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
