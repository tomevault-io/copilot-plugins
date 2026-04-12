## nexuscore

> **Global Rule: Refactoring Guidelines (Template)**

**Global Rule: Refactoring Guidelines (Template)**

**Purpose:** To guide the AI in identifying opportunities for code refactoring, suggesting appropriate refactoring techniques, and ensuring that refactoring is done safely and effectively to improve code quality without altering external behavior.

**Instructions for the AI:**
"When reviewing code or assisting with development, proactively identify opportunities for refactoring. When suggesting or performing refactoring, adhere to the principles and techniques outlined below. The primary goal of refactoring is to improve the internal structure of the code, making it easier to understand, maintain, and extend, without changing its observable behavior."

**1. Goals of Refactoring:**
    * "(User to specify, e.g.:
        * Improve code readability and understandability.
        * Reduce complexity (e.g., simplify conditional logic, break down large methods/classes).
        * Enhance maintainability and extensibility.
        * Remove duplication (DRY - Don't Repeat Yourself).
        * Improve performance (though this should often be a secondary goal after clarity, unless addressing a known bottleneck – refer to 'Performance Optimization Reminders').
        * Make code easier to test.)"

**2. When to Refactor:**
    * "(User to specify, e.g.:
        * **The Boy Scout Rule:** 'Leave the code cleaner than you found it. Make small improvements whenever you touch a piece of code.'
        * **Before Adding New Features:** 'If working on a complex or poorly structured area of code, refactor it first to make adding the new feature easier and safer.'
        * **After a Bug Fix:** 'Once a bug is fixed, refactor the surrounding code if its structure contributed to the bug or made it hard to find.'
        * **When Code Smells are Detected:** 'Proactively address identified code smells (see section below).'
        * **During Code Reviews:** 'Identify and suggest refactorings as part of the code review process.')"

**3. When NOT to Refactor (Cautionary Principles):**
    * "(User to specify, e.g.:
        * 'Avoid large-scale refactoring close to a critical deadline unless absolutely necessary to fix a severe issue.'
        * 'Do not refactor code that is poorly understood or lacks adequate test coverage, as the risk of introducing regressions is high. (Prioritize adding tests first).'
        * 'If the code is part of a very unstable or experimental module that is expected to change drastically or be discarded soon.'
        * 'Avoid refactoring external libraries or code you don't own, unless contributing upstream in a controlled manner.')"

**4. Prerequisites for Refactoring:**
    * "**Solid Test Coverage:** Before undertaking any significant refactoring, ensure that the code to be refactored has a comprehensive suite of (passing) automated tests. These tests are crucial for verifying that the refactoring has not changed the code's behavior."
    * "**Version Control:** Ensure the code is under version control and all current changes are committed before starting a refactoring session."

**5. Identifying Code Smells (Opportunities for Refactoring):**
    * "(User to list common code smells they want the AI to look for. Examples:)
        * **Duplicate Code:** Identical or very similar code found in multiple places.
        * **Long Method/Function:** A method or function that is too long and likely doing too many things.
        * **Large Class/Module:** A class or module that has too many responsibilities, fields, or methods.
        * **Feature Envy:** A method that seems more interested in a class other than the one it actually is in.
        * **Primitive Obsession:** Using primitive data types to represent domain concepts instead of creating small classes/structs.
        * **Long Parameter List:** A function or method with too many parameters.
        * **Shotgun Surgery:** A single change requires making many small changes in different classes.
        * **Data Clumps:** Groups of variables that often appear together in multiple places.
        * **Switch Statements/Type-Checking Conditionals:** Complex conditional logic based on type codes or enums that might be better handled by polymorphism.
        * **Comments Explaining Complex Code:** Comments used as a "deodorant" for code that is hard to understand (the code itself should be made clearer).
        * **Lazy Class/Speculative Generality:** A class that doesn't do enough to justify its existence or code added "just in case" it might be needed."

**6. Common Refactoring Techniques to Suggest/Apply:**
    * "(User to list preferred techniques or confirm general knowledge is acceptable. AI should know these, but preferences can be stated. Examples:)
        * **Extraction Family:** Extract Method, Extract Class, Extract Variable, Extract Superclass/Interface.
        * **Inlining Family:** Inline Method, Inline Temp.
        * **Renaming:** Rename Variable, Rename Method, Rename Class (emphasize clear, intention-revealing names).
        * **Moving Features:** Move Method, Move Field.
        * **Simplifying Conditionals:** Decompose Conditional, Consolidate Conditional Expression, Replace Nested Conditional with Guard Clauses.
        * **Making Method Calls Simpler:** Introduce Parameter Object, Preserve Whole Object, Remove Setting Method, Replace Parameter with Explicit Methods.
        * **Generalization:** Pull Up Method/Field, Push Down Method/Field, Extract Interface.
        * **Dealing with Data:** Encapsulate Field, Replace Magic Number with Symbolic Constant, Replace Array with Object."

**7. Refactoring Process:**
    * "**Small, Incremental Changes:** Refactorings should be done in small, deliberate steps."
    * "**Test After Each Step:** Run tests after each small refactoring to ensure no behavior has been broken. If tests fail, revert the last change and try a different approach or break it down further."
    * "**Do Not Mix Refactoring with Feature Addition:** Keep commits focused. A commit should either be purely for refactoring or purely for adding/changing features, not both. This makes history clearer and regressions easier to track."

"When suggesting refactorings, clearly state the code smell being addressed and the refactoring technique(s) being proposed. Explain the benefits of the suggested change. If I ask for refactoring help, guide me through the process using these principles."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Chronovyan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
