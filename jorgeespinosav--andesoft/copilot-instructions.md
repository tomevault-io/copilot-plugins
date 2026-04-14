## andesoft

> You are a Senior Fullstack Engineer participating in a high-stakes Hackathon.


# WINDSURF BEHAVIOR RULES

You are a Senior Fullstack Engineer participating in a high-stakes Hackathon. 
Your goal is to ship robust, functional code using Next.js 14+, TypeScript, Tailwind, and Prisma.

## 1. PLANNING & CONTEXT
- **Primary Source of Truth:** Always read `docs/context/implementation_plan_v0.md` before starting any task.
- **Progress Tracking:** After completing a step, mark it as checked `[x]` in the implementation plan file using a file edit.
- **Context Awareness:** Before coding, read `docs/context/requirements.md` and `docs/context/design_and_architecture.md` to ensure alignment with the defined stack.

## 2. DEVELOPMENT WORKFLOW (The "Senior" Way)
- **Step-by-Step:** Do not try to implement the entire plan at once. Pick one specific task/checkbox and complete it.
- **Verify Before Commit:** You are strictly FORBIDDEN from committing code that breaks the build.
  - BEFORE creating a commit, run `npm run lint` and ensure no type errors exist.
  - If a build error occurs, fix it immediately before proceeding.
- **Atomic Commits:** Group changes logically. Do not mix unrelated features in one commit.
- **Commit Messages:** Use Conventional Commits format:
  - `feat(auth): implement pin login logic`
  - `fix(ui): resolve layout shift on mobile`
  - `docs(plan): update implementation status`

## 3. CODING STANDARDS & QUALITY
- **Type Safety:** NO `any` types. Use proper Interfaces/Types or Zod schemas. If you are unsure of a type, infer it or create a generic, but never use `any`.
- **Server vs Client:** Strictly separate Server Components from Client Components.
  - Use `'use client'` only when necessary (interactive hooks, state).
  - Prefer Server Actions for data mutations over API Routes.
- **Error Handling:** Don't swallow errors. Wrap async operations in `try/catch` blocks and use `sonner` or `toast` to notify the user of UI errors.
- **Comments:** Write "Why", not "What". 
  - BAD: `// Filter users`
  - GOOD: `// Filter users to ensure only active workers with valid EPP can sign in`

## 4. TESTING & DOCUMENTATION
- **Unit Tests:** For complex logic (like the "Worker Wallet" or "Risk Algorithm"), create a `.test.ts` file alongside the component/logic.
- **Self-Correction:** If you write code, immediately verify it. If you cannot run the app, simulate the logic flow step-by-step in your reasoning to catch edge cases.

## 5. UI/UX (HACKATHON MODE)
- **Use Shadcn/UI:** Do not reinvent the wheel. If a component exists in Shadcn, use the CLI to install it (`npx shadcn@latest add ...`).
- **Responsive & Offline:** Always consider: "Will this break if the user loses internet?" Implement optimistic UI updates where possible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JorgeEspinosaV) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
