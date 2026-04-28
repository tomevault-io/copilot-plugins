## logo-lsp

> LSP server for the LOGO programming language, written in Kotlin. Test assignment.

# LOGO-LSP

LSP server for the LOGO programming language, written in Kotlin. Test assignment.

## What it does

Implements the Language Server Protocol for LOGO — an educational language with turtle graphics. The server provides:
1. **Semantic tokens** — syntax highlighting (keywords, commands, variables, numbers, comments)
2. **Go-to-declaration** — navigate to procedure and variable definitions
3. **Diagnostics** — parse error reporting
4. **Completion** — context-aware autocompletion for commands and variables

## Architecture

```
Source text → Lexer → Tokens → Parser → AST (with spans) → LSP features
```

- **Full document sync** — LOGO files are small; re-parse everything on each edit
- **Two-pass parsing** — Pass 1: scan procedure arities; Pass 2: full parse (needed because LOGO parsing depends on arity)
- **Case-insensitive** — all lookups use uppercase, original casing preserved for display
- **AST spans** — every node tracks start/end position for precise cursor-to-node mapping
- **AST traversal** — `Node.walk()` (depth-first), `Node.findNodePath(line, col)` (cursor → path from root)

## Project structure

```
app/src/main/kotlin/logo/
├── Main.kt                      # LSP stdio launcher
├── lexer/
│   ├── Token.kt                 # Token data class + TokenType enum (42+ types)
│   └── Lexer.kt                 # Single-pass character scanner
├── parser/
│   ├── Ast.kt                   # Sealed AST hierarchy (20+ node types) + walk/findNodePath
│   └── Parser.kt                # Two-pass recursive descent parser
├── analysis/
│   └── BuiltinCommands.kt       # ~60 built-in commands with arities
├── lsp/
│   ├── LogoLanguageServer.kt    # LSP entry point, declares capabilities
│   ├── LogoTextDocumentService.kt # Handles document events + feature dispatch
│   ├── LogoWorkspaceService.kt  # Minimal no-op
│   └── DocumentManager.kt       # Document state + reparse pipeline
└── features/
    ├── SemanticTokensProvider.kt # Lexer tokens → delta-encoded LSP semantic tokens
    ├── GoToDeclarationProvider.kt # Variable/procedure → definition location
    ├── DiagnosticsProvider.kt    # ParseError → LSP Diagnostic
    └── CompletionProvider.kt     # Context-aware completion (scoped variables, builtins, keywords)
```

## Build & run

```bash
./gradlew test          # run tests
./gradlew shadowJar     # fat JAR → app/build/libs/logo-lsp.jar
java -jar app/build/libs/logo-lsp.jar   # start server (stdio)
```

## Tech stack

- Kotlin 2.1.20, JVM 17+
- LSP4J 0.24.0
- Gradle + Shadow plugin 8.1.1
- JUnit 5

## Implementation status

- [x] Project skeleton & build system
- [x] Lexer (Token.kt, Lexer.kt) + tests (30 cases)
- [x] Built-in Commands Registry (~60 commands)
- [x] Parser (AST + two-pass recursive descent) + tests (49 cases)
- [x] LSP Core (DocumentManager, Server, Services)
- [x] Diagnostics (parse errors → LSP diagnostics)
- [x] Completion (scoped variables, builtins, keywords) + tests (12 cases)
- [x] Go-to-declaration (variables + procedures) + tests (15 cases)
- [x] Semantic tokens (syntax highlighting) + tests (12 cases)
- [x] Main.kt LSP launcher wiring
- [x] README.md

## Parser design decisions

- **Two-pass parsing**: Pass 1 scans `TO...END` blocks to learn procedure arities before parsing. This allows forward references (calling a procedure before its definition).
- **Tight vs full expressions**: Reporter arguments in expression context use "tight" parsing (primaries/unary only, no infix operators). Statement-level command arguments use full expression parsing with operators. Example: `PRINT SUM 3 4 + 5` → `PRINT((SUM 3 4) + 5)`. This matches standard UCBLogo behavior.
- **WHILE**: Both `WHILE [cond] [body]` and `WHILE cond [body]` are supported — disambiguated by whether the first token after WHILE is `[`.
- **MAKE/LOCALMAKE**: Parsed as `VariableAssignment` when followed by a quoted word (`MAKE "x 5`), otherwise as a generic `CommandCall`.
- **Unknown procedures**: Default to arity 0 at parse time; the analyzer will flag them.
- **Error recovery**: Errors are accumulated in a list (never thrown). Boundary tokens (NEWLINE, EOF, `]`, `)`, END) are never consumed by expression parsing, preventing cascade errors. Critical failures (e.g., missing procedure name after TO) trigger `recover()` which skips to the nearest NEWLINE/END/`]`/EOF. Minor failures (unexpected token at statement level) advance one token and continue.

## LOGO language specifics

- Procedures: `TO name :param1 :param2 ... END`
- Variables: `:varname` (reference), `MAKE "varname value` (assignment)
- Case-insensitive keywords
- Control structures: REPEAT, IF, IFELSE, WHILE, FOR, FOREACH, FOREVER
- Comments: `;` to end of line
- Quoted words: `"word` (string literals with no closing quote)
- `LOCAL "x` / `LOCALMAKE "x` — explicit local variable declaration; without it, `MAKE` creates globals

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Aramarchuk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
