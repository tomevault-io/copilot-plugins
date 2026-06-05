## solid-js

> This project is a Solid.js application built with TypeScript, focusing on type safety, reactive state management, and functional components. The goal is to create a maintainable and scalable codebase with strict TypeScript checks and best practices.

# Solid.js (TypeScript)

# Solid.js with TypeScript .cursorrules

## Project Background
This project is a Solid.js application built with TypeScript, focusing on type safety, reactive state management, and functional components. The goal is to create a maintainable and scalable codebase with strict TypeScript checks and best practices.

## Coding Standards
- Prefer functional components over class components.
- Use `createSignal<T>()` for typed reactive state.
- Implement proper type definitions for components.
- Utilize TypeScript's strict mode.
- Use type inference where possible.
- Implement interfaces for complex prop types.
- Utilize utility types provided by Solid.js.

## Preferred Libraries
- **Solid.js**: For reactive state management and component architecture.
- **TypeScript**: For type safety and developer productivity.
- **Solid Router**: For routing with proper typing.
- **Type-safe Context**: Use `createContext` with proper typing.

## File Structure
```
src/
  routes/
  components/
  pages/
  utils/
  types/
  App.tsx
  index.tsx
public/
  index.html
tsconfig.json
```

## Performance Optimization
- Use `createMemo` for derived state to avoid unnecessary re-renders.
- Implement lazy loading for components where applicable.
- Optimize `createEffect` dependencies to prevent unnecessary side effects.

## Testing Requirements
- Write unit tests using **Jest** and **Solid Testing Library**.
- Ensure test coverage is at least 80%.
- Use snapshot testing for UI components.

## Documentation
- Write comments for functions and components in **JSDoc** format.
- Components must include **PropTypes** validation.
- Each main directory must contain a `README.md` file.

## Error Handling
- Use `try/catch` blocks to handle asynchronous operations.
- Implement global error boundaries for runtime errors.

## Additional Instructions
1. Use `.tsx` extension for files with JSX.
2. Implement strict TypeScript checks.
3. Utilize Solid Router with proper typing.
4. Use type-safe context with `createContext`.
5. Implement proper typing for event handlers.
6. Follow TypeScript best practices and naming conventions.
7. Use type assertions sparingly and only when necessary.

---
> Source: [chikingsley/rivena-ai-app](https://github.com/chikingsley/rivena-ai-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
