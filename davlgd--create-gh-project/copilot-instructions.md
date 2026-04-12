## create-gh-project

> This project is **create-gh-project**, a CLI tool to bootstrap AI-ready GitHub projects with all essential files and configurations.

# project-init - Development Instructions

## Project Overview

This project is **create-gh-project**, a CLI tool to bootstrap AI-ready GitHub projects with all essential files and configurations.

**Version**: 0.1.2 | **License**: Apache-2.0 | **Author**: davlgd

## Core Development Principles

### 🎯 KISS (Keep It Simple, Stupid)

- **Simplicity over cleverness**: Write code that is easy to understand and maintain
- **Minimal dependencies**: Only add dependencies when absolutely necessary (Commander.js only)
- **Clear naming**: Use descriptive names for variables, functions, and classes
- **Avoid over-engineering**: Solve the current problem, not all possible future problems

### 🔄 DRY (Don't Repeat Yourself)

- **Extract common functionality**: Create reusable functions, classes, and modules
- **Configuration centralization**: Keep configuration in dedicated files
- **Template reuse**: Use templates and generators for repetitive code patterns
- **Documentation consistency**: Maintain consistent documentation patterns

### 🏗️ SOLID Principles

- **Single Responsibility**: Each class/function should have one reason to change
- **Open/Closed**: Open for extension, closed for modification
- **Liskov Substitution**: Derived classes must be substitutable for their base classes
- **Interface Segregation**: Many specific interfaces are better than one general-purpose interface
- **Dependency Inversion**: Depend on abstractions, not concretions

## Code Quality Standards (NON-NEGOTIABLE)

### 🧪 Testing Requirements

- **100% test coverage** for core business logic
- **Integration tests** for all public APIs and critical paths
- **No mocking shortcuts** - tests must verify actual behavior
- **Test-first approach** - write tests before implementation when possible
- **All tests must pass** - no exceptions, no workarounds
- **Edge case coverage** - test boundary conditions and error scenarios

### 🔍 Quality Gates

Before any commit or merge:

1. **All tests pass** (`bun run test`)
2. **Code formatting** (`bun run format`)
3. **Linting passes** (`bun run lint`)
4. **Type checking passes** (`bun run typecheck`)
5. **Full CI pipeline** (`bun run ci`)
6. **Security validation** - no injection vulnerabilities, input validation

### 🛡️ Security Requirements (CRITICAL)

- **Input validation**: All user inputs must be validated and sanitized
- **Safe command execution**: Use parameter arrays, never string concatenation for commands
- **Directory protection**: Never overwrite existing directories without explicit consent
- **No secret exposure**: Never log, store, or expose sensitive information
- **Injection prevention**: All external command calls must use typed parameter arrays

## Architecture

### Modular Design

