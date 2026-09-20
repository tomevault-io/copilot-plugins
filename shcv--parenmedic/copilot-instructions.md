## parenmedic

> Parenmedic is a Zig command-line tool that diagnoses and fixes parenthesis issues in Lisp code using indentation as a guide.

# CLAUDE.md

## Project Overview

Parenmedic is a Zig command-line tool that diagnoses and fixes parenthesis issues in Lisp code using indentation as a guide. 

## Development Environment

**Language**: Zig 0.14+  
**Build System**: Zig build system  
**Binary Output**: Single static executable with no runtime dependencies

### Building
```bash
# Build the project
zig build

# Run the binary
./zig-out/bin/parenmedic --help
```

### Running Tests
```bash
# Run unit tests
zig build test
```

## Usage Examples

### Basic Usage
```bash
# Diagnose parenthesis issues
./zig-out/bin/parenmedic diagnose file.lisp

# Fix issues (output to stdout)
./zig-out/bin/parenmedic fix file.lisp

# Fix in-place
./zig-out/bin/parenmedic fix --in-place file.lisp
./zig-out/bin/parenmedic fix -i file.lisp     # short form

# Preview changes with diff
./zig-out/bin/parenmedic fix --diff file.lisp
./zig-out/bin/parenmedic fix -D file.lisp      # short form

# Check mode (exit code indicates issues)
./zig-out/bin/parenmedic fix --check file.lisp
./zig-out/bin/parenmedic fix -c file.lisp      # short form

# Different output formats
./zig-out/bin/parenmedic diagnose --format=json file.lisp
./zig-out/bin/parenmedic diagnose -f gcc file.lisp

# Stdin/stdout support
echo '(defn test [x]' | ./zig-out/bin/parenmedic diagnose -
cat file.lisp | ./zig-out/bin/parenmedic fix

# Directory processing (recursive)
./zig-out/bin/parenmedic diagnose src/

# Multiple files and glob patterns
./zig-out/bin/parenmedic fix -i *.clj
./zig-out/bin/parenmedic fix -c src/*.lisp test/*.lisp

# Force dialect
./zig-out/bin/parenmedic diagnose --dialect=clojure file.txt
./zig-out/bin/parenmedic diagnose -d elisp file.el
```

## Architecture Overview

### Core Processing Pipeline
1. **Parser** (`src/parser.zig`): Tokenizes input and tracks both indentation and parenthesis depth per line
2. **Analyzer** (`src/analyzer.zig`): Detects discrepancies between indentation and parenthesis depth
3. **Fixer** (`src/fixer.zig`): Generates fixes for mechanical issues (missing/extra/mismatched parentheses)
4. **Formatter** (`src/formatter.zig`): Outputs results in various formats (rich/simple/gcc/json/lsp)

### Key Modules
- **Main** (`src/main.zig`): Command-line interface and argument parsing
- **Config** (`src/config.zig`): Dialect-specific configuration (comment styles, string literals, etc.)
- **MCP** (`src/mcp.zig`): Model Context Protocol server for LLM integration

### Data Structures
- **ParseResult**: Contains line info and tokens from parsed code
- **Issue**: Represents detected problems with type, location, and error details
- **Repair**: Represents fixes with action type (insert/delete/replace)

## Key Features

### Fix Capabilities
- Fixes mechanical issues: missing parentheses, extra parentheses, and mismatched delimiters
- Preserves original code structure and indentation
- Operates based on indentation analysis to determine correct placement
- Intelligently places closing delimiters before end-of-line comments
- Handles dialect-specific features (character literals, reader conditionals, etc.)

### Output Formats
- **Rich**: Context-rich with surrounding code and colored diff (default for terminal)
- **Simple**: One line per issue (auto-selected when piping output)
- **GCC**: Editor-compatible format
- **JSON**: Machine-readable for tooling
- **LSP**: Language Server Protocol format

### Issue Types
- `missing_close`: Missing closing parentheses
- `extra_close`: Extra closing parentheses  
- `mismatched_types`: Wrong bracket/brace/paren type

## Testing

The project includes unit tests in each module. Run with:
```bash
zig build test
```

Example test files in `/examples/` directory contain various parenthesis issues for testing the tool.

---
> Source: [shcv/parenmedic](https://github.com/shcv/parenmedic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
