## guidelines

> Code Patterns & Guidelines


# Cursor AI Assistant Rules

## Assistant Behavior

- Be terse and direct in responses
- Ask clarifying questions before implementing complex features
- Anticipate needs and suggest improvements proactively
- Focus on high-quality, production-ready code
- Use analogies when explaining complex concepts

## Code Generation Principles

### Quality Standards
- Generate code that could ship to production
- Follow established patterns in the codebase
- Maintain consistency with existing architecture
- Every piece of code should be testable
- Documentation is not optional

### Style Guidelines
- Respect .prettierrc and linting configurations
- Prefer clarity over cleverness
- Use descriptive names that reveal intent
- Keep line lengths reasonable
- Consistent indentation and formatting

### Architecture Focus
- Think in systems, not just functions
- Consider scalability implications
- Design for maintainability
- Separate concerns appropriately
- Make dependencies explicit

## Project Awareness

### Context Loading
Always check for `.cursor/PROJECT_CONTEXT.md` which contains:
- Current architectural patterns
- Technology stack decisions
- Team conventions and preferences
- Recent design decisions
- Performance considerations
- Known technical debt

### File Organization
- Follow existing project structure
- Keep related code together
- Use standard naming conventions
- Maintain clear module boundaries

## Code Patterns

### Error Handling
- Fail fast with clear messages
- Use appropriate error types
- Handle errors at the right level
- Provide actionable error messages
- Log errors appropriately

### Testing Approach
- Write tests alongside implementation
- Test edge cases explicitly
- Use descriptive test names
- Keep tests simple and focused
- Mock external dependencies

### Documentation
- Document why, not what
- Include examples for complex APIs
- Keep docs in sync with code
- Use consistent documentation format
- Document assumptions and constraints

## Language-Specific Rules

### Ruby on Rails
Refer to `rails-rules.md` for:
- Rails conventions and patterns
- ActiveRecord best practices
- Testing with RSpec
- API design guidelines


## Performance & Security

### Performance
- Consider algorithmic complexity
- Profile before optimizing
- Cache strategically
- Use appropriate data structures
- Avoid premature optimization

### Security
- Sanitize all inputs
- Use secure defaults
- Follow OWASP guidelines
- Never expose sensitive data
- Keep dependencies updated

## AI Assistant Guidelines

### When Generating Code
1. Check `PROJECT_CONTEXT.md` first
2. Follow existing patterns in codebase
3. Include appropriate tests
4. Add necessary documentation
5. Consider edge cases

### When Explaining Concepts
- Use clear analogies
- Provide concrete examples
- Link to authoritative sources
- Keep explanations concise
- Focus on practical application

### When Suggesting Improvements
- Consider the bigger picture
- Propose incremental changes
- Explain trade-offs clearly
- Respect existing constraints
- Suggest best-in-class solutions

---

*Note: Language-specific and PROJECT_CONTEXT rules override these defaults.*

---
> Source: [levifig/rails-instructions](https://github.com/levifig/rails-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
