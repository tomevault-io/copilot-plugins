## testing

> - ✅ ALWAYS use `go run .` instead of building the binary for testing

# Testing Rules

## Use `go run .` for Testing
- ✅ ALWAYS use `go run .` instead of building the binary for testing
- ✅ This is faster and avoids unnecessary binary generation
- ❌ Don't use `go build -o agentuity` unless you need a persistent binary

## Testing Workflow
```bash
# ✅ Good: Test with go run
go run . bundle
go run . dev
go run . project create

# ❌ Avoid: Building every time for testing
go build -o agentuity && ./agentuity bundle
```

## When to Build
- Only build when you need a persistent binary for:
  - Installation scripts
  - Distribution
  - CI/CD pipelines
  - Manual testing outside the project directory

## Development Benefits
- Faster iteration cycle
- No need to manage binary files
- Cleaner development workflow
- Less disk space usage

---
> Source: [agentuity/cli](https://github.com/agentuity/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
