## n1-vgs-components

> This guide is designed for developers using Claude Code to work on or with the `@h4b-dev/n1-vgs-components` package.

# Claude Code Integration Guide for n1-vgs-components

This guide is designed for developers using Claude Code to work on or with the `@h4b-dev/n1-vgs-components` package.

## Quick Start with Claude Code

### Asking Claude Code about this project

You can ask Claude Code questions about the codebase in natural language. Claude Code will automatically search and understand the repository structure. Examples:

```
"How does the CollectForm component work?"
"What are all the props that CollectForm accepts?"
"Show me where VGS Collect is initialized"
"How do I add support for new card brands?"
"What are the validation rules for card fields?"
```

## Using Claude Code for Development Tasks

### Understanding the Codebase

Ask Claude Code to explore specific areas:

```
"Show me the component hierarchy"
"What's in the hooks directory?"
"How are the tests structured?"
"Explain the build configuration"
```

Claude Code will search the repository and provide accurate information about the implementation details.

### Making Changes

When you want to make changes, you can ask Claude Code to:

1. **Fix bugs**: "There's a bug where the form doesn't reset after submission. Can you fix it?"
2. **Add features**: "Add support for American Express cards"
3. **Improve code**: "Refactor the onSubmitCallback function to be cleaner"
4. **Write tests**: "Write a test for the CollectFormWrapper component loading behavior"
5. **Update documentation**: "Update the README to include TypeScript usage examples"

### Code Review

Ask Claude Code to review your code:

```
"Review this change for security issues"
"Are there any performance problems in this implementation?"
"Does this follow the project's patterns?"
```

## Project Structure for Reference

When working with Claude Code, it helps to understand the key files:

### Main Component Files
- `src/components/CollectForm/CollectForm.jsx` - Core form implementation
- `src/components/CollectFormWrapper/CollectFormWrapper.jsx` - VGS script loader
- `src/index.js` - Public API exports

### Configuration & Build
- `vite.config.js` - Build configuration
- `vitest.config.js` - Test configuration
- `package.json` - Dependencies and scripts

### Tests
- `src/test/CollectForm.test.jsx` - Main component tests
- `src/test/CollectFormWrapper.test.jsx` - Wrapper tests
- `src/test/mockData.js` - Test utilities

## Common Claude Code Workflows

### Workflow 1: Adding a New Feature

1. Ask Claude Code: "What are all the places I need to update to add support for [new card brand]?"
2. Claude Code will search and identify all relevant files
3. Ask: "Can you implement this change?"
4. Claude Code will update all necessary files
5. Ask: "Write tests for this new feature"
6. Ask: "Is the documentation up to date?"

### Workflow 2: Understanding a Bug

1. Ask: "Why does [specific behavior] happen?"
2. Claude Code will find the relevant code and explain the flow
3. Ask: "How can we fix this?"
4. Claude Code will propose and implement a fix
5. Ask: "Are there any edge cases we should handle?"

### Workflow 3: Refactoring

1. Ask: "How can we improve the code in [specific file]?"
2. Claude Code will analyze and suggest improvements
3. Ask: "Can you refactor this?"
4. Claude Code will make the changes while maintaining functionality
5. Ask: "Does this need new tests?"

### Workflow 4: Checking Compatibility

1. Ask: "What's the current React version requirement?"
2. Ask: "Can we upgrade to React 20?"
3. Claude Code will search dependencies and compatibility concerns
4. Ask: "What changes would we need to make?"
5. Ask: "Are there any breaking changes?"

## Development Commands to Use with Claude Code

When working on development, you can ask Claude Code to run commands:

```
"Run the tests and show me if anything fails"
"Build the project and show me the output"
"Run the linter and fix any issues it finds"
"Start the development environment"
```

## VGS-Specific Notes for Claude Code

The component integrates with VGS Collect. When asking Claude Code about VGS-related functionality:

