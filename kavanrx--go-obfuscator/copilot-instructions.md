## go-obfuscator

> A zero-dependency Go library for data obfuscation. Module: `go-obfuscator`. Target: Go 1.24+.

# go-obfuscator Agent Instructions

## Project Overview

A zero-dependency Go library for data obfuscation. Module: `go-obfuscator`. Target: Go 1.24+.

## Key Commands

```bash
# Build
go build ./...

# Test all packages
go test ./...

# Test with race detection
go test ./... -race

# Run benchmarks
go test ./benchmark -bench=. -benchtime=1s

# Test single package
go test ./obfuscator -v
```

## Architecture Notes

- **Core interfaces** (`Matcher`, `Transformer`) are defined in `internal/types/types.go`
- `matcher/` and `transformer/` re-export interfaces as type aliases for convenience
- Engine is **immutable after construction** - safe for concurrent use
- **Each cloned engine gets its own cache** - don't share `*TypeCache` between engines
- Type metadata is **cached by reflect.Type** - rule matching happens at runtime, not cache time

## Important Implementation Details

### Clone() Behavior
- Creates new `walker.Walker` and `cache.TypeCache` instances
- Copies config rules, ignore fields, and hash secret
- Deep copies HashSecret only if non-empty (safe for nil)

### Engine Design
- `Obfuscate()` creates **new struct/map/slice** - does NOT mutate original
- `CloneAndObfuscate()` does extra deep copy - only needed if original must stay untouched
- Reflection cache stores field metadata (name, path, tags), not rule indices
- Rule matching is done **at runtime** in `applyMatchers()`

### Matcher/Transformer Types
- `types.Rule` contains `Matcher` and `Transformer` interfaces
- Implement custom matchers: `matcher.MatcherFunc(fn)` for function-based matching
- Implement custom transformers: `transformer.TransformerFunc(fn)` or `TransformerFunc(fn) any`

## Struct Tag Support

```go
type User struct {
    Email string `mask:"email"`      // EmailMask()
    Phone string `mask:"phone"`      // PhoneMask()
    PIN   string `mask:"partial"`    // PartialMask(2, 4)
    SSN   string `mask:"id"`         // IDNumberMask()
}
```

## Default PII Rules

`obfuscator.DefaultEngine()` includes built-in rules for: email, phone, nik, ktp, passport, birth_date, address, pin, otp, password, token, jwt, api_key, client_secret, etc.

## Common Patterns

```go
// Field matching
engine := obfuscator.New(
    obfuscator.WithField("email", transformer.EmailMask()),
)

// Regex matching
engine := obfuscator.New(
    obfuscator.WithRegex(regexp.MustCompile(`.*password.*`), transformer.Mask()),
)

// Composite matcher (OR)
engine := obfuscator.New(
    obfuscator.WithCompositeMatcher(matcher.MatchAny,
        transformer.Mask(),
        matcher.NewField("email"),
        matcher.NewField("phone"),
    ),
)
```

## Testing Conventions

- Table-driven tests preferred
- Tests in `*_test.go` files alongside source
- Benchmark tests in `benchmark/benchmark_test.go`
- Run `go vet ./...` before committing

## Performance Targets

| Operation | Target |
|-----------|--------|
| SmallStruct (5 fields) | <2µs/op |
| LargeStruct (50 fields) | <50µs/op |
| Map (100 entries) | <30µs/op |
| CloneAndObfuscate | <2µs/op |

Benchmarks run on M2: expect ~2x slower on Intel.

---
> Source: [kavanrx/go-obfuscator](https://github.com/kavanrx/go-obfuscator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
