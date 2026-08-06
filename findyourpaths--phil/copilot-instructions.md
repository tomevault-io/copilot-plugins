## phil

> **Last Updated**: 2025-11-17

# Phil Project Context

**Last Updated**: 2025-11-17
**Project Type**: Go library
**Full Name**: PHIL - Parser for Human-generated Interval Language

---

## Branching

**Commit straight to `main`. Do not create feature branches in this repo.**

The paths project consumes this library via:
- **Local**: paths uses a Go workspace (`/Users/wag/Dropbox/Projects/go_<worktree>.work`) listing `./phil` — local paths builds use your sibling worktree. goskyr does the same.
- **CI**: paths' `go.mod` (and goskyr's `go.mod` independently) pins a pseudo-version (`v0.0.0-<ts>-<sha>`). `go mod download` resolves it from `origin/main` via `proxy.golang.org` (or direct, with `GOPRIVATE=github.com/findyourpaths/*`). If the pinned SHA is not reachable on `origin/main`, the build fails with `unknown revision`.

When a paths or goskyr change depends on a phil change: push phil to `origin/main` FIRST. Then in the consuming repo, run `GOPRIVATE=github.com/findyourpaths/* go get github.com/findyourpaths/phil@main && go mod tidy` (paths has `make bump-libs` as a shortcut), commit `go.mod`/`go.sum`, and push. `/gitsh` in paths enforces this order.

For exploratory work, use a local branch as scratch space, then rebase to `main` before the dependent paths/goskyr change ships. See `../docs/WORKFLOW.md` §9 "Cross-Repo Discipline".

---

## Project Overview

**Phil** is a natural language date/time parser that converts human-readable date expressions into structured datetime objects.

**Use cases**:
- Parse event dates from emails ("Next Tuesday", "March 15-17")
- Extract date ranges from natural text
- Handle complex date patterns (ISO 8601, RFC 3339, casual language)
- Validate and normalize date/time inputs

**Examples**:
```
Input: "Next Tuesday"           → 2025-11-19 (next Tuesday from today)
Input: "March 15-17"            → 2025-03-15 to 2025-03-17
Input: "2023-02"                → 2023-02-01 (year-month format)
Input: "10am to 2pm"            → 10:00:00 to 14:00:00
Input: "Friday, March 15, 2024" → 2024-03-15
```

---

## Architecture

### Parser Structure

**Technology**: GLR (Generalized LR) parser with yacc grammar

**Components**:
1. **Lexer** (`parse_lex.go`) - Tokenizes input string
2. **Grammar** (`parse_yacc.y`) - Defines date/time syntax rules
3. **Parser** (`parse_glr.go`, `parse_yacc.go`) - Generated from grammar
4. **DateTime types** (`datetime.go`) - Structured date/time representations

### Parser Change Policy

`parse_yacc.y` is the source of truth for datetime syntax. When a parse failure
or wrong parse is caused by missing syntax, update the grammar first and
regenerate the parser. Do not add preprocessing regular expressions in
`parse_lex.go` to work around grammar gaps unless the grammar-first attempt has
been documented in the change and the remaining problem is demonstrably lexical
normalization (for example HTML artifacts, token boundaries, or punctuation
classification). New syntax support should normally include:

1. A failing parser test that names the real input shape.
2. A `parse_yacc.y` production or token classification change.
3. Regenerated parser artifacts from `make generate`.
4. A passing targeted test.

### Key Types

```go
type DateTime struct {
    Date  *Date      // Year, month, day
    Time  *Time      // Hour, minute, second
    Range *Range     // Start and end for ranges
}

type Date struct {
    Year  int
    Month int
    Day   int
}

type Time struct {
    Hour   int
    Minute int
    Second int
}
```

---

## Integration with Paths

The paths project uses phil to parse event dates from emails and scraped web pages.

**Usage**:
```go
import "github.com/findyourpaths/phil/datetime"

// Parse a date string
dt, err := datetime.Parse("Next Tuesday")
if err != nil {
    // Handle parse error
}

// Use parsed date
startTime := dt.StartTime()
endTime := dt.EndTime()
```

