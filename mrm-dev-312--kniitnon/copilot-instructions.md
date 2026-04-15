## kniitnon

> This document provides the mandatory operational instructions for the Gemini CLI. Adherence is required to ensure quality and efficiency.

# Gemini CLI Project Guidance
This document provides the mandatory operational instructions for the Gemini CLI. Adherence is required to ensure quality and efficiency.

## 1. Core Principles
- **Methodical Execution:** Follow the workflow step-by-step. No shortcuts.
- **Simplicity & Focus:** Implement the most direct and efficient solution that meets the requirements.
- **Foundation First:** Always verify config, dependencies, and the database before debugging application code.
- **Document as You Go:** Keep TASK.md and MEMORY.md continuously updated.
- **Automate Validation:** Use tests and linters to verify all changes before marking a task as complete.

## 2. Development Workflow
1.  **Analyze:** Thoroughly review project requirements (e.g., PRD.md).
2.  **Plan:** Create a detailed, sequential task list in TASK.md.
3.  **Verify:** Await plan approval before writing any code.
4.  **Execute:** Complete tasks one by one, updating TASK.md as you go.
5.  **Log:** Add a brief entry to MEMORY.md after each session or significant change.

## 3. Technical Best Practices
### Foundation-First Debugging
Always check in this order before debugging code:

1.  **Configuration:** .env.local variables, 
ext.config.mjs.
2.  **Dependencies:** Run 
pm install; verify package.json and imports.
3.  **Database:** Check Vercel Postgres status and ensure prisma/schema.prisma aligns with the database state.

### Code & Database Rules
- **Style:** Format and lint all code (
pm run lint) before completing any task.
- **Testing:** Run all relevant tests after every change.
- **Migrations:** Use 
px prisma migrate dev --name <descriptive-name> for all schema changes. Test migrations on a clean database and have a documented rollback plan.
- **Security:** Regularly check for vulnerabilities (
pm audit) and outdated dependencies.

## 4. Quality Gates Checklist
A task is not complete until every one of these checks has passed:

- [ ] All relevant tests pass.
- [ ] No TypeScript or ESLint errors.
- [ ] Code is formatted (prettier --write .) and linted (
pm run lint).
- [ ] Database schema matches model definitions.
- [ ] Migrations can upgrade and downgrade cleanly on a test database.
- [ ] All new functions, components, and modules have clear JSDoc comments.
- [ ] Project documentation (README.md, etc.) is updated with any changes.
- [ ] Robust error handling is implemented for all new code paths.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrm-dev-312)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mrm-dev-312)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
