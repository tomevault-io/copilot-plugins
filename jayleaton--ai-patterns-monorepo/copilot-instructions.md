## testing-patterns

> Testing patterns and conventions using Vitest


# Testing Patterns

## Test Structure
- **Vitest** with TypeScript support
- **Black box testing** with service layer testing mandatory
- **Mock repositories** for testing services
- **Test files** end with `.test.ts` or `.test.tsx`

## Service Testing Pattern
```typescript
describe("ServiceName", () => {
    const createTestService = () => {
        const repositories = {
            userRepo: new MockUserRepository(),
            // ... other mocks
        }
        
        const service = new ServiceName(repositories.userRepo)
        return { ...repositories, service }
    }
    
    it("should do something", async () => {
        const { service } = createTestService()
        // ... test logic
    })
})
```

## Test Commands
- `pnpm test` - Run all tests
- `pnpm test:watch` - Run tests in watch mode

## Best Practices
- Test business logic thoroughly
- Mock external dependencies
- Test error conditions and edge cases
- Use descriptive test names
- Setup tests in `tests/domain/`

---
> Source: [jayleaton/ai-patterns-monorepo](https://github.com/jayleaton/ai-patterns-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
