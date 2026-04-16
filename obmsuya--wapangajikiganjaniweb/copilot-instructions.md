## wapangajikiganjaniweb

> **For Active Next.js Frontend Project with Existing Structure**

# 🧠 Engineering & AI Collaboration Guide  
**For Active Next.js Frontend Project with Existing Structure**

---

## 🔐 Existing Folder Structure

> ⚠️ **Do not alter folder locations, naming conventions, or depth unless explicitly instructed.**  
> This codebase is already 60% made alreadt and follows specific structural and stylistic decisions that must be respected.

---

## 🤖 AI Assistant & Prompt Engineering Instructions

This guide ensures AI/codegen tools do not produce context-breaking, duplicate, or stylistically inconsistent work.

### Core Directives

- Never assume this is a fresh project. **It is actively developed and already designed.**
- Do not generate or assume folders/files (e.g., `services/`, `types/`, etc.) unless they **already exist**.
- Before adding logic or components, **scan existing components and pages** for patterns.
- Only create new logic if:
  - It does not exist elsewhere in the repo
  - It clearly cannot be reused or extended
- Always match design, naming, and layout **from already developed components/pages**.

---

## 🎨 Component Design & Implementation

### Style Rules

> 🧭 Always **reference existing components and pages** for style, animation, naming, and behavior.

| Rule | Instruction |
|------|-------------|
| **Styling** | Uses Tailwind CSS calling and  matching the  global css. All new styling should reflect spacing, color, typography, and layout patterns found in existing components. |
| **Animations** | Use Framer Motion **only if it’s already being used**. Otherwise, replicate transitions exactly as done in current components. |
| **Responsiveness** | Required. All new components must be responsive using existing breakpoints as reference. |
| **Accessibility** | Maintain existing accessibility conventions. Match use of roles, alt attributes, aria labels, etc. |
| **Icons** | Use only the icon libraries already present in the codebase. No new libraries unless explicitly instructed. |
| **Component Naming** | Follow the naming style of already-made components. If they are PascalCase, continue with PascalCase. Match naming depth and prefixes where applicable. |

---

## 🧠 Context Awareness & Uniformity

To avoid logic duplication and design fragmentation:

- **Before creating anything**, check:
  - The nearest similar page in the folder
  - Related components in the `components/` directory and  its  subfolders
- If styling or behavior appears similar to an existing component:
  - **Re-use it**
  - Or create a prop-driven variant if needed

### For Every New Feature:

- Use the same layout structure as already implemented pages
- Align section padding, spacing, and heading sizes with visually similar sections
- Place new components where similar ones are found, unless instructed otherwise

---

## ⚙️ Logic Implementation Instructions

| Rule | Instruction |
|------|-------------|
| **API Calls** | If a pattern exists (e.g., through custom hooks or a helper file), follow that structure. Don’t introduce raw fetch/axios unless that is already the practice. |
| **State Logic** | Use React hooks consistently. Match existing practices (e.g., `useState`, `useReducer`, custom hooks). |
| **Reusable Logic** | If a transformation or helper function exists, reuse it rather than recreating it. |
| **Error Handling** | Match the project's current way of handling async errors (e.g., try/catch, toast notifications, logging). |

---

## 📦 File Naming & Placement Guidelines

> 📌 Use **only the conventions already visible** in the project.

| File Type | Location | Naming Convention |
|-----------|----------|-------------------|
| Components | Match existing structure | Match existing casing and naming depth |
| Styles | Match `.module.css` or other style files already in use | Match naming and folder placement |
| Hooks | Only if existing custom hooks exist | Match `useXyz.ts` style |
| Utilities | Reuse logic from existing utility files | Never introduce `utils/` unless it exists already |

---

## ✅ Final Checklist Before Code is Accepted

- [ ] File placed in correct folder (no new folders unless instructed)
- [ ] Naming convention matches project
- [ ] Component style is visually consistent with existing design
- [ ] Tailwind spacing and color palette matches nearby components
- [ ] Responsive behavior has been manually checked
- [ ] No duplicate logic exists (hooks, transformations, data fetching)
- [ ] No external libraries added
- [ ] Context or layout not broken
- [ ] Only use what exists — **no speculative abstractions**

---

## 📣 Communication to AI or Developers

If unclear about:

- Which folder to use
- Which naming pattern to follow
- Whether logic already exists

> **You must first check across the codebase**, starting with components and layout files.  
> If unclear after inspection, **ask the maintainer before assuming anything**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/obmsuya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