- **VGS Configuration**: Stored in environment variables in `.env.production`
- **Three Environments**: dev (for development), sandbox (for staging), prod (for production)
- **Security**: Card data never touches your application - it's tokenized by VGS
- **Vaults**: Each environment has a different VGS vault ID

### Asking about VGS Integration

```
"How does the component tokenize card data?"
"What are the environment-specific settings?"
"How does the authorization token get used?"
"What's the submission flow from form to VGS?"
```

## Testing with Claude Code

Ask Claude Code to help with tests:

```
"Write a test that verifies the form submits with the correct data"
"What's the coverage for CollectForm?"
"Write an integration test for the complete card creation flow"
"Can you add tests for error handling?"
```

## Debugging with Claude Code

When something isn't working:

1. Ask: "What could cause [specific behavior]?"
2. Ask: "Add some debug logging to help troubleshoot [issue]"
3. Ask: "Can you trace through the code path for [specific flow]?"
4. Ask: "What validation happens for [specific field]?"

## Package Version Management

When preparing to deploy:

1. Ask: "What version should the next release be?"
2. Ask: "Update the version in package.json"
3. Ask: "What changed since the last version?"
4. Ask: "Generate release notes for version X.Y.Z"

## Documentation with Claude Code

Ask Claude Code to help maintain documentation:

```
"Update the MANUAL.md to document [new feature]"
"Add examples for [specific use case]"
"Update the README with current dependencies"
"Add a troubleshooting section for [issue]"
```

## Security Considerations

When working on security-related changes, ask Claude Code to:

1. Review code for vulnerabilities
2. Check that sensitive data (tokens) isn't logged
3. Verify that form inputs are properly validated
4. Ensure external scripts are loaded securely

Example: "Review the component for any security vulnerabilities, especially around token handling and form input validation"

## Performance Considerations

Ask Claude Code about performance:

```
"Are there any performance bottlenecks in this component?"
"Can we optimize the VGS script loading?"
"How can we reduce the bundle size?"
"Are there any unnecessary re-renders?"
```

## Tips for Best Results

1. **Be specific**: "Fix the bug where..." is better than "Fix bugs"
2. **Provide context**: "I'm trying to add Amex support..." helps Claude Code understand intent
3. **Ask follow-ups**: "Now add tests for this" after an implementation
4. **Reference files**: "In src/components/CollectForm/CollectForm.jsx..." for clarity
5. **Ask for explanations**: "Why did you make that change?" helps you learn

## Integration with Your Workflow

- **Before making changes**: Ask Claude Code to explain the affected code
- **While making changes**: Ask Claude Code to implement and verify
- **After making changes**: Ask Claude Code to write tests and update docs
- **Before committing**: Ask Claude Code to review the changes

## Getting Help

If something isn't working as expected:

1. Ask Claude Code to explain what's happening
2. Ask for step-by-step debugging
3. Ask to try a different approach
4. Reference specific files and line numbers for clarity

## Example Questions

Here are some real questions you might ask Claude Code:

```
"Can I use this component with TypeScript?"
"How do I test the onError callback?"
"What happens if the VGS script fails to load?"
"How can I customize the form styling?"
"What's the difference between onSubmit and onUpdate?"
"How do I make this work with Next.js?"
"Can the component handle multiple card submissions?"
"What validation happens before submission?"
```

## Integration with GitHub

Claude Code can help with git workflows:

```
"Show me what changed in this file"
"Create a commit message for these changes"
"What's the diff between this branch and main?"
"Can you revert this change?"
```

## Next Steps

- Review [MANUAL.md](./MANUAL.md) for package usage instructions
- Check [README.md](./README.md) for development setup details
- Look at the test files to understand testing patterns
- Explore the component stories in `.ladle/` for usage examples

Remember: Claude Code is here to help you understand, modify, and improve this codebase. Don't hesitate to ask questions!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/h4b-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
