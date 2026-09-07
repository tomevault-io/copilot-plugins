## liveleaf

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Engineering Principles & Mentorship Style

## 1. Engineering Principles
* **Production-Grade Simplicity:** Write code that is easy to understand, debug, and extend. Prefer simple solutions until complexity is justified.
* **No Premature Abstraction:** A feature should become real before its architecture becomes generic.
* **Clean Fields:** Every database column and code parameter must have a single, clear responsibility.
* **Optimize for Maintainability:** Write self-documenting code with minimal comments. Explain concepts in the chat, not through comments.

## 2. Mentorship Rules
* **No Vibe Coding:** Do not copy/paste code or suggest workarounds. Explain the "why" behind every design choice.
* **Concept Introduction Flow:**
  1. Explain the real engineering problem.
  2. Explain why existing/default solutions are insufficient.
  3. Explain why the proposed solution is chosen.
  4. Explain the implementation details.
  5. Discuss tradeoffs (e.g. Startup vs. Enterprise approach).
* **Verify, Don't Guess:** Do not assume framework behavior. Check source code or documentation when dealing with dependency boundaries.

## 3. Code Comments

* **Default to No Comments:** Do not add comments unless they communicate information that cannot be made obvious through code alone. Prefer improving the code over explaining it.
* **Treat Comments as Production Code:** Every comment must provide long-term value. Comments should survive refactors and remain accurate.
* **No AI-Style Narration:** Never add comments that simply describe what the code is doing (e.g. `// Update the document`). If the code is self-explanatory, write no comment.
* **Comment the "Why", Not the "What":** Comments are reserved for non-obvious business rules, architectural decisions, performance optimizations, framework quirks, security considerations, or intentional tradeoffs.
* **No Tutorial or Teaching Comments:** This is a production repository, not educational material. Explain concepts in the chat, never through inline comments.
* **Delete Before Adding:** If a comment becomes unnecessary after improving names or structure, remove the comment instead of keeping redundant documentation.

## 4. Library & Framework Consistency

* **Prefer Existing Tools:** Before introducing a new library, first determine whether the current stack already provides a clean, production-grade solution.
* **Optimize for Consistency:** Keep implementation patterns consistent across the codebase. If the project uses a primary library or framework (e.g. shadcn/ui, TanStack Query, React Hook Form, Zod), prefer its conventions over mixing multiple libraries that solve the same problem.
* **Avoid Dependency Creep:** Do not introduce additional dependencies for small conveniences or isolated use cases. Every dependency increases maintenance, bundle size, upgrade complexity, and long-term ownership.
* **Recommend Before Replacing:** If another library would genuinely improve the project (better performance, maintainability, accessibility, developer experience, or long-term support), do not introduce it directly. Explain:
  1. Why the current solution is insufficient.
  2. Why the new library is a better fit.
  3. The tradeoffs of adopting it.
  4. Wait for approval before changing the project's technology stack.
* **Respect Existing Architecture:** New code should feel like it belongs in the repository. Prioritize consistency with established patterns over personal preference or newer alternatives.

## 5. Architecture & Design

* **Design Before Coding:** Before implementing a feature, identify the domain model, responsibilities, and data flow. Avoid writing code until the architecture is clear.
* **Single Responsibility:** Every function, component, hook, and module should have one primary responsibility. Split code when responsibilities begin to diverge.
* **Prefer Composition Over Configuration:** Build small composable pieces instead of highly configurable abstractions with numerous flags and options.
* **Follow Existing Patterns:** When implementing a new feature, first understand how similar features are structured in the repository and remain consistent unless there is a compelling reason to improve the pattern.

## 6. API & Data Flow

* **Validate at Boundaries:** Validate all external input (API requests, forms, route params, webhooks, etc.) at the application's boundaries.
* **One Source of Truth:** Never duplicate information that can be derived. Prefer deriving TypeScript types from Zod schemas, Prisma selects, or other existing sources instead of manually maintaining duplicate types.
* **Keep Layers Separate:** UI components should not contain business logic. Business logic belongs in services, actions, or dedicated domain modules.
* **Explicit Data Flow:** Prefer predictable top-down data flow. Avoid hidden global state unless there is a clear architectural reason.

## 7. Performance & Scalability

* **Measure Before Optimizing:** Do not introduce caching, memoization, virtualization, or other performance optimizations without identifying an actual bottleneck.
* **Start Simple:** Design for today's requirements while leaving room for future growth. Avoid enterprise-scale architecture for startup-scale problems.
* **Optimize Database Access:** Prefer solving performance problems with efficient queries and proper indexing before adding caching layers.

## 8. Error Handling

* **Fail Explicitly:** Do not silently swallow errors. Errors should either be handled meaningfully or allowed to propagate.
* **Actionable Errors:** Error messages should help developers understand what went wrong and how to resolve it.
* **Consistent Error Strategy:** Follow one consistent error-handling approach across the project instead of mixing patterns.

## 9. Code Changes

* **Minimal Surface Area:** Change only the code required for the requested feature or fix. Avoid unrelated refactors unless explicitly requested.
* **Protect Existing Behavior:** Preserve backwards compatibility unless a breaking change is intentional and discussed.
* **Refactor with Purpose:** Every refactor should improve readability, maintainability, correctness, or performance—not simply reflect personal preference.

---
> Source: [amanhuh/liveleaf](https://github.com/amanhuh/liveleaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
