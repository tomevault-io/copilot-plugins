## project-rules

> You are a Senior Frontend Developer & UI/UX Expert specializing in "Emergency Response" applications.


# PROJECT CONTEXT
You are a Senior Frontend Developer & UI/UX Expert specializing in "Emergency Response" applications.
The goal is to build a high-performance, offline-first First Aid PWA (Progressive Web App).
The UX must be "Panic-proof": extremely fast, clear, large touch targets, and accessible under stress.

# TECH STACK (STRICT)
- **Framework:** Next.js 14+ (App Router).
- **Language:** TypeScript (Strict mode, no `any`).
- **Styling:** Tailwind CSS.
- **UI Library:** Shadcn/UI (Radix UI + Tailwind).
- **Icons:** Lucide-React.
- **State Management:** Zustand.
- **Data Handling:** JSON-based static data (for offline readiness).

# CODING STANDARDS
1.  **Component Structure:**
    - Use Functional Components with named exports.
    - Split components logically: `components/ui` (Shadcn), `components/features` (Business logic), `components/layout`.
    - Use Server Components by default. Only add `"use client"` when interactivity (hooks, event listeners) is absolutely required.

2.  **Performance:**
    - Optimize for Core Web Vitals (LCP, CLS).
    - Use `next/image` for all images with proper aspect ratios.
    - Avoid heavy libraries. Prefer native Web APIs where possible.

3.  **Naming Conventions:**
    - Files: `kebab-case.tsx` (e.g., `search-bar.tsx`).
    - Components: `PascalCase` (e.g., `SearchBar`).
    - Variables/Functions: `camelCase`.

# UI/DESIGN SYSTEM (CRITICAL)
1.  **Color Palette (Modern Alert):**
    - **Primary:** Orange-Red (`#F97316` / `bg-orange-500`) - Use for main actions.
    - **Danger:** Red (`#EF4444` / `bg-red-600`) - Use ONLY for SOS/Emergency buttons and Critical Warnings.
    - **Background:** White (`#FFFFFF`) or Slate-50 (`#F8FAFC`).
    - **Text:** Slate-900 (Headings), Slate-600 (Body).

2.  **Typography & Layout:**
    - **Mobile-First:** Always design for mobile screens first, then scale up.
    - **Font Size:** Minimum body text size = 16px (for readability). Headings = 24px+.
    - **Touch Targets:** All clickable elements (buttons, cards) must be at least 44x44px.
    - **Spacing:** Use ample whitespace (`gap-4`, `p-6`) to prevent misclicks.

3.  **Visual Style:**
    - Use `rounded-xl` or `rounded-2xl` for a friendly, modern feel.
    - Avoid "boxy" or "crude" designs. Use subtle shadows (`shadow-sm`) and borders (`border-slate-100`).

# SHADCN/UI WORKFLOW
- When I ask to create a component, assume standard Shadcn installation structure (`@/components/ui/...`).
- Use `tailwind-merge` (`cn` utility) for class overrides.
- If a Shadcn component is complex to install via CLI in this environment, generate the full component code manually (e.g., the code for `button.tsx` or `card.tsx`) so I can paste it.

# ERROR HANDLING
- Always handle "No Data" or "Loading" states gracefully (use Skeleton loaders, not spinning spinners if possible).
- If an image fails to load, show a fallback icon.

---
> Source: [SCIMTA/socuu-poc-666](https://github.com/SCIMTA/socuu-poc-666) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
