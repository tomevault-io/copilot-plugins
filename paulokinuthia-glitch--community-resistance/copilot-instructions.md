## community-resistance

> This document provides context and guidelines for AI assistants (like Claude) working with this codebase.

# CLAUDE.md - AI Assistant Guide for Community-Resistance

This document provides context and guidelines for AI assistants (like Claude) working with this codebase.

## Project Overview

**Community-Resistance** is a project focused on community organization, activism tools, and collective action support. The project aims to empower communities through technology.

> **Note**: This project is in its initial stages. This document will be updated as the codebase evolves.

## Repository Structure

```
Community-Resistance/
├── CLAUDE.md           # AI assistant guidelines (this file)
├── README.md           # Project documentation (to be added)
├── src/                # Source code (to be added)
├── tests/              # Test files (to be added)
├── docs/               # Documentation (to be added)
└── .github/            # GitHub workflows and templates (to be added)
```

## Development Workflow

### Getting Started

```bash
# Clone the repository
git clone <repository-url>
cd Community-Resistance

# Install dependencies (when package.json is added)
npm install  # or yarn install, pip install -r requirements.txt, etc.
```

### Branch Naming Conventions

- `main` or `master` - Production-ready code
- `develop` - Integration branch for features
- `feature/<description>` - New features
- `fix/<description>` - Bug fixes
- `docs/<description>` - Documentation updates
- `claude/<session-id>` - AI-assisted development branches

### Commit Message Guidelines

Follow conventional commits format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

## Key Conventions for AI Assistants

### Code Style

1. **Prioritize readability** - Write clean, self-documenting code
2. **Keep it simple** - Avoid over-engineering; solve the immediate problem
3. **Be inclusive** - Use inclusive language in code comments and documentation
4. **Security first** - Never hardcode secrets, validate all inputs
5. **Accessibility** - Ensure UI components are accessible when applicable

### When Making Changes

1. **Understand before changing** - Always read relevant files before modifying
2. **Minimal changes** - Only change what's necessary to complete the task
3. **Test your changes** - Run existing tests and add new ones when appropriate
4. **Document significant changes** - Update relevant documentation

### Community Values

This project serves community organizing efforts. When contributing:

- **Privacy**: Protect user data and community information
- **Security**: Consider the safety implications of features
- **Accessibility**: Make tools usable by everyone
- **Transparency**: Write clear code and documentation
- **Inclusivity**: Use welcoming and inclusive language

## Testing

```bash
# Run tests (when test framework is set up)
npm test          # JavaScript/TypeScript
pytest            # Python
go test ./...     # Go
```

## Common Tasks

### Adding a New Feature

1. Create a feature branch
2. Implement the feature with tests
3. Update documentation if needed
4. Create a pull request

### Fixing a Bug

1. Reproduce the bug
2. Write a failing test
3. Fix the bug
4. Verify the test passes
5. Submit a pull request

### Updating Documentation

1. Keep documentation in sync with code
2. Use clear, concise language
3. Include examples where helpful

## Environment Variables

When environment variables are needed, document them here:

```bash
# Example (update as needed)
# DATABASE_URL=postgresql://localhost:5432/community_resistance
# API_KEY=your_api_key_here
# NODE_ENV=development
```

## Dependencies

Major dependencies will be documented here as they are added.

## API Documentation

API endpoints and their usage will be documented here as they are developed.

## Troubleshooting

Common issues and their solutions will be documented here.

---

## Updating This Document

As the project evolves, update this CLAUDE.md file to reflect:

- New directories and their purposes
- Added dependencies and how to install them
- New scripts and commands
- Changed workflows or conventions
- Important architectural decisions

---

*Last updated: December 2024*
*Repository status: Initial setup - awaiting first commits*

---
> Source: [paulokinuthia-glitch/Community-Resistance](https://github.com/paulokinuthia-glitch/Community-Resistance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
