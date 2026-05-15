## learn-funx

> This file provides Claude Code specific guidance when working with this repository.

# CLAUDE.md

This file provides Claude Code specific guidance when working with this repository.

## Project Overview

**This is a functional programming playground** using the [Funx](https://github.com/JKWA/funx) library. See `usage-rules.md` for complete context and functional programming guidance.

## Critical for Claude

**ALWAYS READ**: `usage-rules.md` contains essential functional programming patterns and LLM guidance for this project. That file explains:
- The two layers of usage rules (local + Funx library)
- Domain-first thinking before code
- How to stay functional and avoid imperative patterns
- Project structure and learning path
- Working with users on functional programming

This file (CLAUDE.md) focuses on Claude-specific technical details.

## Development Commands

### Interactive Learning (Primary Use Case)
- `iex -S mix` - Primary interface for experimenting with functional patterns
- Copy commands from `example.txt` to explore the domain model

### Standard Development
- `mix compile` - Compile the project
- `mix test` - Run all tests  
- `mix format` - Format code
- `mix deps.get` - Fetch dependencies

## Key Dependencies

### Funx Library (`~> 0.1.5`)
Functional programming library designed specifically for learning:
- Monads: Maybe, Either, Effect, Reader, Writer, Identity
- Comprehensive validation and error handling patterns
- Predicate composition and logical operations
- Protocol-based equality and ordering
- **Extensive usage rules** designed for LLM guidance

## Claude's Strengths for This Project

You excel at:
- **Reading and applying usage rules** - use them to guide all decisions
- **Functional composition** - building proper monadic chains
- **Domain modeling discussions** - helping think through business rules before code
- **Pattern recognition** - seeing how to apply functional templates to new problems
- **Incremental development** - making small, focused changes that build understanding
- **Multi-file analysis** - understanding how domain models connect to repositories

## Claude-Specific Capabilities

- **Batch tool calls** - when reading multiple files or running parallel commands, use multiple tool calls in a single message for efficiency
- **File reading context** - you can read multiple files simultaneously to understand patterns across the codebase
- **Error pattern recognition** - you're particularly good at spotting when code drifts toward imperative patterns

## Teaching Approach

This project follows **discovery over instruction**:
- **Context over examples** - complete domain model, not isolated snippets  
- **Patterns in action** - functional concepts solve real domain problems
- **Learn by breaking** - encourage users to modify code and see what happens
- **Working foundation** - solid base to extend and explore

Focus on **learning through curiosity-driven experimentation**.

## Technical Notes

- Code formatting includes all `.ex`/`.exs` files in standard locations
- Tests use ExUnit with comprehensive examples showing functional patterns
- **Educational project** - not intended for production use
- Local Funx dependency provides immediate access to usage rules
- ETS storage for simple persistence without external dependencies

## Workflow with Users

1. **Always reference usage-rules.md** - both you and the user should understand the functional programming context
2. **Encourage domain discussions** before implementation - "What are the business rules here?"
3. **Point to existing patterns** - Hero.ex demonstrates the complete template
4. **Suggest small experiments** - "Try modifying the validation and see what happens"
5. **Course correct gently** - if you see imperative patterns, remind about the usage rules

---

**Remember**: Your role is to help users discover functional programming patterns through guided experimentation. Always consult `usage-rules.md` for the complete functional programming context and guidance.

---
> Source: [JKWA/learn_funx](https://github.com/JKWA/learn_funx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
