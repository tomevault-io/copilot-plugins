## claudemcpcodeautomation

> - **Always read `PLANNING.md`** at the start of a new conversation to understand the project's architecture, goals, style, and constraints.

### 🔄 Project Awareness & Context
- **Always read `PLANNING.md`** at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
- **Check `TASK.md`** before starting a new task. If the task isn't listed, add it with a brief description and today's date.
- **Use consistent naming conventions, file structure, and architecture patterns** as described in `PLANNING.md`.
- **Use dotnet CLI and NuGet** for package management and project operations.

### 🧱 Code Structure & Modularity
- **Never create a file longer than 500 lines of code.** If a file approaches this limit, refactor by splitting it into separate modules, classes, or services.
- **Organize code into clearly separated layers and projects**, grouped by feature or responsibility.
  For applications this looks like:
    - `MyApp.Domain` - Domain models and business logic
    - `MyApp.Application` - Application services and use cases
    - `MyApp.Infrastructure` - Data access and external services
    - `MyApp.API` - Controllers and API endpoints (ASP.NET Core)
    - `MyApp.UI` - MAUI application for cross-platform UI
- **Use clear, consistent namespaces** following .NET naming conventions.
- **Use appsettings.json and IConfiguration** for configuration management.

### 🧪 Testing & Reliability
- **Always create unit tests for new features** (functions, classes, modules, endpoints, etc).
- **After updating any logic**, check whether existing unit tests need to be updated. If so, do it.
- **Tests should live in separate test projects** with `.Tests` suffix mirroring the main project structure.
  - Include at least:
    - 1 test for expected behavior
    - 1 edge case
    - 1 failure case
- **Use xUnit as the primary testing framework** with FluentAssertions for readable assertions.

### ✅ Task Completion
- **Mark completed tasks in `TASK.md`** immediately after finishing them.
- Add new sub-tasks or TODOs discovered during development to `TASK.md` under a "Discovered During Work" section.

### 📎 Style & Conventions
- **Use C#** as the primary language with .NET 8 or later.
- **Follow Microsoft C# coding conventions**, use nullable reference types, and format with EditorConfig.
- **Use FluentValidation for data validation** and built-in model validation for simple cases.
- Use **ASP.NET Core** for APIs, **Entity Framework Core** for ORM, and **MAUI** for cross-platform applications.
- **Write XML documentation comments for every public member**:
  ```csharp
  /// <summary>
  /// Brief summary of what this method does.
  /// </summary>
  /// <param name="parameter">Description of the parameter.</param>
  /// <returns>Description of what is returned.</returns>
  public async Task<Result> ExampleMethodAsync(string parameter)
  {
      // Implementation
  }
  ```

### 📚 Documentation & Explainability
- **Update `README.md`** when new features are added, dependencies change, or setup steps are modified.
- **Comment non-obvious code** and ensure everything is understandable to a mid-level developer in that language.
- When writing complex logic, **add inline comments** explaining the why, not just the what.

### 🧠 AI Behavior Rules
- **Workspace Safety**: When an existing project is specified via a `PROJECT_SOURCE` (GitHub URL or local path), you MUST first create a safe copy (by cloning or copying) into a `workspace` directory. All work—including file edits, builds, and tests—MUST be performed ONLY within this `workspace` directory. You are explicitly forbidden from modifying the original source code or pushing to a remote repository.
- **Never assume missing context. Ask questions if uncertain.**
- **Never hallucinate packages, libraries, or APIs** – only use known, verified packages and language features.
- **Always confirm dependencies and imports** exist before referencing them in code or tests.
- **Never delete or overwrite existing code** unless explicitly instructed to or if part of a task from `TASK.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TGabeCar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TGabeCar)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
