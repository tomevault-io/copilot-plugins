## myceliainteractive

> 1. **MANDATORY READING**: Before generating ANY plan, writing ANY code, or executing ANY command, you MUST read the following file:

# COPILOT CORE DIRECTIVES — MYCELIA INTERACTIVE

1. **MANDATORY READING**: Before generating ANY plan, writing ANY code, or executing ANY command, you MUST read the following file:
   - `AGENTS.md` — project source root. Rules, architecture, coding standards, execution protocol.
2. **Blind Obedience to AGENTS.md**: The rules in `AGENTS.md` supersede any default Copilot behaviors.
3. **Acknowledge**: Begin every response with `"AGENTS.md acknowledged"` to prove you have read the constraints for the current session.
4. **PROTECTED FILES**: `AGENTS.md` and `README.md` always reside in the project source root and **cannot be moved, renamed, replaced, or deleted** without the user's explicit command or permission.

Moving forward, whenever asked to build a feature, create a component, or wire up any API connection, follow this exact, unbreakable execution protocol:

**STEP 1: The Micro-Plan**
- Before writing any code, output a step-by-step plan.
- Break the task into microscopic, isolated steps.
- Wait for user approval before proceeding.

**STEP 2: Execute ONE Tiny Step**
- Execute ONLY Step 1. Do not touch Step 2.
- Write the minimal code required. NEVER overwrite existing functional code to make it "better".
- **ALWAYS ask before removing code or deleting files.**

**STEP 3: Mandatory Testing**
- Run `npx tsc --noEmit`, `npm run prettier --write <file>`, and `npx eslint --fix <file>` on modified files only.
- Fix all errors before advancing.
- **Always run `npm run build` before any deploy.** Never run `wrangler deploy` directly — use `npm run deploy`.

**STEP 4: Commit and Push**
```bash
git add <changed files>
git commit -m "feat/fix: completed [Step Name] - tiny increment"
git push origin main
```

**STEP 5: Await Human Confirmation**
- Ask: *"Step complete and pushed to main. Ready for the next step?"*
- Only proceed after the user says "yes".

Moving forward, whenever asked to build a feature, create a component, or wire up any API connection, follow this exact, unbreakable execution protocol:

**STEP 1: The Micro-Plan**
- Before writing any code, output a step-by-step plan.
- Break the task into microscopic, isolated steps.
- Wait for user approval before proceeding.

**STEP 2: Execute ONE Tiny Step**
- Execute ONLY Step 1. Do not touch Step 2.
- Write the minimal code required. NEVER overwrite existing functional code to make it "better".
- **ALWAYS ask before removing code or deleting files.**

**STEP 3: Mandatory Testing**
- Run `npx tsc --noEmit`, `npm run prettier --write <file>`, and `npx eslint --fix <file>` on modified files only.
- Fix all errors before advancing.
- **Always run `npm run build` before any deploy.** Never run `wrangler deploy` directly — use `npm run deploy`.

**STEP 4: Commit and Push**
```bash
git add <changed files>
git commit -m "feat/fix: completed [Step Name] - tiny increment"
git push origin main
```

**STEP 5: Await Human Confirmation**
- Ask: *"Step complete and pushed to main. Ready for the next step?"*
- Only proceed after the user says "yes".

---
> Source: [TheSeeker713/myceliainteractive](https://github.com/TheSeeker713/myceliainteractive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
