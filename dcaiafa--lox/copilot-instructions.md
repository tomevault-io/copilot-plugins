## lox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Lox is a lexer and parser generator for Go. It takes `.lox` grammar files and generates Go code for lexers and parsers.

## Commands

### Building and Installation
```bash
# Build the lox command
go build ./cmd/lox

# Install globally
go install ./cmd/lox
```

### Using Lox
```bash
# Generate parser/lexer for current directory
lox .

# Generate with detailed report
lox --report ./my-parser
```

### Testing
```bash
# Run all tests
go test ./...

# Run specific test
go test -run TestName

# Update baseline tests
go test -run TestName -args -update-baseline
```

### Regenerating Lox's Own Parser
```bash
cd internal/parser
lox .
```

## Architecture

### Code Generation Flow
1. `.lox` files define grammar using Lox's DSL
2. `lox` command parses these files and generates:
   - `base.gen.go` - Base types and interfaces
   - `lexer.gen.go` - Generated lexer
   - `parser.gen.go` - Generated parser tables and logic
3. Developers implement action methods in their own Go files

### Key Components
- **internal/parser/** - Lox's self-hosted parser (defined in parser.lox)
- **internal/codegen/** - Code generation logic using Jet templates
- **internal/lexergen/** - DFA/NFA-based lexer generation
- **internal/parsergen/** - LR(1) parser generation
- **internal/ast/** - AST definitions for the Lox grammar language

### Grammar File Structure
```
@lexer
// Token definitions
NUM = [0-9]+
ADD = '+'

@parser
// Parser rules
@start expr
expr = expr '+' expr  @left(1)
     | NUM
```

### Testing Approach
- Unit tests use standard Go testing
- Integration tests in internal/tests/ use YAML files to define test cases
- Baseline testing compares output against stored snapshots in _baseline/ directories

## Important Notes
- Lox is self-hosted: it uses its own parser generator to parse .lox files
- Generated files (*.gen.go) should not be edited manually
- When modifying Lox's grammar, regenerate the parser by running `lox .` in internal/parser/

---
> Source: [dcaiafa/lox](https://github.com/dcaiafa/lox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
