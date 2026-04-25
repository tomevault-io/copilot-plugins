## factguard-website

> Coding best practices for software engineering quality, testing, security, and maintainability


# Coding Best Practices

## Naming Conventions

- Use descriptive, intention-revealing names for variables, functions, and classes.
- Avoid abbreviations unless universally understood (`id`, `url`, `api`).
- Boolean variables: prefix with `is`, `has`, `should`, `can` (e.g., `isValid`, `hasAccess`).
- Constants: use `UPPER_SNAKE_CASE` (e.g., `SALT_ROUNDS`, `JWT_EXPIRATION`).
- Types/Interfaces: use `PascalCase` (e.g., `AuthUser`, `UserProfile`).
- Functions: use `camelCase` with a verb prefix (e.g., `getProfile`, `validateInput`).

## Testing

### Test Naming

- Use descriptive `it`/`test` descriptions that read as sentences explaining the expected behavior.
- Format: `it("<expected behavior> when <condition>")` or `it("<action verb> <expected outcome>")`.

```typescript
// BAD
it("test1", ...)
it("works", ...)
it("should work correctly", ...)

// GOOD
it("returns 401 when no auth header is provided", ...)
it("hashes the password before storing", ...)
it("rejects password shorter than 8 characters", ...)
```

### Test Data

- Use generic, clearly-fake test data — never real emails, names, or credentials.
- Prefer `test@example.com`, `user@example.com` for emails (`example.com` is RFC-reserved).
- Use obvious placeholder IDs: `"uuid-1"`, `"550e8400-e29b-41d4-a716-446655440000"`.
- Use simple, obviously-fake passwords: `"password123"`, `"newpass1234"`.

### Test Structure

- Follow the **Arrange-Act-Assert** (AAA) pattern in every test.
- Each test should verify exactly one behavior.
- Use `beforeEach` to reset shared state (e.g., `jest.clearAllMocks()`).
- Extract reusable test helpers (e.g., request factory functions) to reduce duplication.
- Test both happy paths and error/edge cases — including boundary values.

```typescript
// GOOD: clear AAA structure
it("returns 200 with full profile data", async () => {
  // Arrange
  mockGetProfile.mockResolvedValue(profileData);

  // Act
  const response = await GET(makeGetRequest());
  const data = await response.json();

  // Assert
  expect(response.status).toBe(200);
  expect(data.user.email).toBe("test@example.com");
});
```

### Mock Management

- Place `jest.mock()` calls before imports of the mocked module.
- Type mock functions explicitly: `const mockFn = fn as jest.Mock`.
- Always call `jest.clearAllMocks()` in `beforeEach`.
- Restore spies (e.g., `consoleSpy.mockRestore()`) to avoid test pollution.

## Error Handling

- Use custom error classes with structured metadata (e.g., `statusCode`).
- Never expose internal error details (stack traces, DB errors) to API consumers.
- Return generic messages like `"Internal server error"` for unexpected failures.
- Log unexpected errors server-side with `console.error` before returning 500.
- Use `instanceof` checks to distinguish known errors from unexpected ones.

```typescript
// BAD — leaks internal details
catch (error) {
  return NextResponse.json({ error: error.message }, { status: 500 });
}

// GOOD — structured and safe
catch (error) {
  if (error instanceof AuthError) {
    return NextResponse.json({ error: error.message }, { status: error.statusCode });
  }
  console.error("Unexpected error:", error);
  return NextResponse.json({ error: "Internal server error" }, { status: 500 });
}
```

## Security

- Never expose sensitive fields (e.g., `passwordHash`) in API responses.
- Hash passwords with bcrypt (cost factor >= 10) — never store plaintext.
- Use constant-time comparison for auth (bcrypt.compare handles this).
- Return identical error messages for "user not found" and "wrong password" to prevent user enumeration.
- Validate and sanitize all user input at the API boundary using schemas (Zod).
- Never hardcode secrets — use environment variables with safe fallbacks only in dev.

## Architecture & Separation of Concerns

- **Route handlers** (`app/api/`): HTTP concerns only — parse request, call service, format response.
- **Services** (`lib/services/`): Business logic, DB access, domain rules.
- **Validators** (`lib/validators/`): Input validation schemas and type inference.
- **Utilities** (`lib/`): Shared helpers (e.g., auth token extraction).
- Each layer should only depend on the layer below it, never upward.

## Code Quality

- Prefer `const` over `let`; never use `var`.
- Use TypeScript strict mode — avoid `any`; use `unknown` and narrow with type guards.
- Extract magic numbers into named constants.
- Keep functions small and focused — one function, one responsibility.
- Use early returns to reduce nesting.
- Prefer explicit return types on exported functions.
- Use path aliases (`@/`) instead of relative paths for imports.

## API Design

- Use correct HTTP methods: `GET` (read), `POST` (create), `PATCH` (partial update), `PUT` (full replace/idempotent action), `DELETE`.
- Return appropriate status codes: `200` (success), `201` (created), `400` (validation), `401` (auth), `404` (not found), `409` (conflict), `500` (server error).
- Use consistent response shapes: `{ user: ... }` for data, `{ error: "...", details?: ... }` for errors.
- Validate request bodies and return `400` with details on failure.
- Handle malformed JSON bodies gracefully (return `400`, not `500`).

## DRY & Maintainability

- Extract repeated patterns into shared utilities or helper functions.
- Co-locate related code (tests mirror source structure: `lib/services/foo.ts` → `__tests__/services/foo.test.ts`).
- Infer types from schemas (`z.infer<typeof schema>`) instead of duplicating type definitions.
- Do not add comments that merely narrate what the code does — only explain *why* when the intent is non-obvious.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ItsMeOX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
