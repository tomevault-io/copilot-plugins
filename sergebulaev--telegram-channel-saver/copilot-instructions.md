## telegram-channel-saver

> Telegram Channel Saver is a Python tool for saving and analyzing Telegram channel content. The application connects to Telegram API using user credentials, allows browsing and selecting channels/groups, and provides functionality to download and store messages, track users, and search through saved content.

# Telegram Channel Saver

## Project Description
Telegram Channel Saver is a Python tool for saving and analyzing Telegram channel content. The application connects to Telegram API using user credentials, allows browsing and selecting channels/groups, and provides functionality to download and store messages, track users, and search through saved content.

### Key Features
- Save channel messages with reactions and media information
- Track channel users and their activity
- Search through saved messages by text, date, or ID
- Support for multiple Telegram accounts
- Message download with rate limiting and error handling
- Detailed statistics about saved content

### File Structure
```
/telegram-channel-saver/
  ├── LICENSE             # MIT License
  ├── README.md           # Project documentation
  ├── docs/
  │   └── telethon.txt    # Telethon library documentation
  ├── requirements.txt    # Project dependencies
  ├── saver.py            # Main application code
  ├── CLAUDE.md           # Best practices and code guidelines
  ├── tools/              # Testing tools and utilities (Claude's workspace)
  │   ├── venv/           # Separate virtual environment for tools
  │   └── *.py            # Test scripts and utilities
  └── temp/               # Storage directory for data and sessions
      └── channel_saver/  # Application data storage location
```

# Claude AI Development Rules

## API Documentation
- **Always use context7** to get the most recent API documentation before implementing features
- Do not rely on training data for library APIs - fetch current docs via context7
- When working with Telethon, Telegram API, or any external library, query context7 first

## Tools Directory
- Claude can create test tools and utilities in the `tools/` directory
- Use the separate venv located at `tools/venv/` for running test scripts
- Activate with: `source tools/venv/bin/activate`
- Install dependencies in tools venv: `tools/venv/bin/pip install <package>`
- Run tools with: `tools/venv/bin/python tools/<script>.py`
- Tools are for testing pipelines, debugging, and validating implementations
- Keep tools separate from main application code

# Clean Code Best Practices

## General Principles
- Keep functions and modules small and focused on a single responsibility
- Use descriptive variable and function names
- Keep modules under 500 lines of code
- Write code that is easy to read and understand for other developers
- Avoid code duplication through proper abstraction

## Function Design
- Functions should do one thing and do it well
- Keep functions short (preferably under 20 lines)
- Minimize the number of arguments (aim for 3 or fewer)
- Avoid side effects when possible
- Return early to reduce nesting

## Variable Naming
- Use meaningful and pronounceable variable names
- Use consistent naming conventions
- Make sure names reflect what the variable contains
- Use nouns for variables and verbs for functions

## Comments and Documentation
- Code should be self-documenting
- Comments should explain "why", not "what"
- Keep comments up-to-date with code changes
- Document public APIs and complex functions
- Use docstrings for functions and classes

## Error Handling
- Handle exceptions at the appropriate level
- Never swallow exceptions without proper handling
- Use specific exception types
- Provide meaningful error messages
- Fail fast and explicitly

## Code Organization
- Keep related functionality together
- Separate concerns into different modules
- Use appropriate design patterns
- Follow the principle of least surprise
- Structure code for testability

## Testing
- Write tests for all new features and bug fixes
- Aim for high test coverage
- Make tests readable and maintainable
- Test edge cases and error conditions
- Use automated testing

## Refactoring
- Refactor regularly to improve code quality
- Pay down technical debt incrementally
- Make small, incremental changes
- Maintain behavior while improving design
- Use automated tests to verify refactoring

## Version Control
- Write clear, descriptive commit messages
- Make small, focused commits
- Use branches for new features and bug fixes
- Review code before merging
- Keep the main branch stable

## Security
- Validate all inputs
- Never store sensitive information in code
- Use proper authentication and authorization
- Follow the principle of least privilege
- Keep dependencies up to date

---
> Source: [sergebulaev/telegram-channel-saver](https://github.com/sergebulaev/telegram-channel-saver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
