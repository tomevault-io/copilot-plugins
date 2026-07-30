## telemt-panel

> Guidelines for writing self-explanatory code with minimal comments. Examples in Go and JavaScript.


# Self-explanatory Code Commenting Instructions

## Core Principle
**Write code that speaks for itself. Comment only when necessary to explain WHY, not WHAT.**
We do not need comments most of the time.

## Commenting Guidelines

### AVOID These Comment Types

**Obvious Comments**
```go
// Bad: States the obvious
counter := 0  // Initialize counter to zero
counter++     // Increment counter by one
```

**Redundant Comments**
```go
// Bad: Comment repeats the code
func getUserName(user *User) string {
    return user.Name  // Return the user's name
}
```

**Outdated Comments**
```go
// Bad: Comment doesn't match the code
// Calculate tax at 5% rate
tax := price * 0.08  // Actually 8%
```

### WRITE These Comment Types

**Complex Business Logic**
```go
// Apply progressive tax brackets: 10% up to 10k, 20% above
tax := calculateProgressiveTax(income, []float64{0.10, 0.20}, []float64{10000})
```

**Non-obvious Algorithms**
```go
// Using Floyd-Warshall for all-pairs shortest paths
// because we need distances between all nodes
for k := 0; k < vertices; k++ {
    for i := 0; i < vertices; i++ {
        for j := 0; j < vertices; j++ {
            // ... implementation
        }
    }
}
```

**Regex Patterns**
```go
// Match email format: username@domain.extension
var emailPattern = regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
```

**API Constraints or Gotchas**
```go
// GitHub API rate limit: 5000 requests/hour for authenticated users
rateLimiter.Wait(ctx)
resp, err := http.Get(githubAPIURL)
```

**Concurrency Reasoning**
```go
// mu protects activeConns from concurrent WebSocket handler access
var mu sync.Mutex
```

## Decision Framework

Before writing a comment, ask:
1. **Is the code self-explanatory?** -> No comment needed
2. **Would a better variable/function name eliminate the need?** -> Refactor instead
3. **Does this explain WHY, not WHAT?** -> Good comment
4. **Will this help future maintainers?** -> Good comment

## Special Cases for Comments

### Exported APIs (Go)
```go
// AuthService handles JWT token creation and validation for the panel.
type AuthService struct {
    secret []byte
    expiry time.Duration
}

// NewAuthService creates an AuthService with the given config.
// The JWT secret must be at least 32 bytes for HS256.
func NewAuthService(cfg *config.Config) *AuthService {
    // ...
}
```

### Public APIs (JavaScript/TypeScript)
```javascript
/**
 * Calculate compound interest using the standard formula.
 *
 * @param {number} principal - Initial amount invested
 * @param {number} rate - Annual interest rate (as decimal, e.g., 0.05 for 5%)
 * @param {number} time - Time period in years
 * @param {number} compoundFrequency - How many times per year interest compounds (default: 1)
 * @returns {number} Final amount after compound interest
 */
function calculateCompoundInterest(principal, rate, time, compoundFrequency = 1) {
    // ... implementation
}
```

### Configuration and Constants
```go
const (
    maxRetries = 3     // Based on network reliability studies
    apiTimeout = 5000  // AWS Lambda timeout is 15s, leaving buffer
)
```

### Annotations
```go
// TODO: Replace with proper user authentication after security review
// FIXME: Memory leak in production - investigate connection pooling
// HACK: Workaround for bug in library v2.1.0 - remove after upgrade
// NOTE: This implementation assumes UTC timezone for all calculations
// WARNING: This function modifies the original slice instead of creating a copy
// SECURITY: Validate input to prevent path traversal before using in file access
```

## Anti-Patterns to Avoid

### Dead Code Comments
```go
// Bad: Don't comment out code — use version control
// func oldHandler(w http.ResponseWriter, r *http.Request) { ... }
func newHandler(w http.ResponseWriter, r *http.Request) { ... }
```

### Changelog Comments
```go
// Bad: Don't maintain history in comments — use git log
// Modified by John on 2023-01-15
// Fixed bug reported by Sarah on 2023-02-03
func processData() {
    // ... implementation
}
```

### Divider Comments
```go
// Bad: Don't use decorative comments — use package/file structure instead
//=====================================
// UTILITY FUNCTIONS
//=====================================
```

## Quality Checklist

Before committing, ensure your comments:
- [ ] Explain WHY, not WHAT
- [ ] Are grammatically correct and clear
- [ ] Will remain accurate as code evolves
- [ ] Add genuine value to code understanding
- [ ] Are placed above the code they describe
- [ ] Use proper spelling and professional language

## Summary

**The best comment is the one you don't need to write because the code is self-documenting.**

---
> Source: [amirotin/telemt_panel](https://github.com/amirotin/telemt_panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
