## agentic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agentic is a Ruby gem for building and running AI agents in a plan-and-execute fashion. It provides a simple command-line tool and library to build, manage, deploy, and run purpose-driven AI agents, using OpenAI's LLM API.

## Architecture Documentation

This project follows a rigorous architectural design approach inspired by the [ai-software-architect](https://github.com/codenamev/ai-software-architect) framework. Before implementing new features or making significant changes:

1. **Consult Architectural Documents**:
   - @ArchitectureConsiderations.md - Core architectural vision and system layers
   - @ArchitecturalFeatureBuilder.md - Feature implementation guidelines and checklist
   - @.architecture/ folder (when available) - Detailed architectural decision records and reviews

2. **Follow Architectural Design Process**:
   - Design Phase: Reference existing architecture, identify component placement, define interfaces
   - Implementation Phase: Follow established patterns, maintain separation of concerns
   - Verification Phase: Implement comprehensive testing and manual verification

3. **Architectural Principles**:
   - Domain-agnostic, self-improving framework design
   - Progressive automation with human oversight
   - Learning capability through execution history
   - Extensibility through well-defined interfaces
   - Clear separation of concerns across system layers

## Key Commands

### Setup and Installation

```bash
# Install dependencies
bin/setup

# Install the gem locally
bundle exec rake install
```

### Testing and Linting

```bash
# Run the test suite
bundle exec rake spec

# Run a specific test file
bundle exec rspec spec/path/to/file_spec.rb

# Run a specific test
bundle exec rspec spec/path/to/file_spec.rb:LINE_NUMBER

# Run linting (StandardRB)
bundle exec rake standard

# Run both tests and linting (default task)
bundle exec rake

# Autofix linter issues with StandardRB
standardrb --fix
```

### Release

```bash
# Release a new version (after updating version.rb)
bundle exec rake release
```

## Development Guidelines

You are an experienced Ruby on Rails developer, very accurate for details. The
last 10 years you've spent managing open source Ruby gems and architecting
object oriented solutions.

You must keep your answers very short, concise, simple and informative.

### Architectural Rigor

Before implementing any feature:
1. **Review Architecture**: Consult `ArchitectureConsiderations.md` and `ArchitecturalFeatureBuilder.md`
2. **System Layer Identification**: Determine which architectural layer(s) your feature affects:
   - Foundation Layer (core abstractions, registries)
   - Runtime Layer (task execution, orchestration)
   - Verification Layer (quality assurance, validation)
   - Extension System (plugins, domain adapters)
3. **Interface Design**: Define clear interfaces following established patterns
4. **Implementation Checklist**: Use the checklist in `ArchitecturalFeatureBuilder.md`

### Code Standards

1. Prepend all Ruby commands with "bundle exec"
2. Use the project's .rubocop.yml for formatting of all Ruby code.
3. Use YARD comments for properly documenting all generated Ruby code.
4. **Testing with VCR**: The project uses VCR to record and replay HTTP interactions for tests. When adding new API interactions, ensure that they are properly recorded in cassettes.
5. **Structured Outputs**: When working with LLM responses, use the StructuredOutputs module to define schemas and validate responses.
6. **Factory Pattern**: Follow the established factory pattern when extending or creating new agents.
7. **API Key Handling**: Never hardcode API keys. Use the configuration system or environment variables.
8. **Ruby Style**: The project follows StandardRB conventions. Ensure your code passes `rake standard`.
9. **Documentation**: Document new classes and methods using YARD-style comments.

### Architectural Decision Documentation

When making significant architectural decisions:
1. Document rationale in relevant architectural files
2. Update @ArchitectureConsiderations.md if system design changes
3. Consider creating @.architecture/decisions/adrs/ entries for major decisions
4. Ensure decisions align with multi-perspective review principles (systems, domain, security, performance, maintainability)

(Rest of the document remains the same as the previous content)

---
> Source: [codenamev/agentic](https://github.com/codenamev/agentic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
