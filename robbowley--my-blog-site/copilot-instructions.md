## my-blog-site

> This document outlines our development standards and best practices. These guidelines emphasize pragmatic, maintainable code that solves real problems without unnecessary complexity.

# Development Standards

This document outlines our development standards and best practices. These guidelines emphasize pragmatic, maintainable code that solves real problems without unnecessary complexity.

## Working Ethos & Collaboration

### Critical Thinking
- Ask clarifying questions before jumping to conclusions
- Challenge assumptions—especially around requirements and edge cases
- Consider the broader context and implications of changes
- Think through failure modes and edge cases

### Communication
- Be explicit about assumptions and trade-offs
- Share knowledge and context with team members
- Document key decisions and their rationale
- Flag potential issues early

### Risk Management
- Flag risky, complex, or brittle implementations
- Identify potential failure points and dependencies
- Consider maintenance and operational implications
- Suggest alternatives when simpler approaches exist

### AI Assistant Collaboration
When working with AI assistants:
- Expect and encourage clarifying questions
- Provide context and constraints upfront
- Be specific about requirements and expectations
- Review and validate generated code and suggestions
- Consider AI suggestions as recommendations, not mandates

### Better Alternatives
Always be open to suggesting and considering better approaches when:
- A simpler solution exists
- A more maintainable approach is possible
- A more robust implementation can be achieved
- Technical debt can be avoided
- Performance can be significantly improved

## Core Principles

### Code Organization
- Follow Next.js conventions for project structure
  ```
  ✅ Good:
  src/
    app/           # Next.js app router pages and layouts
    components/    # Reusable UI components
    utils/         # Utility functions and helpers
    lib/          # Third-party library configurations
    types/        # TypeScript type definitions
    styles/       # Global styles and theme
    hooks/        # Custom React hooks
    tests/        # Test files matching source structure

  ❌ Avoid:
  features/
    feature-a/
    feature-b/
  ```

- Keep related code close together within the appropriate directory
- Use clear, intention-revealing names that reflect business concepts
- Place components close to where they are used when they are specific to a feature
- Use the app directory for routing and page-specific components

### Design Philosophy
- Prefer composition over inheritance
  - Build complex behavior by combining simple, focused pieces
  - Avoid deep inheritance hierarchies
  - Use interfaces to define contracts

- Apply Single Responsibility Principle pragmatically
  - Each unit should do one job well
  - "One job" means "one reason to change"
  - Balance between too granular and too coupled

- Think in terms of responsibilities and boundaries
  - Focus on what each module is responsible for
  - Define clear interfaces between modules
  - Hide implementation details

- Avoid premature abstraction
  ```
  // ✅ Good: Simple and direct
  function calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  }

  // ❌ Avoid: Over-engineered
  class PriceCalculationStrategy {
    calculate(items) {
      throw new Error('Abstract method')
    }
  }
  class StandardPriceCalculator extends PriceCalculationStrategy {
    calculate(items) {
      return items.reduce((sum, item) => sum + item.price * item.quantity, 0)
    }
  }
  ```

### Code Structure

#### Functions and Components
- Write modular, testable functions
  - Pure functions where possible
  - Clear inputs and outputs
  - Minimal side effects
  - Easy to test in isolation

- Keep functions focused
  ```
  // ✅ Good: Single responsibility
  function validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }

  function notifyUser(user, message) {
    return sendEmail(user.email, message)
  }

  // ❌ Avoid: Mixed responsibilities
  function validateAndNotifyUser(email, message) {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      throw new Error('Invalid email')
    }
    return sendEmail(email, message)
  }
  ```

- Model real behaviors
  ```
  // ✅ Good: Models real-world concept
  const shoppingCart = {
    addItem(item, quantity) { /* ... */ },
    removeItem(itemId) { /* ... */ },
    updateQuantity(itemId, quantity) { /* ... */ },
    getTotal() { /* ... */ }
  }

  // ❌ Avoid: Arbitrary technical abstraction
  const cartManager = {
    executeOperation(operation) { /* ... */ },
    calculateMetrics() { /* ... */ }
  }
  ```

#### State Management
- Keep state as local as possible
- Lift state up only when necessary
- Choose appropriate state management tools based on:
  - Scale of application
  - Team experience
  - Performance requirements
  - Development workflow

## Testing Strategy

### Unit Tests
- Focus on pure logic
  - Utilities
  - Helpers
  - Business logic
  - State management

```
// ✅ Good: Testing pure business logic
describe('calculateOrderTotal', () => {
  it('correctly calculates total with multiple items', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 15, quantity: 1 }
    ]
    expect(calculateOrderTotal(items)).toBe(35)
  })
})
```

### Integration Tests
- Test key user flows
- Focus on critical business paths
- Test real user interactions
- Use appropriate testing tools for your stack

```
// ✅ Good: Testing key user flow
test('user can add item to cart and checkout', async () => {
  await goToProducts()
  await addItemToCart('test-item')
  await checkout()
  await expectOrderConfirmed()
})
```

### Testing Guidelines
- Aim for confidence, not coverage
  - Test what might break
  - Test edge cases
  - Test error conditions

- Mock external dependencies appropriately
  ```
  // ✅ Good: Mocking external API
  mockApi.fetchUserData.mockResolvedValue({
    id: 1,
    name: 'Test User'
  })

  test('handles API error gracefully', async () => {
    mockApi.fetchUserData.mockRejectedValue(new Error('Network error'))
    // Test error handling
  })
  ```

- Test behavior, not implementation
  ```
  // ✅ Good: Testing behavior
  test('filters active todos', () => {
    const todos = createTodoList()
    todos.setFilter('active')
    expect(todos.getVisible()).toContainEqual(
      expect.objectContaining({ status: 'active' })
    )
  })

  // ❌ Avoid: Testing implementation details
  test('internal state is updated', () => {
    const todos = createTodoList()
    expect(todos._filterState).toBe('active')
  })
  ```

## Code Review Guidelines

When reviewing code, focus on:
- Does it solve the business problem?
- Is it maintainable?
- Is it testable?
- Are responsibilities clear?
- Is abstraction level appropriate?
- Are tests meaningful?
- Have edge cases been considered?
- Are there simpler alternatives?

## Documentation

- Document the why, not the what
- Keep documentation close to code
- Use type hints and interfaces as documentation where applicable
- Include examples for complex logic
- Document architectural decisions
- Maintain a decision log for significant choices

Remember: These standards are guidelines, not rules. Use judgment and adapt to specific situations. The goal is to write maintainable, understandable code that solves real problems effectively.

## Technology Stack

Note: Specific technology choices (languages, frameworks, tools) will be documented separately once decided. These standards focus on principles that apply regardless of the specific technologies chosen.

## Version Control

### Git Workflow

1. **Regular Check-ins**
   - Commit changes at least daily
   - Create meaningful commit messages that describe what and why
   - Break large changes into smaller, logical commits
   - Push to remote repository regularly to ensure backup
   - Never leave work uncommitted at the end of the day

2. **Branch Strategy**
   - `main` branch is always production-ready
   - Create feature branches for new work
   - Use descriptive branch names (e.g., `feature/user-auth`)
   - Delete branches after merging

3. **Commit Messages**
   - Use present tense ("Add feature" not "Added feature")
   - Keep first line under 50 characters
   - Add detailed description if needed
   - Reference issue numbers when applicable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robbowley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
