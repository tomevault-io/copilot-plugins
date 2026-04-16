## magidesk-v1

> This directory contains all development rules, guardrails, and policies for the Magidesk POS project. All files are in `.mdc` format for Cursor AI to understand and enforce.

# Cursor Rules Index

This directory contains all development rules, guardrails, and policies for the Magidesk POS project. All files are in `.mdc` format for Cursor AI to understand and enforce.

## Rules Files

### Architecture & Design
- **architecture.mdc**: Clean Architecture enforcement, layer responsibilities, dependency rules
- **domain-model.mdc**: Domain entity design, relationships, value objects, domain services
- **guardrails.mdc**: Development guardrails that block violations and enforce requirements

### Development Patterns
- **mvvm-pattern.mdc**: MVVM pattern rules, ViewModel guidelines, data binding rules
- **ui-controls.mdc**: WinUI 3 controls, XAML best practices, accessibility requirements
- **code-quality.mdc**: Code standards, complexity limits, refactoring rules

### Policies & Standards
- **development-policies.mdc**: Code review, Git workflow, testing, documentation requirements
- **git-workflow.mdc**: Branch strategy, commit rules, PR process, merge rules
- **security-policies.mdc**: Data protection, authentication, input validation, audit requirements

### Testing & Quality
- **testing-requirements.mdc**: Test coverage requirements, test types, test organization
- **invariants-enforcement.mdc**: How invariants are enforced, testing requirements, exception types

### Infrastructure
- **database-rules.mdc**: PostgreSQL rules, EF Core guidelines, naming conventions, migration rules
- **scripts-control.mdc**: Script organization, PowerShell standards, security rules

### Reference & Documentation
- **floreantpos-reference.mdc**: Rules for using FloreantPOS as reference (read-only, no copying)
- **documentation-control.mdc**: Documentation structure, standards, maintenance requirements

### Tools & Libraries
- **mcp-tools.mdc**: Rules for using MCP tools (Context7, PostgreSQL MCP, Sequential Thinking)
- **library-usage.mdc**: Library selection, usage patterns, EF Core, Dapper, Polly, etc.

### General
- **general-development.mdc**: General development rules, async/await, logging, configuration, MCP tools

## Quick Reference

### Most Important Rules
1. **No business logic in UI** - See `mvvm-pattern.mdc` and `architecture.mdc`
2. **All invariants enforced** - See `invariants-enforcement.mdc`
3. **Clean Architecture** - See `architecture.mdc` and `guardrails.mdc`
4. **FloreantPOS is reference only** - See `floreantpos-reference.mdc`
5. **PostgreSQL in `magidesk` schema** - See `database-rules.mdc`
6. **Use Context7 for library docs** - See `mcp-tools.mdc` and `library-usage.mdc`
7. **Use Sequential Thinking for complex problems** - See `mcp-tools.mdc`

### Required Libraries
- **EF Core**: ORM (Infrastructure only)
- **CommunityToolkit.Mvvm**: MVVM framework
- **FluentValidation**: Validation
- **Polly**: Resilience patterns
- **xUnit/Moq**: Testing
- **Dapper**: Optional, for performance-critical queries

### MCP Tools
- **Context7**: Get latest library documentation (REQUIRED before new features)
- **PostgreSQL MCP**: Database analysis and optimization
- **Sequential Thinking**: Complex problem analysis (REQUIRED for complex domain logic)
- **Memory Bank**: Store architectural decisions and project knowledge (REQUIRED for important decisions)

## Enforcement

- These rules are enforced through:
  - Code review process
  - Automated linting
  - Architecture validation
  - Test requirements
  - CI/CD checks
  - MCP tool usage verification

## Updates

- Update rules when architecture changes
- Document exceptions to rules
- Review rules regularly
- Keep rules current with codebase

## Rule Categories

### Must Follow (Blocking)
- Architecture violations
- Security issues
- Invariant violations
- Code quality below standards
- Skipping library documentation lookup
- Using unmaintained libraries

### Should Follow (Warning)
- Code style
- Documentation
- Test coverage
- Performance
- MCP tool usage

### Best Practices (Guidance)
- Code organization
- Naming conventions
- Error handling patterns
- Logging practices
- Library selection

## How to Use

1. **During Development**: Cursor AI will reference these rules automatically
2. **Code Review**: Reviewers check against these rules
3. **New Team Members**: Read these rules to understand project standards
4. **Architecture Changes**: Update relevant rule files
5. **Library Usage**: Always check Context7 MCP tool first
6. **Complex Problems**: Use Sequential Thinking MCP tool

## Rule File Format

All files use `.mdc` format (Markdown for Cursor) with:
- Clear sections
- Examples where helpful
- Prohibited practices listed
- Required practices listed
- Enforcement mechanisms described

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zedfauji) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
