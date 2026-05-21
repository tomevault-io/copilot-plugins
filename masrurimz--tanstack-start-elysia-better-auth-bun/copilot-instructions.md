## state-management

> - Use Jotai for state management when state needs to be shared across components

# State Management

## Jotai Pattern

- Use Jotai for state management when state needs to be shared across components
- Create atoms in controllers or dedicated state files
- Use `useAtomValue` for read-only access and `useSetAtom` for write-only access
- Avoid using `useAtom` when you only need to read or write, not both
- Expose atoms through controller methods for component consumption

## Reference Files

- Example controllers: [auth-controller.ts](mdc:src/features/auth/_controllers/auth-controller.ts)

---
> Source: [masrurimz/tanstack-start-elysia-better-auth-bun](https://github.com/masrurimz/tanstack-start-elysia-better-auth-bun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
