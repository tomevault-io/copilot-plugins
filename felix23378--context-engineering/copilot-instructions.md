## context-engineering

> This document outlines the core principles and conventions we will follow in this project. As my AI partner, your adherence to these rules is critical for building high-quality, maintainable software.

#  The Developer's Pact
This document outlines the core principles and conventions we will follow in this project. As my AI partner, your adherence to these rules is critical for building high-quality, maintainable software.

---

## ️ Principle 1: Architecture & Project Structure

- **500-Line Rule:** No single file should exceed 500 lines. If it grows too large, propose a clear refactoring plan to break it into smaller files or modules.

- **Feature-Based Grouping:** Group files by feature or component, not by file type. For example:

/src/components/TodoList.jsx /src/components/TodoItem.jsx /src/features/auth/firebaseConfig.js

- **Modular by Default:** Reuse logic via utility functions (`/src/utils/`) and shared UI components.

- **Simple Examples Folder:** Use `/examples/` to store working reference code that Gemini can learn from. Keep them clean and focused on one concept per file.

---

## ✅ Principle 2: Code Quality & Maintainability

- **Readable Over Clever:** Prioritize readability. Avoid complex one-liners if they reduce clarity.

- **Manual Testing:** Since you’re developing on mobile (Termux), you’ll manually test features in-browser. If something should be tested, explain how to test it in the PRP response.

- **Inline Documentation:**
- Use `// WHY:` or `/* WHY: */` comments for non-obvious logic.
- Keep logic grouped in sections with clear comment headers if needed.

- **Naming Consistency:**
- Functions: `camelCase`
- Components: `PascalCase`
- Files: match component/function names

---

## ✍️ Principle 3: Styling & UI

- **Tailwind First:** Use Tailwind CSS for all styling. Group classes logically:
```html
<div class="flex flex-col gap-2 p-4 bg-white shadow-md rounded-xl">

Responsiveness Required: All layouts must be responsive to fit both mobile and desktop.

Animation and Motion: Use libraries compatible with mobile builds (e.g., Framer Motion, Tailwind plugins). Keep transitions smooth and purposeful.

Dark Mode Optional: Use system-level dark: variants in Tailwind where possible.



---

 Principle 4: Environment & Security

No Hardcoded Secrets: Never include real API keys or secrets in code. Use .env and .env.example.

Firebase/Auth Configs: Store Firebase setup files in /src/config/firebase.js or similar — keep it isolated from core logic.

Safe Defaults: Always provide fallback values (e.g., process.env.API_URL || 'http://localhost:3000').



---

 Principle 5: Behavior as AI Assistant

No Hallucination: Do not invent library names, file paths, or tool chains. If unsure, ask the user for clarity.

Explain Before Code: For medium or large tasks, provide a bullet-plan or pseudocode first. Don’t write code until the plan is approved.

Context-Aware Suggestions: Suggestions should match:

Mobile-first UI development (Tailwind + React)

Termux limitations (no Docker, no VS Code extensions)

Git + CLI workflows


Be Explicit About Output: If the output includes multiple files or updates, list them clearly first, then show the content

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Felix23378)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Felix23378)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
