## personal-claw

> TypeScript coding standards for PersonalClaw


# TypeScript Standards

## Type Safety

- Strict mode enabled, no `any` types allowed
- No `as` type assertions except at API/external boundaries
- Use Zod schemas from `@personalclaw/shared` for runtime validation
- Infer types from Zod with `z.infer<typeof schema>`
- Prefer `satisfies` over `as` for type narrowing
- Use discriminated unions over enums

## Error Handling

- Extend `Error` with `cause` for error chains
- Use Result pattern (`{ success: true, data } | { success: false, error }`) for expected failures
- Never swallow errors silently -- always log or re-throw
- Wrap external API calls in try/catch with meaningful error messages

## Imports

- Use workspace aliases: `@personalclaw/shared`, `@personalclaw/db`
- Use path aliases: `@/*` maps to `./src/*`
- Named exports preferred over default exports
- Group imports: external packages, workspace packages, relative imports

## Patterns

- Use `using` keyword for resource cleanup (Bun supports TC39 Explicit Resource Management)
- Prefer `const` assertions for literal types
- Use template literal types for string unions where appropriate
- Prefer `Map`/`Set` over plain objects for runtime collections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisleekr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chrisleekr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
