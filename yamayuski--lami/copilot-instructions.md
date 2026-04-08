## lami

> This repository contains **lami**, a next-generation Git CLI tool implemented in Zig. The project aims to enhance and modernize the Git command-line experience.

# Google Gemini CLI Instructions for lami

## Project Context
This repository contains **lami**, a next-generation Git CLI tool implemented in Zig. The project aims to enhance and modernize the Git command-line experience.

## Development Specifications

### Language Requirements
- **Programming Language**: Zig
- **Required Version**: 0.14.1
- Ensure all code suggestions use Zig 0.14.1 syntax and features
- Follow Zig language conventions and best practices

### Platform Support
- **Primary Platform**: linux/amd64
- **Future Plans**: Additional platform builds will be added
- All development should prioritize linux/amd64 compatibility
- Design with cross-platform expansion in mind

## Technical Guidelines

### Zig Development Best Practices
- Use Zig's compile-time features effectively
- Leverage Zig's error handling system (`!` and `catch`)
- Utilize appropriate allocators for memory management
- Follow Zig's naming conventions (snake_case for variables/functions)
- Use `zig fmt` for consistent code formatting

### Build System
- Use `zig build` for compilation
- Test with `zig test` for unit tests
- Build outputs go to `zig-out/` directory
- Cache files are stored in `.zig-cache/`

### Git Tool Considerations
- This is a Git-adjacent tool, so understand Git internals
- Consider Git repository structure and operations
- Test against real Git repositories
- Handle Git edge cases and error conditions
- Ensure compatibility with existing Git workflows

### Code Quality
- Write self-documenting code with clear variable names
- Add comments for complex algorithms or Git-specific logic
- Handle errors gracefully using Zig's error system
- Validate input and provide helpful error messages

## When Providing Assistance
- Focus on Zig 0.14.1 compatibility and features
- Prioritize linux/amd64 platform requirements
- Consider Git interoperability and workflows
- Suggest efficient Zig standard library usage
- Help implement robust error handling
- Assist with proper memory management patterns
- Provide guidance on Zig's compile-time programming features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yamayuski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yamayuski)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
