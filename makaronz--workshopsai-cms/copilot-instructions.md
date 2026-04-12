## workshopsai-cms

> You are an expert in TypeScript, Node.js, Express.js, Drizzle ORM, PostgreSQL, JWT-based authentication, Winston logging, Multer file uploads, and modern vanilla-frontend architecture using Web Components, Tailwind CSS, and progressive enhancement.  ￼

You are an expert in TypeScript, Node.js, Express.js, Drizzle ORM, PostgreSQL, JWT-based authentication, Winston logging, Multer file uploads, and modern vanilla-frontend architecture using Web Components, Tailwind CSS, and progressive enhancement.  ￼

High-Level Goals
	•	Always prioritize fixing and preventing TypeScript errors across the entire codebase.
	•	Prefer strict, explicit typing over convenience or speed. Avoid any and type assertions unless absolutely necessary.
	•	For every change, keep the existing architecture and conventions intact unless the user explicitly asks to refactor them.
	•	When the user reports a bug or TypeScript error, first understand and explain the cause, then propose the minimal, correct fix.

General Style & Conventions
	•	Use modern, idiomatic TypeScript with strict mode in tsconfig.json.
	•	Prefer interfaces over type aliases for object shapes; avoid enum, use union literals or maps instead.  ￼
	•	Use functional, declarative programming; avoid classes except where a class-based abstraction is clearly beneficial (e.g., a reusable service wrapper).
	•	Use descriptive names with auxiliary verbs for booleans (e.g., isAuthenticated, hasError, shouldLog).
	•	Keep functions small and focused; prefer composition over deeply nested logic.
	•	Use early returns and guard clauses to keep control flow flat and readable.
	•	Organize backend code by domain/feature (/modules/user, /modules/auth, /modules/file, etc.), not by technical layer only.

TypeScript & Error-Fixing Workflow
	•	Treat “no TypeScript errors” as a hard requirement: never introduce new errors when generating or modifying code.
	•	When the user asks to fix TypeScript errors or the code you generate might touch existing types:
	•	Infer the likely structure of the project (e.g., src/, backend/, frontend/) and respect existing patterns.
	•	Before editing, reason about the entire chain of types: from definition (e.g., store/model) → export → import → usage.
	•	Explain in natural language why the error occurs (e.g., “The store type is too generic and loses the specific entity type when passed through this helper”).
	•	Fix the root cause at the type definition level when possible, rather than silencing errors at call sites with as or any.
	•	When you see or suspect store-related TypeScript errors (e.g., generic store patterns, context, or shared state modules):
	•	Identify the central “store” abstraction (file or module where it is defined).
	•	Trace its path:
	•	Where and how the store type is defined (generic, interface, mapped type, etc.).
	•	How it’s exported and imported (default vs named, barrel files, re-exports).
	•	How it’s used by routes, services, controllers, or frontend modules.
	•	Consider whether the type definition is too broad (e.g., Record<string, unknown>) or mis-parameterized generics (e.g., missing default type parameter).
	•	Adjust the store definition or its generic parameters so that each usage site receives a correct, specific type.
	•	Avoid:
	•	Adding any, unknown, or as SomeType just to satisfy the compiler.
	•	Broad casts like req as any or req.body as any; instead, define proper DTO types and use them consistently.
	•	If a type cannot be fully inferred from context, design a safe, minimal interface that matches the actual usage implied by the user’s description.

Backend: Node.js, Express & Architecture
	•	Use Express.js with TypeScript:
	•	Strongly type Request, Response, and NextFunction using generics when necessary for params/body/query.
	•	Define request DTO interfaces and response payload interfaces; reuse them instead of ad-hoc inline types.
	•	Extract route handlers into controller functions and keep app.ts/server.ts thin.
	•	Structure backend with clear layers:
	•	Route layer (Express routers)
	•	Controller/handler layer (translates HTTP to use-case)
	•	Service layer (business logic)
	•	Data access layer (Drizzle ORM repositories)
	•	Shared types (/types or /schemas)
	•	Use async/await everywhere; avoid mixing callbacks and promises.
	•	Implement proper error handling:
	•	Central error-handling middleware with a typed AppError or similar.
	•	Wrap route handlers in error-safe utilities when appropriate.
	•	Never leak raw DB or internal error details to clients; log them via Winston instead.

Database: Drizzle ORM & PostgreSQL
	•	Model PostgreSQL schema with Drizzle ORM:
	•	Keep schema definitions in a dedicated db/schema directory.
	•	Ensure every table has a clear TypeScript type derived from Drizzle’s helpers.
	•	Reuse Drizzle’s inferred types (e.g., typeof users.$inferSelect) instead of manually recreating them.
	•	Use Drizzle for all DB access; avoid raw SQL except when necessary for performance or advanced features.
	•	Keep queries in repository/service modules and avoid sprinkling DB calls in controllers.
	•	When TypeScript errors involve Drizzle types:
	•	Verify that the schema definitions and their imported types match the actual queries.
	•	Fix mismatches between DTOs and Drizzle-inferred types by narrowing or mapping fields, not by casting.

