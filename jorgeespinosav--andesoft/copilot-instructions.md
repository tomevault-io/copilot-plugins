## andesoft

> You are strictly required to follow this "Code-Verify-Commit" loop for every task. Do not skip steps.


## GITFLOW & QUALITY GATE PROTOCOL

You are strictly required to follow this "Code-Verify-Commit" loop for every task. Do not skip steps.

### 1. DEVELOPMENT (Feat/Fix)
- Focus on one logical task at a time (Atomic Changes).
- Do not mix purely stylistic changes (formatting) with logic changes in the same group.

### 2. VERIFICATION (The "Senior" Check)
**BEFORE** suggesting a commit or considering a task "done", you MUST execute the following checks in the terminal:

1.  **Linter/Type Check:** Run `npm run lint` (or `tsc --noEmit`) to catch TypeScript errors.
2.  **Build Check:** Run `npm run build` if the change affects core config, routing, or environment variables.
3.  **Unit Tests:** Run `npm run test:unit` for utility functions or isolated components.
4.  **INTEGRATION TESTS (CRITICAL):**
    - If you modified Server Actions, Database Schema, or API Routes, you **MUST** run the integration suite (e.g., `npm run test:integration` or `vitest`).
    - **Verification Goal:** Ensure the code actually talks to the DB/API correctly. Mocking is not enough for core flows.
5.  **Self-Correction:** If any check fails, FIX IT immediately. Do not ask the user if they want to commit broken code.

### 3. COMMITTING (Descriptive & Grouped)
Once ALL verification passes, construct a commit message following **Conventional Commits**:
- Format: `<type>(<scope>): <description>`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.

**Rules for Messages:**
- ❌ BAD: "add auth tests", "fix login".
- ✅ GOOD: "test(auth): add integration test for worker pin login flow", "fix(db): resolve prisma schema relation for job areas".

### 4. EXECUTION
- Automatically stage related files.
- Present the commit message to the user for confirmation before executing `git commit`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JorgeEspinosaV)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JorgeEspinosaV)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