**Key integration points**:
- `internal/eventrecord/` - Parse event dates from LLM responses
- `internal/event/` - Normalize event start/end times

---

## Grammar Features

### Supported Formats

**ISO 8601 & RFC 3339**:
- `2023-02-15` (date)
- `2023-02` (year-month)
- `14:30:00` (time with seconds)
- `14:30` (time without seconds)
- `2023-02-15T14:30:00Z` (full timestamp)

**Natural Language**:
- `Next Tuesday`
- `Tomorrow at 3pm`
- `March 15`
- `Friday, March 15, 2024`

**Ranges**:
- `March 15-17` (date range)
- `10am to 2pm` (time range)
- `March 15-17, 2024` (date range with year)

**Lists**:
- `March 15, 16, 17` (comma-separated days)
- `Monday, Tuesday, Friday` (weekday list)

---

## Technology Stack

**Language**: Go 1.22+
**Parser**: GLR (Generalized LR) with yacc
**Build system**: Bazel (optional)
**Testing**: Go testing framework

---

## Development Environment

### Building

**Standard Go build**:
```bash
cd datetime
go generate  # Regenerate parser from .y file
go build
```

**Bazel build** (optional):
```bash
bazel build //datetime:datetime
```

### Testing

**Run all tests**:
```bash
cd datetime
go test -v
```

**Test statistics**:
- Total tests: 225
- Passing: ~197 (87.56%)
- Failing: ~28 (12.44%)

**Common failure patterns**:
- Complex natural language patterns
- Timezone handling
- Semantic validation (e.g., invalid ranges)

---

## Documentation

**Issues**: [ISSUES.md](datetime/ISSUES.md) - Known issues and fixes
**Source**: [datetime.go](datetime/datetime.go), parse_yacc.y

---

## Code Generation

The parser is generated from yacc grammar:

**Grammar file**: `datetime/parse_yacc.y`
**Generated files**:
- `parse_yacc.go` - Parser code
- `parse_glr.go` - GLR parser implementation
- `parse_yacc.states.txt` - Parser states (debug)

**Regenerate parser**:
```bash
cd datetime
go generate
```

**Grammar changes**:
1. Edit `parse_yacc.y`
2. Run `go generate`
3. Test with `go test`
4. Commit both .y and generated .go files

---

## Common Commands

**Parse a date string** (interactive):
```bash
cd cmd
go run main.go "Next Tuesday"
```

**Run tests**:
```bash
cd datetime
go test -v
```

**Debug parser**:
```bash
cd datetime
go test -v -run TestParse/001  # Run specific test
```

---

## Known Issues

See [ISSUES.md](datetime/ISSUES.md) for detailed list.

**Major categories**:
1. ISO 8601 format edge cases
2. Semantic validation (invalid range interpretations)
3. Complex natural language patterns
4. Timezone and context-dependent parsing

**Test coverage**: ~87.56% passing (197/225 tests)

---

## Related Projects

**paths**: Primary consumer of phil (event date parsing)

---

## Quick Start for Development

```bash
# Navigate to phil
cd /Users/wag/Dropbox/Projects/phil/datetime

# Run tests
go test -v

# Edit grammar (if needed)
vim parse_yacc.y

# Regenerate parser
go generate

# Test changes
go test -v
```

---

## Related Documentation

**Shared standards** (parent level):
- [go_style.md](../docs/go/go_style.md) - Go coding conventions
- [WORKFLOW.md](../docs/WORKFLOW.md) - LLM collaboration patterns
- [go_testing.md](../docs/go/go_testing.md) - Testing patterns

**Project-specific**:
- [ISSUES.md](datetime/ISSUES.md) - Known issues and fixes
- [parse_yacc.y](datetime/parse_yacc.y) - Grammar definition

---
> Source: [findyourpaths/phil](https://github.com/findyourpaths/phil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
