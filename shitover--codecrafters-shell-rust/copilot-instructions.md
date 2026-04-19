## codecrafters-shell-rust

> - Implement modular design with separate components for:

# Project: Rust Shell Implementation (CodeCrafters)

## Code Architecture
- Implement modular design with separate components for:
  - Command parsing and lexing
  - Process management
  - Built-in commands
  - REPL (Read-Eval-Print Loop)
- Use trait-based abstractions for command execution
- Follow POSIX compliance standards for shell behavior

## Rust-Specific Standards
- Use Rust 2021 edition
- Implement proper error handling with custom Error types
- Use `Result` and `Option` types appropriately
- Prefer ownership and borrowing over raw pointers
- Use strong typing with minimal use of `unsafe` blocks

## Style Guidelines
- Follow the official Rust style guide (rustfmt defaults)
- Maximum line length: 100 characters
- Use 4 spaces for indentation
- Organize imports in groups:
  1. Standard library
  2. External crates
  3. Local modules
- Name conventions:
  - Use snake_case for functions and variables
  - Use PascalCase for types and traits
  - Use SCREAMING_SNAKE_CASE for constants

## Shell-Specific Best Practices
- Implement proper signal handling (SIGINT, SIGTERM, etc.)
- Use appropriate system calls for process management
- Handle command line parsing according to POSIX rules
- Implement proper quoting and escape character handling
- Use efficient string manipulation for command processing

## Error Handling
- Create custom error types for different failure scenarios
- Implement detailed error messages for debugging
- Use the `thiserror` crate for error definitions
- Properly propagate errors up the call stack
- Handle all Result types explicitly

## Testing
- Write unit tests for all modules
- Create integration tests for shell commands
- Test edge cases in command parsing
- Implement tests for built-in commands
- Test process management functionality
- Aim for >80% test coverage

## Documentation
- Use rustdoc comments for all public items
- Document POSIX compliance details
- Include examples in documentation
- Keep inline comments focused on "why" not "what"
- Maintain a comprehensive README.md

## Performance Considerations
- Minimize allocations in hot paths
- Use appropriate data structures for command history
- Implement efficient process spawning
- Consider memory usage in long-running sessions
- Profile code for performance bottlenecks

## Security
- Validate all user inputs
- Handle environment variables safely
- Implement proper permission checking
- Sanitize command arguments
- Handle sensitive data appropriately

## Dependencies
- Minimize external dependencies
- Prefer well-maintained crates with good security records
- Use specific versions in Cargo.toml
- Review dependency updates regularly
- Document why each dependency is needed

## Version Control
- Use meaningful commit messages
- Keep commits focused and atomic
- Use feature branches for development
- Write detailed PR descriptions
- Regular commits with clear progress

Remember to:
- Follow POSIX specifications strictly
- Consider cross-platform compatibility
- Focus on robust error handling
- Maintain clear documentation
- Prioritize security in implementation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shitover) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
