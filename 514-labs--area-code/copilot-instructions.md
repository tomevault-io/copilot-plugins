## area-code

> apply when working on the web applications including any -web- systems


# Web Application Rules

These rules apply to all web applications in the `apps/` directory that contain "-web-" in their folder name.

## Development Guidelines

- Use React with TypeScript
- Follow consistent component naming conventions (PascalCase for components)
- Use Tailwind CSS for styling 
- Leverage shared UI components from `@workspace/ui` package
- Use Tanstack Router for routing
- Use Tanstack Query for interfacing with APIs
- Use Tanstack Form for forms
- Use Tanstack Table for tables (Data Table in ShadCN)
- Prefer typescript interfaces over Zod schemas
- Keep casing consistent within each directory
- Prefer composition over inheritance for components
- Use proper TypeScript types and interfaces

## File Organization

- Components should be organized in logical directories
- Keep route components in the `src/routes/` directory
- Use proper barrel exports (`index.ts`) for clean imports
- Separate business logic from UI components when possible

## Performance

- Use lazy loading for routes when appropriate
- Optimize bundle size by leveraging shared packages
- Follow React best practices for re-renders

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/514-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
