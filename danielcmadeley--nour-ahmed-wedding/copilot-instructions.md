## nour-ahmed-wedding

> Only use technologies and libraries that are listed in package.json dependencies and devDependencies


# Technology Stack Restrictions

Only use technologies, libraries, and tools that are explicitly listed in this project's package.json. Do not suggest or use any technologies that are not already in the dependencies or devDependencies.

## Allowed Technologies

### Core Framework & Runtime
- **React** (^19.2.3) - Use React 19 features only
- **React DOM** (^19.2.3)
- **TypeScript** (^5.9.3) - As peer dependency
- **Vite** (^7.3.1) - Build tool and dev server
- **Bun** - Runtime (per existing project rules)

### Routing & Framework
- **@tanstack/react-router** (^1.149.3) - Routing solution
- **@tanstack/react-start** (^1.149.4) - Framework

### UI Components & Styling
- **@radix-ui/react-*** - All Radix UI components (accordion, alert-dialog, aspect-ratio, avatar, checkbox, collapsible, context-menu, dialog, dropdown-menu, hover-card, label, menubar, navigation-menu, popover, progress, radio-group, scroll-area, select, separator, slider, slot, switch, tabs, toggle, toggle-group, tooltip)
- **tailwindcss** (^4.1.18) - CSS framework
- **@tailwindcss/vite** (^4.1.18) - Vite plugin
- **tw-animate-css** (^1.4.0) - Tailwind animations
- **tailwind-merge** (^3.4.0) - Tailwind class merging
- **lucide-react** (^0.562.0) - Icons
- **next-themes** (^0.4.6) - Theme management

### Forms & Validation
- **react-hook-form** (^7.71.1) - Form management
- **@hookform/resolvers** (^5.2.2) - Form validation resolvers
- **zod** (^4.3.5) - Schema validation

### File Upload
- **uploadthing** (^7.7.4) - File upload service and server SDK
- **@uploadthing/react** (^7.3.3) - React components for UploadThing file uploads

### Utilities & Helpers
- **clsx** (^2.1.1) - Conditional class names
- **class-variance-authority** (^0.7.1) - Component variants
- **date-fns** (^4.1.0) - Date utilities
- **cmdk** (^1.1.1) - Command menu component
- **input-otp** (^1.4.2) - OTP input component
- **react-day-picker** (^9.13.0) - Date picker
- **embla-carousel-react** (^8.6.0) - Carousel component
- **react-resizable-panels** (^4.4.1) - Resizable panels
- **vaul** (^1.1.2) - Drawer component
- **sonner** (^2.0.7) - Toast notifications
- **recharts** (2.15.4) - Chart library

### Build Tools & Plugins
- **@vitejs/plugin-react** (^5.1.2) - Vite React plugin
- **vite-tsconfig-paths** (^6.0.4) - TypeScript path resolution
- **shadcn** (^3.6.3) - CLI tool for component management (devDependency)

### Type Definitions
- **@types/bun** - Bun type definitions
- **@types/node** (^25.0.8) - Node.js type definitions
- **@types/react** (^19.2.8) - React type definitions
- **@types/react-dom** (^19.2.3) - React DOM type definitions

## Restrictions

- **DO NOT** suggest or use any library, framework, or tool that is not listed above
- **DO NOT** add new dependencies without explicit user approval
- **DO NOT** use alternatives to the listed technologies (e.g., don't suggest Zustand if not in package.json, don't suggest Next.js, don't suggest other UI libraries)
- When suggesting solutions, only use the technologies and patterns that are already in the project
- If a feature requires a library not in package.json, explicitly ask the user if they want to add it first

## Examples

✅ **Correct**: Use `react-hook-form` with `zod` resolver for form validation
❌ **Incorrect**: Suggest using Formik or React Final Form

✅ **Correct**: Use `@tanstack/react-router` for routing
❌ **Incorrect**: Suggest React Router or Next.js routing

✅ **Correct**: Use Radix UI components for UI primitives
❌ **Incorrect**: Suggest Material-UI, Chakra UI, or other component libraries

✅ **Correct**: Use `tailwindcss` for styling
❌ **Incorrect**: Suggest styled-components, emotion, or CSS-in-JS libraries not in package.json

✅ **Correct**: Use `uploadthing` and `@uploadthing/react` for file uploads
❌ **Incorrect**: Suggest other file upload libraries like react-dropzone, multer, or cloudinary SDK

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielcmadeley)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielcmadeley)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
