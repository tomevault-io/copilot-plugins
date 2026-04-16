## test-genkit

> * `Address code imperfections (e.g., bad designs, TODOs, poor code) promptly; suggest fixes or mark areas needing attention rather than propagating them.`


# Pragmatic programming

* `Address code imperfections (e.g., bad designs, TODOs, poor code) promptly; suggest fixes or mark areas needing attention rather than propagating them.`
* `Generate the simplest solution that meets the core requirements; avoid unnecessary complexity or over-embellishment.`
* `Analyze requirements by considering context, motivations ('Who benefits?'), applicability ('When/Where would this work?'), potential issues ('Why is this a problem?'), and root causes ('Ask Five Whys') before generating code.`
* `Generate comments and documentation that are clear, concise, non-redundant (DRY), and easily maintainable.`
* `Generate comments explaining the purpose and rationale ('why') of the code, not merely repeating the implementation ('how'). Document significant decisions or trade-offs.`
* `Prioritize designs that enhance flexibility and maintainability (Easier to Change - ETC).`
* `Ensure every piece of knowledge and intent has a single, unambiguous, authoritative representation (DRY). Avoid duplication in code, data, configuration, and documentation.`
* `Generate modular and reusable functions/components.`
* `Design components to be self-contained, independent, and cohesive, minimizing dependencies (Orthogonality).`
* `Design for replaceability and abstract external dependencies (e.g., databases, services) behind interfaces to allow for future changes.`
* `When generating complex features or integrating components, consider generating a basic end-to-end skeleton first to validate structure and integration points.`
* `Generate code using terminology and structures that reflect the user's application domain.`
* `Consider the time and space complexity (Big-O) of generated algorithms and prefer efficient solutions appropriate for the expected scale.`
* `Where appropriate, use human-readable, standard, plain-text formats (e.g., JSON, YAML) for data persistence or configuration instead of proprietary binary formats.`
* `Generate code that is well-formatted, clean, and easily modifiable.`
* `When fixing errors, focus on providing the corrected code or solution.`
* `When fixing code or adding features, define the expected correct behavior (perhaps via tests or explicit criteria) first.`
* `Thoroughly analyze error messages and stack traces before attempting fixes.`
* `Assume errors originate in the application code under analysis/generation, not in stable underlying platforms or libraries, unless evidence proves otherwise.`
* `Verify assumptions explicitly in the generated code (e.g., via assertions or checks) rather than proceeding based on unverified premises.`
* `Define and check preconditions, postconditions, and invariants for functions and components to ensure robustness.`
* `Generate code that fails immediately and clearly upon detecting an impossible or inconsistent state, rather than attempting to continue with potentially corrupt data.`
* `Include assertions or runtime checks to validate conditions that should logically never be false. Ensure these checks remain active in production environments.`
* `Ensure resources (memory, files, locks, connections) are deallocated/released by the same scope or component that allocated them.`
* `Minimize the scope and lifetime of resource allocations.`
* `Generate code with minimal coupling between components.`
* `Design objects/components to perform actions involving their own state directly, rather than exposing state for external components to manipulate ('Tell, Don't Ask').`
* `Avoid chaining multiple method calls together on sequences of object accesses, unless using stable, standard library functions.`
* `Avoid using global variables or mutable global state. Minimize reliance on singleton patterns acting as global containers.`
* `Encapsulate necessary shared state or external resources (databases, services) behind well-defined APIs.`
* `Structure programs as a series of data transformations where possible, passing data between functions/stages.`
* `Pass state explicitly between functions/components rather than relying on shared mutable state.`
* `Avoid using implementation inheritance (class subclassing) for sharing code or defining types due to high coupling costs.`
* `Use interfaces or protocols to define shared types and enable polymorphism without implementation inheritance.`
* `Use composition ('Has-A') and delegation to services over implementation inheritance ('Is-A') to reuse functionality, especially from frameworks.`
* `Use mixins, traits, or equivalent language features as an alternative to inheritance for sharing functionality between classes.`
* `Externalize parameters, credentials, and environment-specific settings into configuration; do not hardcode them.`
* `Avoid designs relying on shared mutable state for concurrency; prefer immutable data or models like actors.`
* `When concurrency is required, consider using the actor model or similar patterns that rely on message passing and isolated state.`
* `Consider decoupled architectural patterns (like blackboard or message queues) for coordinating complex, asynchronous workflows.`
* `Generate code based on a deliberate plan and clear understanding; avoid relying on coincidental behavior or unverified assumptions.`
* `Analyze and consider algorithm time/space complexity (Big-O) when generating code.`
* `Identify opportunities to refactor generated or existing code for improvement (DRY, decoupling, clarity) and suggest or perform these refactorings.`
* `Consider testability (API design, low coupling) during code generation, as if writing a test for the code first.`
* `Design code modules and functions with testability in mind from the outset.`
* `Generate testable code; ideally, generate corresponding unit or property-based tests.`
* `Consider generating property-based tests based on code contracts and invariants to validate assumptions and explore state space.`
* `Generate simple code where possible. Sanitize external inputs. Minimize the number and privilege of authenticated interfaces.`
* `Prefer using up-to-date, secure libraries and dependencies.`
* `Use clear, intention-revealing names for variables, functions, classes, etc., following language/project conventions.`
* `When requirements are unclear or potentially incomplete, ask clarifying questions or suggest ways to explore the requirements further rather than making assumptions.`
* `Engage in an iterative clarification process; present interpretations or partial solutions to confirm understanding before generating the final code.`
* `Distinguish between core, invariant requirements and configurable policy; implement general mechanisms driven by external metadata/configuration for policy.`
* `Identify and operate within the actual constraints and degrees of freedom of the problem; challenge implicit or assumed constraints.`
* `Ensure generated code passes all specified tests or criteria before considering it complete.`
* `Generate tests that cover different states and edge cases, not just simple line coverage.`
* `When fixing a bug, ensure a test case covering that specific bug is generated or updated to prevent regression.`
* `Consider potential failure modes and security aspects; generate code that handles errors robustly and safely.`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markgoho)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/markgoho)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
