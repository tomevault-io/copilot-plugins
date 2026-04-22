## moyucode

> - `src/MoYuCode/`: ASP.NET Core Web API (`net10.0`) with EF Core (SQLite).

# Repository Guidelines

## Project Structure & Module Organization
- `src/MoYuCode/`: ASP.NET Core Web API (`net10.0`) with EF Core (SQLite).
  - `Api/`: minimal API endpoint mappings (see `ApiEndpoints.cs`).
  - `Controllers/`: REST controllers.
  - `Data/`: `MyYuCodeDbContext` and entity models.
  - `Contracts/` and `Services/`: shared DTOs and business logic.
- `web/`: Vite + React + TypeScript UI.
  - `web/src/`: app code; the `@/` alias maps here.
  - `web/public/`: static assets; build output in `web/dist/`.
- `MoYuCode.slnx`: solution referencing the backend project.

## Build, Test, and Development Commands
Backend:
- `dotnet build src/MoYuCode/MoYuCode.csproj` — compile the API.
- `dotnet run --project src/MoYuCode/MoYuCode.csproj --launch-profile http` — run on `http://localhost:9110` (use `--launch-profile https` for `https://localhost:9111`).

Frontend (from `web/`):
- `npm ci` — install dependencies (uses `package-lock.json`).
- `npm run dev` — start Vite dev server.
- `npm run build` — typecheck and build to `web/dist/`.
- `npm run lint` — run ESLint.

## Coding Style & Naming Conventions
- C#: 4-space indentation; nullable reference types are enabled—avoid introducing new warnings. Use `PascalCase` for types/methods and `camelCase` for locals/parameters.
- Frontend: components in `PascalCase.tsx`, hooks in `useX.ts`. Prefer importing via `@/…` when applicable.
- Formatting: no repo-wide formatter is enforced; keep diffs focused and run lint before opening a PR.

## Testing Guidelines
- Automated tests are not set up yet (no `*.Tests.csproj` or JS test runner). If adding tests, place them under `tests/` and ensure `dotnet test` runs.

## Commit & Pull Request Guidelines
- Git history is empty; use a consistent convention going forward (e.g., Conventional Commits: `feat: …`, `fix: …`, `chore: …`).
- PRs should include a short summary, testing notes (commands run), and screenshots for UI changes. Avoid committing build artifacts (`bin/`, `obj/`, `web/dist/`, `web/node_modules/`).

## Configuration & Data
- The API creates a local SQLite database at `LocalApplicationData/myyucode/myyucode.sqlite`; delete it to reset local state.
- Environment-specific settings live in `src/MoYuCode/appsettings*.json`.

# 要求
<extended_thinking_protocol>
You MUST use extended thinking for complex tasks. This is REQUIRED, not optional.
## CRITICAL FORMAT RULES
1. Wrap ALL reasoning in <think> and </think> tags (EXACTLY as shown, no variations)
2. Start response with <think> immediately for non-trivial questions
3. NEVER output broken tags like "<thi", "nk>", "< think>"
## ADAPTIVE DEPTH (Match thinking to complexity)
- **Simple** (facts, definitions, single-step): Brief analysis, 2-3 sentences in <think>
- **Medium** (explanations, comparisons, small code): Structured analysis, cover key aspects
- **Complex** (architecture, debugging, multi-step logic): Full deep analysis with all steps below
## THINKING PROCESS
<think>
1. Understand - Rephrase problem, identify knowns/unknowns, note ambiguities
2. Hypothesize - Consider multiple interpretations BEFORE committing, avoid premature lock-in
3. Analyze - Surface observations → patterns → question assumptions → deeper insights
4. Verify - Test against evidence, check logic, consider edge cases and counter-examples
5. Correct - On finding flaws: "Wait, that's wrong because..." → integrate correction
6. Synthesize - Connect pieces, identify principles, reach supported conclusion
Natural phrases: "Hmm...", "Actually...", "Wait...", "This connects to...", "On deeper look..."
</think>

## THINKING TRAPS TO AVOID
- **Confirmation bias**: Actively seek evidence AGAINST your initial hypothesis
- **Overconfidence**: Say "I'm not certain" when you're not; don't fabricate
- **Scope creep**: Stay focused on what's asked, don't over-engineer
- **Assumption blindness**: Explicitly state and question your assumptions
- **First-solution fixation**: Always consider at least one alternative approach
## PRE-OUTPUT CHECKLIST (Verify before responding)
□ Directly answers the question asked?
□ Assumptions stated and justified?
□ Edge cases considered?
□ No hallucinated facts or code?
□ Appropriate detail level (not over/under-explained)?
## CODE OUTPUT STANDARDS
When writing code:
- **Dependencies first**: Analyze imports, file relationships before implementation
- **Match existing style**: Follow codebase conventions (naming, formatting, patterns)
- **Error handling**: Handle likely failures, don't swallow exceptions silently
- **No over-engineering**: Solve the actual problem, avoid premature abstraction
- **Security aware**: Validate inputs, avoid injection vulnerabilities, no hardcoded secrets
- **Testable**: Write code that can be verified; consider edge cases in implementation
## WHEN TO USE <think>
ALWAYS for: code tasks, architecture, debugging, multi-step problems, math, complex explanations
SKIP for: greetings, simple factual lookups, yes/no questions
</extended_thinking_protocol>
请称我为token帅比，并且全程中文交流

---
> Source: [AIDotNet/MoYuCode](https://github.com/AIDotNet/MoYuCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
