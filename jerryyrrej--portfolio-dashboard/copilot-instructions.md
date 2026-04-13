## portfolio-dashboard

> This project adheres to a strict "Restrained Minimalism" aesthetic, heavily inspired by modern Apple design (macOS & iOS). Every AI assistant MUST follow these core principles when modifying the UI:

# Gemini Project Context: Portfolio Dashboard

## 🎨 Design Philosophy & UI Principles
This project adheres to a strict "Restrained Minimalism" aesthetic, heavily inspired by modern Apple design (macOS & iOS). Every AI assistant MUST follow these core principles when modifying the UI:

### 1. Visual Hierarchy & Spacing
- **Generous White Space**: Use large gaps (`gap-10`, `gap-12`, `py-10`) to create a sense of premium quality and focus.
- **Physical Centering**: Key content (like the Settings box) should be the absolute physical center of the layout. Navigational elements should sit to the side as "appendages" to the main content, balanced by invisible spacers.
- **Typography**: Prefer `Inter` or system fonts. Use `tracking-tight` for bold headers and `tabular-nums` for all financial data to ensure alignment.

### 2. Interaction Standards
- **Expandable Workspace (Accordion)**: For secondary editing tasks (e.g., changing email, updating password), **DO NOT** use modals if the context can be maintained. Instead, use a "smooth drawer" effect that pushes down subsequent content. 
  - *Implementation Hint*: Use CSS Grid (`grid-rows-[0fr] -> grid-rows-[1fr]`) for fluid height animations.
- **Translucency**: Use `backdrop-blur-xl` with semi-transparent backgrounds (`bg-white/70`) for sticky headers and high-level overlays to create depth.
- **Color Palette**: 
  - Primarily **Black** and **Gray** scale for UI elements.
  - **Red (`rose-500`)** is reserved EXCLUSIVELY for destructive actions (Logout, Delete, Loss).
  - **Emerald Green** for success states and positive returns.

### 3. Consistency & Restraint
- **Icon Usage**: Avoid unnecessary icons in lists or menus. Text should stand on its own.
- **Circular Primaries**: Use circular containers (`rounded-full`) for profile photos and company logos. Apply `object-cover` to fill the circle completely.
- **Remote Asset Logos**: All remote stock / asset / company logos MUST use the shared `CachedAssetLogo` component. Do not render raw remote logo URLs directly with `<img>` or `next/image` inside feature components. This is a performance and UX rule, not an optional styling preference.

### 4. Micro-Interaction Standards
- **Active Icon Effect**: When a setting item is in "edit mode" or "active", its icon container MUST:
  - Scale slightly (`scale-110`).
  - Add a soft halo (`ring-4 ring-black/5`).
  - Transition icon color from `gray-400` to `black`.
  - Use `duration-300 ease-in-out` for all transitions to ensure a "premium" feel.
- **Destructive Actions**: Use `bg-rose-50/30` with `text-rose-500` for sign-out or delete buttons, adding a subtle icon translation (e.g., `group-hover:-translate-x-0.5`) on hover.

### 5. Component Design Standards (Physicality & Depth)
To maintain the Apple-inspired "Restrained Minimalism," all interactive components must follow these physical interaction laws:

- **Segmented Controls (Tab Switchers)**:
    - **The Track**: Always use a recessed container: `bg-element/50 p-1 rounded-2xl gap-1`.
    - **The Floating Pill**: The selected item must look "elevated" using `bg-white dark:bg-zinc-100 text-black shadow-sm rounded-xl`. 
    - **Unselected items**: Use `text-secondary hover:text-primary` with no background.
- **Physical Switches (Toggles)**:
    - **OFF State**: Use a neutral, recessed track: `bg-element shadow-inner border border-border/50`.
    - **ON State**: Use the primary high-contrast track: `bg-primary`.
    - **The Knob**: Always high-contrast (`bg-white dark:bg-zinc-100`) with a distinct shadow (`shadow-[0_1px_3px_rgba(0,0,0,0.4)]`) to suggest it sits *on top* of the track.
- **Secondary & Cancel Actions**:
    - **Clarity over Contrast**: For "Cancel" or secondary buttons, use `text-secondary`.
    - **Active State**: When in "Edit Mode" or "Active," the button can take a subtle `bg-element-hover` to show context, but the text must remain `text-secondary` to avoid visual competition with the "Save/Confirm" action.
    - **Dark Mode**: Never use `text-gray-700` on dark backgrounds; always use the adaptive `text-secondary`.

---

## 🏗️ Architecture (System Context)

### Directory Structure
```
src/
├── app/                      # Next.js App Router
│   ├── api/                 # API Routes
│   ├── components/          # UI Components
│   │   ├── settings/        # Settings specific components
│   │   └── AddTransactionModal.tsx
│   ├── settings/            # Dedicated settings layout (Account & Security unified)
│   ├── transactions/        # Transaction history list
│   └── page.tsx             # Home (Dashboard)
├── lib/                     # Shared utilities (Supabase, Finnhub)
└── hooks/                   # Custom React hooks (useStock, etc.)
```

### Important Implementation Details
- **Unified Identity**: Account settings and Security/Privacy are merged into a single "Account & Security" domain.
- **Data Flow**: Server Components fetch via Prisma -> Props passed to Client Components.
- **API Strategy**: External calls are proxied through server-side routes to protect keys.

---

## 📜 Development Commandments
- **NEVER** add a global state manager (Redux/Zustand) unless explicitly ordered.
- **NEVER** use TailwindCSS v4 features if the project is on v3 (and vice versa).
- **NEVER** perform any Git operations (add, commit, push, etc.) - all Git operations are handled by the user.
- **ALWAYS** check for `tabular-nums` when rendering currency or percentages.
- **ALWAYS** maintain the "Center-aligned White Box" layout for major management interfaces.
- **ALWAYS** route remote asset/company logos through `CachedAssetLogo` so logo rendering is cached and behavior stays consistent across pages, modals, and cards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JerryyrreJ)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JerryyrreJ)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
