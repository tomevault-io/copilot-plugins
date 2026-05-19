## findduplicates

> This project is a high-performance duplicate file detection tool.

# AGENTS.md

## Project Purpose
This project is a high-performance duplicate file detection tool.

Primary goals:
- Correctness of duplicate detection
- Low memory usage
- Predictable and scalable performance
- Cross-platform compatibility (Windows + Linux)

Secondary goals:
- Clean and maintainable architecture
- Observability and structured logging
- Extensibility without unnecessary complexity

Do not introduce complexity unless there is a clear and measurable benefit.

---

## Architecture Rules
- Controllers/UI must remain thin
- Business logic belongs in Services
- File system access must be abstracted behind interfaces
- Hashing implementations must be injectable
- Avoid static mutable state
- Avoid service locator patterns
- Prefer composition over inheritance

---

## Approved Technologies

### Backend
- .NET 8
- Serilog
- xUnit

### Frontend
- React
- TypeScript
- react-query

### Infrastructure
- Docker / Docker Compose
- Linux containers

### Do NOT introduce
- MediatR
- Redux (unless explicitly required)
- Autofac or new DI frameworks
- New ORMs or data access abstractions without approval

---

## Coding Standards

### C# Backend
- Use async/await consistently
- Propagate CancellationToken through all async operations
- Prefer immutable DTOs
- Enable nullable reference types
- Avoid Task.Run in ASP.NET / IO-bound code
- Avoid blocking calls (.Result, .Wait)

### React Frontend
- Use functional components only
- Use hooks instead of class components
- Prefer react-query for server state
- Keep components small and focused

---

## Performance Constraints
- Never load entire large files into memory
- Prefer streaming APIs for file processing
- Avoid excessive allocations in hot paths
- Use bounded parallelism (never unbounded Task creation)
- Avoid LINQ in performance-critical loops if it introduces allocations
- Measure performance before optimizing

---

## Concurrency Rules
- Never create unbounded parallel tasks
- Prefer Parallel.ForEachAsync with explicit limits
- Always respect CancellationToken
- File scanning must be interruptible
- Avoid shared mutable state unless properly synchronized

---

## Logging and Observability
- Use Serilog structured logging only
- Do NOT use Console.WriteLine
- Include contextual metadata in logs
- Avoid logging inside tight loops unless necessary
- Log errors with enough detail for debugging

---

## Testing Requirements
Required coverage:
- File hashing logic
- Duplicate grouping logic
- Cancellation behavior
- Edge cases in filesystem traversal

Testing principles:
- Prefer deterministic tests
- Use isolated unit tests
- Use temporary directories for IO tests
- Avoid flaky or timing-dependent tests

---

## Pull Request Guidelines
- Keep changes focused and small
- Avoid unrelated refactoring
- Preserve existing architectural patterns
- Add or update tests when behavior changes
- Explain tradeoffs in PR descriptions when relevant

---

## High-Risk Areas
Be extra careful when modifying:
- Concurrent scanning logic
- Hash computation pipeline
- File deletion or mutation logic
- Symlink handling
- Path normalization and OS-specific behavior

---

## Agent Workflow Rules
Before making changes:
1. Understand existing patterns in the codebase
2. Reuse existing abstractions instead of creating new ones
3. Avoid introducing new dependencies
4. Keep changes minimal and scoped

When uncertain:
- Prefer conservative changes
- Do not invent architecture
- Ask for clarification instead of guessing

---

## Preferred Patterns

### Good patterns:
- Dependency injection
- Streaming file processing
- Small, focused services
- Explicit boundaries between layers

### Avoid:
- Large utility classes
- Static global state
- Deep inheritance hierarchies
- Over-engineering abstractions

---

## General Principle
Simplicity and predictability are more important than cleverness or abstraction.

---
> Source: [adipuscasu/FindDuplicates](https://github.com/adipuscasu/FindDuplicates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
