## rama

> - When a user asks in a language other than English, reiterate the request in English before proceeding


## Language Requirements
- When a user asks in a language other than English, reiterate the request in English before proceeding
- Always think, answer, and perform in English

## Code Quality Standards

### Core Principles
- Don't write unused code - ensure everything written is utilized in the project
- Prioritize readability for human understanding over execution efficiency
- Maintain long-term maintainability over short-term optimization
- Avoid unnecessary complexity - implement simple solutions unless complexity is truly required
- Follow Linus Torvalds' clean code principles: keep it simple, make code readable like prose, avoid premature optimization, express intent clearly, minimize abstraction layers

### Documentation Standards
- Comments must explain 'what' (business logic/purpose) and 'why' (reasoning/decisions), not 'how'
- Avoid over-commenting - excessive comments indicate poor code quality
- Function comments must explain purpose and reasoning, placed at function beginnings
- Well-written code should be self-explanatory through meaningful names and clear structure

### Development Process
1. **Understand first**: Use available tools to understand data structures before implementation
2. **Design data structures**: Good data structures lead to good code
3. **Define interfaces**: Specify all input/output structures before writing logic
4. **Define functions**: Create all function signatures before implementation
5. **Implement logic**: Write implementation only after structures and definitions are complete

### Quality Guidelines
- Avoid over-engineering - focus on minimal viable solutions meeting acceptance criteria
- Only create automated tests if explicitly required
- Never add functionality "just in case" - implement only what's needed now

## Ruby-Specific Guidelines
- Follow the Ruby Style Guide: https://rubystyle.guide/
- Use RuboCop for static code analysis
- Prefer single quotes for strings unless string interpolation is needed
- Use 2 spaces for indentation (no tabs)
- Use snake_case for method and variable names
- Use CamelCase for class and module names
- Keep lines under 80 characters
- Use frozen string literals: `# frozen_string_literal: true`

## Testing
- Use RSpec for testing
- Follow the RSpec best practices
- Keep tests focused and isolated
- Use descriptive test names
- Prefer `let` over instance variables in tests
- Use factories (like FactoryBot) for test data

## Git Workflow
- Write clear, descriptive commit messages
- Keep commits atomic and focused
- Reference issue numbers in commit messages when applicable
- Rebase feature branches before merging
- Keep pull requests small and focused

## Decision-Making Framework
Apply these principles systematically:
1. Gather Complete Information
2. Multi-Perspective Analysis
3. Consider All Stakeholders
4. Evaluate Alternatives
5. Assess Impact & Consequences
6. Apply Ethical Framework
7. Take Responsibility
8. Learn & Adapt

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itisbryan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