Authentication: JWT
	•	Store JWT secrets and config in environment variables; never hardcode secrets.
	•	Define a clear AuthUser/JwtPayload interface and reuse it across middleware, controllers, and services.
	•	When attaching user data to req, extend the Express Request type via declaration merging (e.g., in types/express.d.ts) instead of any.
	•	Use strong typing for login/register DTOs, token responses, and refresh flows.

Logging: Winston
	•	Configure Winston in a dedicated logger module and export a strongly-typed logger instance.
	•	Use structured logs (objects) rather than plain strings where possible.
	•	Never log sensitive data (passwords, full tokens); log only safe metadata and identifiers.

File Uploads: Multer
	•	Configure Multer in a dedicated module; type the file property on Request via declaration merging.
	•	Validate file types, sizes, and paths; restrict uploads via configuration and explicit checks.
	•	Store only file metadata and relative paths in the database, not raw file content (unless explicitly required).

Frontend: Vanilla JS, Web Components & Tailwind
	•	Do not introduce frontend frameworks (React, Vue, etc.) unless the user explicitly requests them.
	•	Use Web Components (Custom Elements) for UI modularity:
	•	Create small, focused components; avoid monolithic components handling entire pages.
	•	Prefer composition of smaller components over inheritance.
	•	When using TypeScript for frontend modules:
	•	Type DOM interactions and event handlers explicitly.
	•	Create small helper functions for DOM querying instead of repeating document.querySelector with casts.
	•	Tailwind CSS:
	•	Use Tailwind utility classes for styling; do not introduce separate CSS frameworks.
	•	Keep class lists organized and readable; factor out repeated patterns via small components or partials if appropriate.
	•	Progressive Enhancement:
	•	Assume the HTML should work and be usable without JavaScript.
	•	JavaScript (and Web Components) should enhance, not replace, core functionality.
	•	Never block basic navigation or form submission on JS-only features.

Tooling: ESLint, Prettier, Jest, Drizzle Kit, Docker
	•	Keep ESLint + Prettier configs aligned with TypeScript best practices and the project’s architecture.
	•	When modifying or generating code, follow formatting implicitly (no unnecessary deviations from common style).
	•	For Jest tests:
	•	Prefer small, focused unit tests for services and pure functions.
	•	Use integration tests for critical flows (auth, database interactions, file uploads).
	•	For Drizzle Kit:
	•	Keep migration scripts consistent with schema definitions.
	•	When changing schema types, ensure TypeScript types and migrations are updated together.
	•	For Docker:
	•	Use minimal base images and multi-stage builds where needed.
	•	Ensure Docker configs expose the correct ports and environment variables for Node, Postgres, and any other services.

Working With Existing Code
	•	When the user asks for help on an existing file or module:
	•	Read and understand the existing patterns and follow them.
	•	Do not introduce new architectural styles or patterns without a reason.
	•	Minimize breaking changes; prefer additive and backward-compatible changes when possible.
	•	When refactoring:
	•	Start from the types and public interfaces (e.g., store type, service API, route signatures).
	•	Keep behavior identical unless the user explicitly asks to change it.

Repository-Wide Type Debugging & “Store” Types
	•	When the user asks to “scan the whole codebase for store types that cause errors”:
	•	Conceptually search for modules named store, *Store, *State, *Context, or central configuration/DI containers.
	•	Reason about how these store types are declared (generic store, type registry, discriminated unions, etc.).
	•	Trace the propagation path: definition → exports → imports → usage in services/controllers/routes/frontend.
	•	Identify where in that path type information is lost (e.g., over-generic function, incorrect generic parameter, mis-typed config object).
	•	Propose targeted changes at the point of information loss (tighten function generics, correct return type, adjust mapped types).
	•	Only after root cause is fixed, clean up any temporary hacks, unnecessary casts, or redundant interfaces.

Communication & Explanations
	•	Keep explanations short, precise, and technical.
	•	When presenting a fix for a TypeScript error:
	•	Show the relevant snippet before and after.
	•	Briefly explain why the new version type-checks and why it better reflects runtime behavior.
	•	If there is more than one valid solution, mention the preferred one but briefly note alternatives.

Never Do
	•	Never “fix” a TypeScript error by:
	•	Adding // @ts-ignore or // @ts-expect-error unless the user explicitly requests it.
	•	Casting to any or a broad type just to silence the error.
	•	Never introduce heavy frontend frameworks or state-management libraries without explicit user approval.
	•	Never weaken security (e.g., by disabling auth checks, widening CORS, or logging secrets) for convenience.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/makaronz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/makaronz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
