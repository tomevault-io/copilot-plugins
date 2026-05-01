## chanitec-frontend-myv

> description:Zidor Protocol

---
description:
globs:
alwaysApply: true
---
description:Zidor Protocol
globs:
  - "**/*"
alwaysApply: true
---

1. Code Quality & Optimization
- Always generate clean, modular, and performant code.
- Prioritize simplicity, but never at the cost of functionality.
- Avoid adding new dependencies, rewriting existing logic, or modifying unrelated code unless explicitly instructed.
- Never refactor or "optimize" code without a direct prompt.

2. Precision & Accuracy
- Follow instructions **exactly** as written.
- Do **not assume, generalize, or infer missing details**.
- If a requirement is unclear or ambiguous, halt and prompt the user for clarification.
- Constants, formulas, and logic must exactly match what's provided, especially in domain-specific tasks.

3. Performance & Scalability
- Optimize only when requested.
- Avoid excessive loops, complex data structures, or premature optimization.
- Keep code scalable through modular structure, but do not over-engineer unless specified.

4. Security & Error Handling
- Implement basic input validation and error handling if it is **part of the instructions**.
- Do not auto-inject security logic unless specifically requested.
- Never expose sensitive data. No hardcoded credentials or keys.

5. Language & Style Consistency
- Adhere strictly to the syntax, style, and conventions of the language in use.
- Use consistent naming, indentation, and formatting across the file.
- Include docstrings or comments **only when explicitly requested** or where clarity is critical.

6. User-Focused Approach
- Ensure a clean, intuitive UX when UI/UX tasks are involved.
- For APIs, follow RESTful standards with clear status codes and messages.
- When building interfaces, avoid assumptions—follow only provided user flow specs.

7. AI Code Refinement Protocol
- Provide the **optimal** solution first—clearly justified.
- If multiple approaches are valid, suggest alternatives with pros and cons.
- **Never auto-implement improvements**—only propose them for review.
- Do not modify existing code unless explicitly instructed to do so.

8. Project-Specific Logic
- Honor all project-specific constants, tax rules, and business logic as described.
- Before editing existing code, check for compatibility and consistency with the existing structure.
- Avoid breaking existing flows or logic unless it's a confirmed bug and the fix is explicitly approved.

9. Powershell Specific
- When writing PowerShell commands, **always separate** commands clearly.
- Avoid aliases; prefer full command names for clarity.

10. GitHub Deployment Readiness
- Ensure code structure and dependencies are compatible with Vercel deployment when preparing for GitHub pushes.
- Add or adjust configuration files (e.g., `vercel.json`) **only if instructed**.

11. Non-Destructive Behavior
- Do not modify, reformat, or clean code segments that are unrelated to the current task.
- Never rewrite entire files unless explicitly told to.
- All changes must be scoped to the user's request only.

12. Change Approval Mechanism
- If a change might improve the code but is not directly requested, suggest it in a comment block or a prompt.
- Do not make structural or logic modifications based on "best practice" unless explicitly allowed.

13.Only act on what I tell you. Suggest improvements only in comments or prompts. Never make assumptions. Never change or touch unrelated code. Always explain your choices when multiple paths are possible.

14. File Creation Discipline
- Do **not create new files** unless it is confirmed that no existing file serves the same purpose.
- Before creating a file, **check all existing files** (by name, structure, and content) to ensure it doesn't already exist under a different name or structure.
- If there's a possible match or overlap, prompt the user for confirmation before proceeding.
- Default behavior: reuse or extend existing files when appropriate and approved.

15. scan this file to understand the project structure before you do anything
C:\Users\Zidor\Desktop\calcul de prix final\Chanitec\chanitec-frontend\.cursor\rules\frontend-pages-analysis.mdc

16. scan this file to understand backend connections
C:\Users\Zidor\Desktop\calcul de prix final\Chanitec\chanitec-frontend\.cursor\rules\connection.mdc

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zidor2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
