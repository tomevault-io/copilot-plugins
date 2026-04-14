## ci0128-2025-02-imparablesiii

> - Apply **SOLID**, **Clean Architecture**, and **Clean Code** principles in every file, function, and class.

# .cursorrules

## Global Engineering Standards
- Apply **SOLID**, **Clean Architecture**, and **Clean Code** principles in every file, function, and class.
- Prioritize **readability, maintainability, testability**, and **low coupling** over brevity.
- Always favor **composition over inheritance** unless inheritance is clearly justified.
- Ensure **single responsibility** for every class, module, or component.
- Avoid God objects, large utility classes, or multipurpose methods.
- Ensure **explicit types** in all languages whenever possible.
- Follow **Dependency Injection** for external services, data access, and cross-module interactions.
- Always validate input and handle errors explicitly.
- Use meaningful, self-explanatory naming — code should communicate intent clearly without extra comments.
- Remove all dead, redundant, or commented-out code.
- No hard-coded configuration, credentials, or constants — extract into config or environment files.

---

## C# Strict Clean Code Rules
1. Use **PascalCase** for classes, interfaces, methods, and properties.  
   Use **camelCase** for local variables and parameters.
2. Prefix **interfaces** with `I` (e.g. `IRepository`, `IService`).
3. Keep methods **under 30 lines** and ensure they perform **one single logical task**.
4. Enforce **SRP** (Single Responsibility Principle) at all levels — classes must represent one clear concept.
5. Apply **Open/Closed Principle**: code should be open to extension, closed to modification.
6. Use **Dependency Injection** via constructors or interfaces; never instantiate dependencies directly inside methods.
7. Avoid **static** classes for business logic — use instances managed by DI.
8. Handle exceptions **specifically** and log them; never catch `Exception` without purpose.
9. Replace **magic numbers or strings** with **named constants or enums**.
10. Use `var` only when the type is **clearly evident** from the right-hand side.
11. Prefer **async/await** for asynchronous operations; never block with `.Result` or `.Wait()`.
12. Always use **generic collections** (`List<T>`, `Dictionary<K,V>`) instead of non-generic.
13. For public APIs or services, define **DTOs** (Data Transfer Objects) and avoid exposing entities directly.
14. Write **unit tests** for all public-facing business logic.
15. Maintain consistent formatting: braces on new lines, 4-space indentation, clear vertical spacing.
16. Avoid regions that hide code; prefer modular, smaller files instead.

---

## JavaScript Strict Clean Code Rules
1. Always use **`const`** or **`let`**; never use `var`.
2. Follow **camelCase** for variables/functions and **PascalCase** for classes and components.
3. Use **ES modules** (`import/export`) to keep code modular.
4. Avoid global variables and implicit globals.
5. Keep functions **pure**, small, and single-purpose.
6. Prefer **arrow functions** for concise callbacks and higher-order functions.
7. Always use **strict equality (`===` / `!==`)**.
8. Handle async code with **async/await** or `.then/.catch`; never mix both styles in the same scope.
9. Avoid deeply nested callbacks or conditionals; refactor into smaller helpers.
10. Replace **magic numbers or strings** with **named constants**.
11. Validate all input and external data sources.
12. Write descriptive error messages and handle edge cases gracefully.
13. Use **linters (ESLint)** and **formatters (Prettier)** to enforce consistent style.
14. Delete unused imports, variables, or functions.
15. Each file should define **one main module/class/function**; split logic into multiple files when necessary.

---

## Vue.js (Options API) Strict Standards
- Use **Options API** for all Vue components — never use Composition API unless explicitly required.
- Component structure order:
  1. `name`
  2. `props`
  3. `data`
  4. `computed`
  5. `methods`
  6. `watch`
  7. `lifecycle hooks`
- Keep components **focused and reusable** — single responsibility per component.
- Extract reusable logic into **mixins** or **composables** if repetition appears.
- Use **PascalCase** for component names and files (e.g., `UserCard.vue`).
- Use **camelCase** for props and methods.
- All props must have **type validation** and default values where applicable.
- Use **async/await** for asynchronous operations in methods; handle rejections properly.
- Avoid direct DOM manipulation; rely on Vue reactivity.
- No inline JavaScript in templates; logic must be in the script section.
- Validate props and sanitize user input to prevent XSS or injection.
- Use **linting and Prettier** to maintain consistent style.

---

## Testing & Quality Rules
- Every use case, service, and domain class must have **unit tests**.
- Test names must express **intent and behavior** clearly.
- Avoid mocking excessively; prefer testing actual boundaries.
- Run tests automatically in CI/CD pipelines.

## Enforcement Summary
- **All generated code** must conform to these rules automatically.
- **No code completion** should produce anti-patterns such as God classes, deep nesting, hardcoded dependencies, or magic constants.
- Always refactor toward **readable, testable, modular, and SOLID-aligned** architecture.
- Prefer code that communicates intent clearly over clever or overly compact solutions.

---

Cursor must interpret this document as **project-wide persistent rules** for all completions, inline edits, and refactors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarioCordero) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