- **src/index.ts** - CLI entry point with Commander.js (positional arguments)
- **src/services/** - Business logic services (UI, templates, filesystem)
- **src/templates/** - Template generators for each file type
- **src/github-setup.ts** - GitHub integration with SSH interactive support
- **src/project-creator.ts** - Project structure orchestration
- **src/types.ts** - Type definitions and interfaces
- **.claude/agents/** - Specialized Claude Code agents for quality, docs, releases

### Template System Philosophy

- **One template per file type**: Each template has a single responsibility
- **Dynamic content generation**: Use TypeScript template literals with validation
- **Consistent patterns**: All templates follow the same structure and naming
- **Security first**: All generated content is validated and escaped

## Key Commands

```bash
bun run dev          # Hot reload development
bun run test         # Run all tests (29+ tests)
bun run typecheck    # TypeScript validation
bun run lint         # Code quality check
bun run format       # Format code with Prettier
bun run ci           # Complete validation pipeline
bun run clean        # Clean artifacts and temp files
```

## Development Workflow

### Adding New Features

1. **Write tests first** - Define expected behavior
2. **Implement with security** - Validate inputs, use safe patterns
3. **Format code** - `bun run format` to apply consistent styling
4. **Run full CI** - `bun run ci` must pass
5. **Update documentation** - README, CLAUDE.md, help text

### Security-First Development

- **Command injection prevention**: Always use `['command', 'arg1', 'arg2']` format
- **Input validation**: Validate project names, paths, and all user inputs
- **Directory safety**: Check existence and permissions before operations
- **Error handling**: Provide clear, actionable error messages

### Template Development

- **Single responsibility**: One template per file type
- **Input validation**: Validate ProjectConfig before generation
- **Testing**: Test with various configurations and edge cases
- **Consistency**: Follow existing template patterns

## AI Assistant Guidelines

### 🎯 Token Efficiency Requirements (CRITICAL)

1. **Use local tools first**: Read, LS, Grep, Glob, Bash over asking questions
2. **Batch operations**: Multiple tool calls in single responses
3. **Avoid redundant searches**: Cache information within conversations
4. **Direct file access**: Use Read tool for specific files

### Project-Specific Patterns

- **Template consistency**: All generated files follow similar patterns
- **Error handling**: Clear messages with actionable solutions
- **User experience**: Focus on helpful output and guidance
- **Security validation**: Always validate inputs and command parameters

### 🧪 User Experience Testing (CRITICAL)

**Always test as a real user before considering a feature complete:**

1. **Fresh User Perspective**: Test the application as someone discovering it for the first time
2. **Error Scenarios**: Intentionally make mistakes to test error handling and recovery
3. **Documentation Validation**: Ensure a beginner can understand and use the tool in minutes
4. **End-to-End Flows**: Test complete user journeys, not just individual features
5. **Edge Cases**: Test with unusual inputs, missing dependencies, different environments

**UX Testing Checklist:**

- [ ] Can a new user understand the tool's value in 30 seconds?
- [ ] Are error messages helpful and actionable?
- [ ] Does the tool provide clear next steps after completion?
- [ ] Are all generated files properly documented?
- [ ] Does the tool work without surprises or unexpected behavior?

**After every significant change, run through these user scenarios:**

- First-time user with no prior knowledge
- User with invalid inputs (wrong names, missing permissions, etc.)
- User trying to understand what the tool created
- User following the generated documentation

## Common Tasks

### Adding Templates

1. Create function in `src/templates/newtemplate.ts`
2. Import in `src/services/template-generator.ts`
3. Add comprehensive tests in `test/templates.test.ts`
4. Test with various ProjectConfig scenarios

### CLI Options

1. Update `src/index.ts` argument definitions
2. Add validation logic with clear error messages
3. Update help text and README documentation
4. Add CLI integration tests

### Security Updates

1. **Review command execution**: Ensure parameter arrays used
2. **Validate all inputs**: Add regex/format validation
3. **Test edge cases**: Invalid characters, long inputs, injection attempts
4. **Update tests**: Security scenarios must be covered

## Performance & Reliability

### Constraints

- **Minimal dependencies**: Commander.js runtime dependency with essential dev tools (Prettier, ESLint)
- **Fast execution**: Templates generated in memory, batch file operations
- **Cross-platform**: Works on macOS, Linux, Windows
- **Reliability**: Synchronous file operations for consistency

### Best Practices

- **Memory efficiency**: Generate templates on-demand, don't cache
- **Error recovery**: Graceful handling of git/gh CLI failures
- **User feedback**: Clear progress indicators and next steps
- **Clean exits**: Proper process.exit() with meaningful codes

## Release History

### v0.1.2 (Current)

- **UX Improvements**: Better message display order and GitHub integration
- **SSH Support**: Interactive mode for SSH passphrase input
- **Aesthetic Enhancements**: Improved line breaks and ellipsis characters
- **Documentation**: Corrected npm/bun create syntax throughout

### v0.1.1

- **SSH Authentication**: Fixed SSH passphrase blocking issues
- **Error Handling**: Better GitHub repository creation error management

### v0.1.0

- **Initial Release**: Complete CLI tool with project scaffolding
- **GitHub Integration**: Automated repository creation
- **AI-Ready Setup**: Claude agents and comprehensive documentation

---

**Remember**: This tool creates the foundation for AI-ready development projects. Every generated file should follow current best practices and be immediately ready for production use.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davlgd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davlgd)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
