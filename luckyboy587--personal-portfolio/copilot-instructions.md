## personal-portfolio

> This document provides essential information for agentic coding agents to maintain consistency and quality in this repository.

# Agent Guidelines: My Portfolio

This document provides essential information for agentic coding agents to maintain consistency and quality in this repository.

## 1. Development Commands

| Task | Command |
| :--- | :--- |
| Start Dev Server | `npm run dev` |
| Build Project | `npm run build` |
| Lint Code | `npm run lint` |
| Deploy to GitHub Pages | `npm run deploy` |
| Preview Build | `npm run preview` |

*Note: No automated test suite is currently configured. Before adding features, manually verify UI changes in the browser.*

## 2. Code Style & Architecture

### Core Technologies
- **Framework:** React 19 (Vite)
- **Styling:** Tailwind CSS v4, Framer Motion, GSAP
- **Icons:** Lucide React, FontAwesome (via CDN/classes)
- **Components:** Headless/Custom UI components

### Naming Conventions
- **Components:** PascalCase (e.g., `AboutMe.jsx`, `Navbar.jsx`)
- **Utilities/Hooks:** camelCase (e.g., `utils.js`, `useTheme.js`)
- **Folders:** kebab-case or lowercase (e.g., `src/components/ui/`)
- **CSS Classes:** Tailwind-first approach. Use kebab-case for custom classes in `index.css`.

### Directory Structure
- `src/components/`: Reusable components.
- `src/components/ui/`: Atomic UI elements (e.g., `animated-theme-toggler.jsx`).
- `src/lib/`: Core libraries and utilities (e.g., `utils.js` with `cn` helper).
- `src/utility/`: Layout/Logic helpers (e.g., `context-box.jsx`).
- `src/assets/`: Static assets like images and fonts.

### Coding Patterns
1. **Component Definition:** Use arrow functions for components.
   ```jsx
   const MyComponent = ({ prop1, prop2 }) => { ... }
   export default MyComponent;
   ```
2. **Imports:** Use the `@` alias for absolute paths from `src/`.
   ```jsx
   import { cn } from "@/lib/utils";
   import ContextBox from "@/utility/context-box.jsx";
   ```
3. **Class Merging:** Always use the `cn()` utility for conditional or merged Tailwind classes.
   ```jsx
   <div className={cn("base-class", isActive && "active-class", className)}>
   ```
4. **Animations:** Prefer **Framer Motion** for simple entry/hover animations and **GSAP** for complex timeline-based sequences.
5. **Theme Support:** Support both Light and Dark modes. Use `document.documentElement.classList.contains("light")` to check state. Styles should handle `.light` and default (dark) states.

### Linting Rules
- **ESLint:** Configured in `eslint.config.js`. 
- **Unused Vars:** Errors on unused variables unless they start with an underscore or uppercase letter (for React components).
- **React Hooks:** Standard `react-hooks/recommended` rules apply.

## 3. Implementation Checklist
- [ ] Strictly adhere to Tailwind CSS v4 syntax.
- [ ] Ensure responsive design (mobile-first approach).
- [ ] Verify animations don't negatively impact GPU performance (use `transform` and `opacity`).
- [ ] Keep components focused and modular.
- [ ] Preserve the "Glassmorphism" aesthetic used in `ContextBox`.

---
> Source: [LuckyBoy587/personal-portfolio](https://github.com/LuckyBoy587/personal-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
