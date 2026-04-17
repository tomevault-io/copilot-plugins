## devtools-security-lab

> You are working on **devtools-engine**, a multi-service AI development platform. The codebase spans Python (FastAPI), Node.js (Express/TypeScript), Go (Gin), Java (Spring Boot), C# (ASP.NET Core), and Rust.

# DevTools Engine — Cursor Project Conventions

## Project Context

You are working on **devtools-engine**, a multi-service AI development platform. The codebase spans Python (FastAPI), Node.js (Express/TypeScript), Go (Gin), Java (Spring Boot), C# (ASP.NET Core), and Rust.

## Code Style

- **Python**: PEP 8, type hints, async/await for I/O-bound code
- **TypeScript**: Strict mode, ESLint, Prettier
- **Go**: `gofmt`, standard naming conventions
- **Java**: Google Java Style or project conventions
- **C#**: Microsoft conventions, nullable reference types
- **Rust**: `rustfmt`, `clippy`-compliant

## Preferred Frameworks

- Python: FastAPI, Pydantic
- Node.js: Express, TypeScript
- Go: Gin
- Java: Spring Boot
- C#: ASP.NET Core
- Rust: Tokio, clap, serde

## Testing Requirements

- Write unit tests for new business logic
- Add integration tests for new API endpoints
- Use dependency injection to enable mocking
- Prefer meaningful assertions over implementation-coupled tests

## Patterns

- Use async/await where applicable
- Apply dependency injection for services
- Keep configuration external (env vars, config files)
- Log with structured format and correlation IDs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Testing-itay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
