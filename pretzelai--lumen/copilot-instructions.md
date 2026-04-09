## lumen

> - **NO explanatory comments that simply restate what the code does**

## Code Quality Guidelines

### Comments and Documentation
- **NO explanatory comments that simply restate what the code does**
- **NO obvious comments** like `// increment counter` for `counter++`
- **NO redundant comments** that duplicate clear variable/function names
- Only add comments for:
  - Complex business logic or algorithms
  - Non-obvious workarounds or edge cases
  - API integrations with specific requirements
  - Performance optimizations
  - Security considerations
  - Temporary code that needs attention

### Code Style
- Write self-documenting code with clear variable and function names
- Prefer descriptive names over comments when possible
- Keep functions small and focused on single responsibilities
- Use meaningful commit messages instead of inline TODO comments

### Examples of What NOT to Do
```javascript
// BAD - Don't add these types of comments
const user = getUser(); // Get the user
if (user.isActive) { // Check if user is active
    user.login(); // Log the user in
}
```

### Examples of What TO Do
```javascript
// GOOD - Clean, self-documenting code
const user = getUser();
if (user.isActive) {
    user.login();
}

// GOOD - Only comment when adding real value
// Workaround: API returns 200 even for validation errors
// so we need to check the response body for error field
if (response.status === 200 && response.body.error) {
    throw new ValidationError(response.body.error);
}
```

### General Principles
- Code should be readable without comments
- If you feel a comment is needed, first try to refactor for clarity
- Focus on **why** something is done, not **what** is being done
- Prioritize clean, maintainable code over verbose explanations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pretzelai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pretzelai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
