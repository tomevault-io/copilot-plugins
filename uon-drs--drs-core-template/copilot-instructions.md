## drs-core-template

> Run this once when you first clone the template, before doing anything else.

# TemplateApp — Agent Instructions

## Project Setup

Run this once when you first clone the template, before doing anything else.

1. Ask the user for the following values:
   - Project name in **PascalCase** (e.g. `AcmePlatform`) — replaces `TemplateApp`
   - Project name in **lowercase, no spaces** (e.g. `acmeplatform`) — replaces `templateapp`
   - ADO service connection name — replaces `templateapp-azure-sc`
   - Keycloak hostname — replaces `keycloak.example.com`
   - Azure region — replaces `australiaeast`
   - Resource group names for uat and prod — replaces `templateapp-uat-rg` / `templateapp-prod-rg`
   - New git remote URL for the project repository

2. Perform **case-sensitive** find-and-replace across the entire repo for each value above.

3. Rename files and update references:
   - Rename `backend/TemplateApp.sln` → `<YourName>.sln`
   - Rename `backend/src/TemplateApp.Api/` → `backend/src/<YourName>.Api/`
   - Rename `backend/src/TemplateApp.Api/TemplateApp.Api.csproj` → `<YourName>.Api.csproj`
   - Rename `backend/tests/TemplateApp.Api.Tests/` and its `.csproj` accordingly
   - Update the project paths inside the `.sln` file to match the new names

4. Confirm all substitutions were made and list every changed file.

5. Update `README.md` with the new project name and ask the user for a one-paragraph description of the project, then write it in.

6. Remove the **Project Setup** and **Before Any Other Work** sections from both `CLAUDE.md` and `AGENTS.md` — they are one-time instructions and should not remain in the adapted project.

7. Clean up git history so the new project starts fresh:
   - Stage all current files and squash the entire template history into a single initial commit:
     ```bash
     git checkout --orphan fresh-start
     git add -A
     git commit -m "Initial commit"
     git branch -D main
     git branch -m main
     ```
   - Update the remote origin to the new project URL and push:
     ```bash
     git remote set-url origin <new-remote-url>
     git push -u origin main
     ```

---

## Before Any Other Work

Check that `TemplateApp` and `templateapp` no longer appear in the codebase. If they do, stop and ask the user to run the Project Setup above before continuing.

---

## Code Style

Follow `.editorconfig` at the repo root for all formatting decisions — indent size, line endings, charset, C# naming conventions, and C# language feature preferences. Do not override or duplicate its rules.

---

## Documentation

### Backend (C#)

All public classes, methods, and properties must have XML doc comments:

```csharp
/// <summary>Brief description of what this does.</summary>
/// <param name="paramName">What the parameter is.</param>
/// <returns>What is returned.</returns>
```

Internal and private members: add a comment only when the intent is not obvious from the name and types alone.

### Frontend (TypeScript)

All exported functions, components, and hooks must have JSDoc comments:

```ts
/**
 * Brief description of what this does.
 * @param paramName - what the parameter is
 * @returns what is returned
 */
```

Inline comments for non-obvious logic only — do not comment self-evident code.

---

## Testing

### Backend

- **Framework:** xUnit
- **Location:** `backend/tests/TemplateApp.Api.Tests/` (rename after project setup)
- **Attributes:** `[Fact]` for single cases, `[Theory]` + `[InlineData]` for parameterised cases
- **Naming:** `MethodName_ExpectedBehaviour_Condition`
- **Pattern:** use `WebApplicationFactory` integration tests for controllers and middleware; plain unit tests for pure logic classes

### Frontend

- **Framework:** Vitest + React Testing Library (set up if not already present)
- **Location:** colocate tests alongside source files as `ComponentName.test.tsx`, or group in a `__tests__/` folder in the same directory
- **Naming:** `describe` block per component or function; `it('does x when y')`

---

## Commits

After making code changes, create commits that group changes in functionally meaningful ways. Each commit should represent one coherent unit of work (e.g. a new feature, a bug fix, a refactor) — not one commit per file, and not all changes lumped into one commit.

---
> Source: [uon-drs/drs-core-template](https://github.com/uon-drs/drs-core-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
