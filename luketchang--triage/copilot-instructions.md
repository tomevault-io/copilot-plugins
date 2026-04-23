## triage

> - Root: `pnpm build`, `pnpm lint:fix`,

## Build and Test Commands

- Root: `pnpm build`, `pnpm lint:fix`,
- Desktop: `pnpm build`, `pnpm lint:fix`, `pnpm start`

## Code Style Guidelines

### General

- Formatting: Follow Prettier config with double quotes, semicolons, es5 trailing commas, 100 char line length, 2 space indentation
- TypeScript: Strict typing required, use explicit return types
- Imports: Group and sort imports (external libs before internal, alphabetical within each group); use NodeNext import statements (while all files are TS, import their transpiled version via .js); avoid dynamic imports; when possible import from top level directories or packages rather than specifying deep subdirectories unnecessarily
- Naming: camelCase for variables/functions, PascalCase for classes/components/types, and PascalCase or camelCase for filenames
- Error Handling: Use try/catch and typed Error objects
- Closures: Use variables scoped to the closure when appropriate rather than defining module-level singletons
- Logging: When printing to test code, use console.info (do not use console.log). For actual logging, use logger.info, logger.warn, logger.error (`import { logger } from '@triage/common'`)
- Architecture: Follow modular design in monorepo structure
- Making Code Changes: Avoid "drive-by" changes of anything unrelated to what the user asked you to do.
- Scripts: Parse any command line arguments with commander
- Installing Dependencies: If you need to add a dependency, NEVER add it to the workspace root `package.json`. Always add it to the specific `package.json` of the app or package that needs it.
- Consistency: Before making changes or answering a question, _always_ look around the codebase for files that may be related or provide hints on structure. Without doing this you will often do bad things like answer questions that are not grounded in real code context, write duplicated code that was actually sitting in another file, or fail to follow structure of existing codebase.
- Throwing Errors: When throwing errors, nest the information in an object so the stack trace is preserved. For example, throw new Error("<error message>", { param: <param> }); also prefer throwing errors instead of calling process.exit(1)

### UI

- React: Use functional components with proper prop typing
- Zustand: Prefer Zustand for state shared by many components
- useEffect: Avoid except for external interactions/cleanup (e.g. subscriptions); keep user interaction logic tightly bound in event handlers, keep derived values inline or in useMemo, etc
- Data fetching: Use React Query
- CSS: Avoid using `!important` where possible
- Tailwind CSS: Group related utilities (layout, typography, colors, etc.); use `className`; prefer Tailwind utility classes over custom CSS; extract patterns to components or use @apply in a CSS file; build responsive design
- ShadCN UI: Import from `@/components/ui`; use provided variants/API; customize via variant props when possible; extend with Tailwind when needed
- Radix UI: Use for unstyled components; follow Radix composition pattern; implement ARIA/keyboard navigation; style with Tailwind; ensure proper event handling
- Icons: Use `lucide-react`

## Rules for AI

- Encourage chain-of-thought.
- Keep answers concise and direct.
- Suggest alternative solutions.
- Prioritize technical explanations.
- Use [gitingest.com](https://gitingest.com) to load all config and relevant files as one page (AI-readable).
- Use [context7.com](https://context7.com) for the latest docs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luketchang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
