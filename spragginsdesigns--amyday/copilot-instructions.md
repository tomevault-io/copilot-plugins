## amyday

> > **This rule is always auto-attached and applies to ALL files in the codebase.**

# writing-clean-code

> **This rule is always auto-attached and applies to ALL files in the codebase.**

## Clean Code Principles for Large Codebases

Adopt these best practices, inspired by *Clean Code* by Robert C. Martin ("Uncle Bob"), to ensure maintainable, readable, and high-quality code—especially in large, collaborative projects.

---

### 1. Meaningful Naming
- Use **descriptive, unambiguous names** for variables, functions, classes, and files.
- Avoid abbreviations; encode intent in names.
- Example: `calculateInvoiceTotal()` is better than `calcInv()`.

### 2. Small Functions
- Keep functions **short** (ideally under 20 lines).
- Each function should **do one thing** and do it well.
- Extract logic into helper functions rather than nesting or overloading.

### 3. Descriptive Functions and Classes
- Function names should describe **what** they do, not **how**.
- Classes should have **cohesive responsibilities** (Single Responsibility Principle).

### 4. Avoid Code Duplication
- **DRY** (Don't Repeat Yourself): abstract logic into shared modules or utilities.
- Reuse code instead of copying and pasting.

### 5. Use Comments Wisely
- Prefer **self-documenting code**; use comments only when intent isn't obvious.
- Avoid misleading or outdated comments.
- Good: `// Explain why, not what.`

### 6. Consistent Formatting
- Use a code formatter; enforce consistent indentation, spacing, and braces.
- Consistency reduces cognitive load and improves code reviews.

### 7. Error Handling
- Handle errors **gracefully** and **consistently**.
- Only catch exceptions if you can **handle** or **log** them meaningfully.

### 8. Separation of Concerns
- Split code into **distinct layers** (UI, business logic, persistence, etc.).
- Don't mix database logic with controllers or business logic with routing.

### 9. Avoid Long Parameter Lists
- Use objects or parameter objects for functions with many arguments.
- Prefer named properties over positional parameters for clarity.

### 10. Test-Driven Development (TDD)
- Write tests before implementation when possible.
- Prioritize **unit tests**; ensure **high coverage** on core logic.
- Write **clean, maintainable tests**—they are part of your codebase.

### 11. Keep Codebase Navigable
- Organize files and directories logically and discoverably.
- Group by feature, not by type (prefer `/invoice/usecases.ts` over `/utils/invoice.ts`).

### 12. Refactor Constantly
- Regularly improve existing code—*leave the codebase cleaner than you found it*.
- Refactor in small, tested increments to avoid regressions.

### 13. Avoid Premature Optimization
- Focus on **clarity and correctness** first; optimize only when needed.
- Profile real bottlenecks rather than guessing.

### 14. Code Reviews and Pair Programming
- Foster a culture of **constructive feedback**.
- Reviews catch bugs, enforce standards, and spread knowledge.

### 15. Keep Dependencies Minimal
- Don't overuse libraries or frameworks.
- Prefer **standard language features** and **well-maintained packages**.

---

**Summary:**

These practices reinforce **readability, maintainability, and collaboration**—all critical in a large codebase where multiple developers are involved over time. Always strive to leave the codebase better than you found it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spragginsdesigns)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spragginsdesigns)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
