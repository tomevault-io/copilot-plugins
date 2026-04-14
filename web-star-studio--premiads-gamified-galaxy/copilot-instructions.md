## premiads-gamified-galaxy

> **Applies to:** All requests

# PremiAds Gamified Galaxy — Project Conventions

**Applies to:** All requests

---

## 1. Technologies & Stack
- **Frontend:** React + TypeScript, bundled with Vite.  
- **UI System:** Tailwind CSS + shadcn/ui.  
- **Animations:** Framer Motion for smooth transitions.  
- **Backend:** Supabase (Auth, Database, Edge Functions).  
- **State Management:** React Query.

## 2. Folder Structure
- `/src/components`: Reusable components, each in its own folder with `index.tsx`, `styles.css`, and tests.  
- `/src/features`: Pages or flows (e.g., `dashboard/`, `missions/`).  
- `/src/lib`: Helpers, hooks, and Supabase integrations.  
- `/src/styles`: Global themes (e.g., `galaxy-dark.css`).

## 3. Coding Conventions
- **File Naming:** PascalCase for components (`UserCard.tsx`), camelCase for hooks and utilities (`useAuth.ts`).  
- **CSS:** Utility classes from Tailwind; always include responsive variants (`md:`, `lg:`) when appropriate.  
- **Componentization:** Extract repeated snippets into lower-level components; avoid “mega-components” longer than 200 lines.

## 4. UX & Dopamine Patterns
- Play **audio feedback** (`playSound('success')`) on victory actions (mission completion, streak achieved).  
- Add Framer Motion’s `whileHover` and `whileTap` on primary action buttons.  
- Default to **Dark Theme** with root class `galaxy-dark`.

## 5. Quality & Testing
- **TDD:** Always ask the AI to generate tests first (`yarn test:unit`) before implementing features.  
- Cover edge cases and negative scenarios in every test.  
- Integrate linting (`npm lint`) and build (`npm run build`) runs via **YOLO Mode**.

## 6. Documentation
- Generate **docstrings**/JSDoc for all public functions.  
- Update the “Project Overview” Notepad whenever new features are added.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Web-Star-Studio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Web-Star-Studio)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
