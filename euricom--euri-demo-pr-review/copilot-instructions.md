## euri-demo-pr-review

> Project structure and file organization guidelines


# Project Structure

## Main Structure

- App is in root folder

```tree
├── public/            # Public assets
├── src/               # Application source code
|   ├── api/           # API types and wrappers
│   ├── components/    # React components
│   ├── context/       # contexts components
│   ├── config/        # Config data
│   ├── hook/          # Custom hooks
│   ├── lib/           # Utility functions
│   ├── pages/         # pages/features components
│   ├── App.tsx        # Application entry point
│   ├── index.css      # Main css and tailwind configuration
│   ├── main.tsx       # Main rendering file
│   └── Router.tsx     # Routes component
├── index.html         # HTML entry point
├── tsconfig.json      # TypeScript configuration
└── vite.config.ts     # Vite configuration
```

## File Naming and Organization

- Use PascalCase for components (e.g. `src/components/Button.tsx`) 
- Shadcn components are in `src/components/ui`
- All other components are in `src/components/`
- Colocate files in the folder where they're used unless they can be used across the app
- If a component can be used in many places, place it in the `src/components` folder

## New Pages

- Create new pages at: `src/pages`
- Place a new page in `src/pages/PAGE_NAME/Page.tsx` folder
- Place specific component for that page inside the page folder 

## Utility Functions

- Create utility functions in `lib/` folder for reusable logic
- Use lodash utilities for common operations (arrays, objects, strings)
- Import specific lodash functions to minimize bundle size:

```ts
import groupBy from "lodash/groupBy";
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Euricom)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Euricom)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
