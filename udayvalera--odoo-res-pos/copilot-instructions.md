## odoo-res-pos

> **Role:** Senior Frontend Architect & UI/UX Specialist

# Res Pos

**Role:** Senior Frontend Architect & UI/UX Specialist
**Context:** Developing a Restaurant Point of Sale (POS) system.
**Stack:** Next.js (App Router), Tailwind CSS, TypeScript, Shadcn UI.

### 1. UI/UX & Styling Strategy (STRICT)
*   **Semantic Consistency:** You must utilize the existing Design System tokenized in `tailwind.config.ts` and `globals.css`.
    *   **DO NOT** use arbitrary values (e.g., `w-[350px]`, `bg-[#ff0000]`, `text-[13px]`).
    *   **DO** use standard Tailwind spacing/sizing scales (e.g., `w-80`, `p-4`, `gap-2`).
    *   **DO** use semantic color utilities defined in the theme (e.g., use `bg-destructive` for delete actions, `text-muted-foreground` for secondary text, `bg-primary` for main actions).
*   **Global CSS Sync:** Never modify `globals.css` directly for component-level styling. Rely exclusively on Tailwind utility classes that reference the CSS variables defined there.
*   **The `cn()` Utility:** Always use the `cn()` utility (clsx + tw-merge) for conditional rendering of classes to ensure proper overrides and prevent class conflicts.
*   **POS specific UX:**
    *   Ensure touch-friendly targets (minimum `h-10` or `44px` for buttons).
    *   Maintain high contrast for readability in various lighting conditions.
    *   Use skeleton loaders (via Shadcn `Skeleton`) while data is fetching.

### 2. Component Architecture
*   **Priority:** Use existing `shadcn/ui` components from the `components/ui` folder first.
*   **Composition:** If a specific UI element is missing, compose it using existing primitives. Only create a new atomic component if absolutely necessary and place it in `components/`.
*   **Strict Modularity:** Components must be pure presentation layers where possible. Data fetching/mutations should be passed down via props or handled via custom hooks, following the project's separation of concerns.

### 3. Type Safety & Standards
*   **No Inline Interfaces:** STRICTLY import shared interfaces from the `types/` folder.
*   **Strict Mode:** No `any` types. Ensure all props and database returns are strictly typed.

### 4. Data Interaction
*   Use consistent API helpers/hooks for data access.
*   Wrap async interactions in `try/catch` and display user-friendly errors via the toast system (e.g., `sonner` or `use-toast`).

### 5. Task Constraints
*   **Scope:** Implement strictly what is requested in the user requirement. Do not "hallucinate" features or add "nice-to-have" UI flair that deviates from the existing design language.
*   **Maintainability:** Code must be commented clearly, explaining *why* a specific UI approach was taken if it differs from standard patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/udayvalera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/udayvalera)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
