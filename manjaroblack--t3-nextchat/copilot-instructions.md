## t3-nextchat

> - **Runner:** `Vitest`


# Testing Guidelines

## Tools

- **Runner:** `Vitest`
- **Library:** `React Testing Library`

## Philosophy

Test user behavior, not implementation details. Query the DOM as a user would, using accessible selectors.

## Guidelines

### 1. Structure

**Rule:** Use the Arrange-Act-Assert (AAA) pattern. `describe()` blocks group tests for a component/hook; `it()` blocks test a single behavior.

**Example:**

```typescript
// Arrange
render(<Component />);
// Act
const element = screen.getByText('Submit');
// Assert
expect(element).toBeInTheDocument();
```

### 2. Queries

**Rule:** Prioritize accessible queries (`getByRole`, `getByLabelText`, `getByText`). Use `getByTestId` only as a last resort.

**Example:** `screen.getByRole('button', { name: /submit/i })`

### 3. User Interactions

**Rule:** Use `@testing-library/user-event` to simulate user interactions, as it more closely mimics real browser events.

**Example:** `await userEvent.click(screen.getByRole('button'))`

### 4. Mocking

**Rule:** Use `vi.fn()` for mocks/spies. Mock API hooks (tRPC) to provide controlled data and prevent network requests.

**Example:** `const mockFn = vi.fn().mockReturnValue(true);`

### 5. Custom Hooks

**Rule:** Test custom hooks with `renderHook`. Wrap state updates in `act()`.

**Example:** `const { result } = renderHook(() => useMyHook());`

### 6. Coverage

**Rule:** Focus on meaningful tests for critical user paths. Do not chase 100% coverage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manjaroblack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
