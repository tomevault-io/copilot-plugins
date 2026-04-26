## geo-guesser

> Testing patterns and conventions


# Testing Guidelines

## Test Structure
```typescript
describe('MatchService', () => {
  describe('createMatch', () => {
    it('should create match with valid players', async () => {
      // Arrange
      const player1 = await createTestUser();
      const player2 = await createTestUser();
      
      // Act
      const match = await MatchService.createMatch(player1.id, player2.id);
      
      // Assert
      expect(match.status).toBe('active');
      expect(match.rounds).toHaveLength(5);
    });
    
    it('should throw if player not found', async () => {
      await expect(
        MatchService.createMatch('invalid-id', 'invalid-id')
      ).rejects.toThrow(PlayerNotFoundError);
    });
  });
});
```

## Naming
- Test files: {filename}.test.ts or {filename}.spec.ts
- Describe blocks: Component/Service name
- It blocks: "should [expected behavior] when [condition]"

## Coverage Targets
- Minimum 80% coverage for services
- 100% coverage for utility functions
- Focus on critical paths (scoring, matchmaking, auth)

## Mocking
- Mock external APIs (Google Maps, Supabase)
- Use factories for test data
- Reset mocks between tests
- Prefer integration tests over unit tests for APIs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/keokukzh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
