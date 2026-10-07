## bilirec

> Place unexported (lowercase) functions and methods at the bottom of Go files


# Private Symbols at the Bottom

In each `.go` file, **exported** names (types, functions, methods with capitalized identifiers) come first. **Unexported** helpers (`func foo`, `func (s *T) bar` with lowercase names) belong at the **bottom** of the file, after all exported API.

## Order (top → bottom)

1. Package comment, `package`, imports
2. Package-level vars / constants (if any)
3. Exported types and their **exported** methods
4. Exported functions
5. Unexported block (all below exported API), in this order:
   - Unexported types (if any)
   - Unexported **methods** (`func (t *T) foo`)
   - Package-level unexported **helper functions** (`func foo()` with no receiver)

Within the private block, methods stay above standalone helpers—even when everything is unexported.

```go
// ✅ exported API first
func DoWork() error { return doWorkImpl() }

type Client struct{}

func (c *Client) Run() error { return c.run() }

// --- private: methods, then package helpers ---

func (c *Client) run() error { ... }

func doWorkImpl() error { ... }

func parseThing(s string) (Thing, error) { ... }
```

When editing a file, move new private helpers to the bottom of the private block (below private methods), not above exported symbols or above other types’ methods.

---
> Source: [bilirec/bilirec](https://github.com/bilirec/bilirec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
