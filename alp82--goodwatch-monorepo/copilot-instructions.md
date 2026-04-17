## goodwatch-monorepo

> - Use types fully and extensively


## TypeScript Coding Guidelines
- Use types fully and extensively
- Don't use `any`. If you must, use `unknown` and explain why
- Avoid enums, use maps instead - use `as const` for literals that never change
- Use `readonly` for immutable properties
- Prefer interfaces over types
- Use functional and declarative programming patterns - avoid classes
- Use higher-order functions (map, filter, reduce) to simplify logic
- Use arrow functions for simple cases (less than 3 instructions), named functions otherwise

## React / Remix
- Follow Remix and React Router best practices
- Use Tanstack Query for data management
- Use Tailwind CSS for styling
- Keep components each in their separate files
- Keep collections of custom hooks in separate hook files
- Always wrap useEffect with a well-named custom hook to improve readability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alp82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
