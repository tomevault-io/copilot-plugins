## pie-shop

> This rule provides instructions for generating interview implementations for the Pie Shop assessment.

# Generate Interview Implementation

This rule provides instructions for generating interview implementations for the Pie Shop assessment.

## Activation

Use `@generate-interview` to activate this rule, then provide:
- **language**: python-fastapi, nodejs-express, csharp-dotnet, java-springboot, go-gin, ruby-rails, typescript-nestjs
- **role**: backend, fullstack, devops, security, accessibility
- **level**: junior, mid, senior, staff

Example: `@generate-interview I need a Python FastAPI implementation for a senior backend engineer`

## Before You Begin

1. **Create the interview branch:**
   ```bash
   git checkout main && git pull
   git checkout -b interview/<language>-<role>-<level>
   ```

2. **Read the specifications:**
   - `.specify/features/001-pie-shop-orchestration.md`
   - `.specify/memory/constitution.md`
   - `.specify/IMPLEMENTATION_PROMPT.md`
   - `AGENTS.md`

## Implementation Requirements

Generate production-like code with **intentional technical debt** for interview discussion:

### What to Generate
- `src/` - Application source code
- `tests/` - Unit and integration tests (mixed quality)
- `mocks/` - Mock services for external dependencies
- `ui/` - HTML/CSS/JS for order form and admin dashboard
- `docker-compose.yml` - Container orchestration
- `migrations/` - Database schema
- `README.md` - Documentation (intentionally imperfect)

### Intentional Issues to Include
- Security: No auth, hard-coded secrets, missing validation
- Resilience: No circuit breakers, basic retry without backoff
- Accessibility: Missing labels, poor contrast, no keyboard nav
- Code Quality: Long functions, magic numbers, inconsistent error handling
- Testing: Happy path only, missing edge cases

### CRITICAL: No Interview Hints

DO NOT add comments that telegraph issues to candidates.

**Bad (hints):**
- `// SECURITY ISSUE: No authentication`
- `// Should be exponential backoff!`
- `// ACCESSIBILITY: Missing labels`

**Good (natural):**
- `// TODO: Add authentication middleware`
- `// Basic retry logic`
- No comment at all

## After Generation

1. **Review for interview hints** - Remove any comments that give away issues
2. **Commit the implementation:**
   ```bash
   git add .
   git commit -m "Add <language> implementation for <role> (<level>)"
   git push -u origin interview/<language>-<role>-<level>
   ```
3. **Generate interview guide separately** (do not commit to repo)
4. **Save guide to company shared drive**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coforma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
