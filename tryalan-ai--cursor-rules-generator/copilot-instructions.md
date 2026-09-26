## code-quality

> Code quality standards and best practices

# Code Quality Standards

## Code Quality Rules
- Strict TypeScript mode
- No `any` types allowed
- Enforce error boundaries

## Documentation Requirements
- Level: minimal
- Comment Styles: Inline comments for complex logic, TODO comments for future improvements
- README Requirements: Setup and installation instructions, API documentation

## Best Practices
- Write self-documenting code
- Follow consistent naming conventions
- Implement proper error handling
- Maintain test coverage
- Use proper type safety

## Code Examples:

```typescript
// Good: Following quality standards
interface UserData {
  id: string;
  name: string;
  email: string;
}


// Good: Proper typing
function processUser(user: UserData): ProcessedUser {
  return {
    ...user,
    displayName: user.name.trim()
  };
}

// Bad: Using any
function badProcessUser(user: any): any {
  return user;
}



// Good: Error boundary implementation
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
}

```

---
> Source: [tryalan-ai/cursor-rules-generator](https://github.com/tryalan-ai/cursor-rules-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
