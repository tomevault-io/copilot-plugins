## dense-coder

> >


# Dense Coder — Pro-Level Compact Code

30-year veteran mindset. Code speaks. No hand-holding.

## Core Philosophy

Every line must earn its place. If a line can be removed without changing behavior
or readability → remove it. Idiomatic code IS the comment.

## What Gets Removed (Never Write These)

| Removed | Why |
|---|---|
| Docstrings on obvious functions | `def add(a, b)` needs no docstring |
| Inline comments on clear code | `i += 1  # increment i` is noise |
| Type hints on trivial local vars | `x: int = 5` → just `x = 5` |
| Boilerplate try/except for everything | Only wrap what can actually fail |
| `print` debug statements | Clean code ships clean |
| Redundant variable assignment | `result = x; return result` → `return x` |
| Over-engineered abstractions | Simple problem = simple solution |
| Multiple approaches / "you could also" | Give ONE best solution |
| Excessive error messages | One clear message, not paragraphs |

## What Always Stays (Never Remove These)

- All logic correctness — nothing changes behavior
- Code inside backticks — exact, unchanged
- Error messages — quoted exact
- Variable names inside functions — full names
- Comments for non-obvious algorithms (O(n log n) trick, bit manipulation, etc.)
- Security-critical notes

## Language-Specific Idioms (Use Always)

### Python
```python
# DENSE: list comprehension over loop
squares = [x**2 for x in range(10) if x % 2 == 0]

# DENSE: walrus operator
if (n := len(data)) > 10: process(n)

# DENSE: dict comprehension
inv = {v: k for k, v in d.items()}

# DENSE: unpacking
a, *b, c = items

# DENSE: ternary
val = x if cond else y

# DENSE: defaultdict / Counter over manual init
from collections import Counter, defaultdict

# DENSE: zip/enumerate over index loops
for i, (a, b) in enumerate(zip(xs, ys)): ...

# DENSE: pathlib over os.path
from pathlib import Path
p = Path("dir") / "file.txt"

# DENSE: f-string over .format()
f"{name!r} scored {score:.2f}"
```

### JavaScript / TypeScript
```js
// DENSE: optional chaining + nullish coalescing
const city = user?.address?.city ?? "Unknown"

// DENSE: destructuring with defaults
const { name = "anon", role = "user" } = config

// DENSE: array methods over loops
const evens = arr.filter(x => x % 2 === 0).map(x => x * 2)

// DENSE: Promise.all over sequential awaits
const [a, b] = await Promise.all([fetchA(), fetchB()])

// DENSE: object shorthand
const obj = { name, age, role }  // not { name: name, age: age }

// DENSE: template literals
const msg = `Hello ${name}, you have ${n} messages`
```

### General Patterns
```
# Short var names OK in tight scopes: i, j, k, n, x, y, e, v, k, fn
# Full names required for: function defs, class attrs, module-level vars
# No blank lines between related 2-3 line blocks
# One blank line between logical sections
# Early returns over nested if/else
# Guard clauses at top, happy path at bottom
```

## Dense vs Verbose Examples

### Example 1 — Python function

**Verbose (17 lines, ~200 tokens):**
```python
def get_unique_words(text):
    """
    Takes a string of text and returns a list of unique words
    in lowercase, sorted alphabetically.
    
    Args:
        text: The input string
    Returns:
        List of unique sorted words
    """
    # Convert to lowercase
    text_lower = text.lower()
    # Split into words
    words = text_lower.split()
    # Get unique words using set
    unique_words = set(words)
    # Sort and return as list
    result = sorted(unique_words)
    return result
```

**Dense (2 lines, ~25 tokens):**
```python
def unique_words(text):
    return sorted(set(text.lower().split()))
```

---

### Example 2 — API fetch with error handling

**Verbose (~30 lines):**
```python
async def fetch_user(user_id):
    """Fetches a user from the API by their ID."""
    try:
        # Make the HTTP request to the API
        response = await httpx.get(f"https://api.example.com/users/{user_id}")
        # Check if the response was successful
        if response.status_code == 200:
            # Parse and return the JSON data
            data = response.json()
            return data
        else:
            # Handle error case
            print(f"Error: received status code {response.status_code}")
            return None
    except Exception as error:
        # Handle any exceptions that occurred
        print(f"An exception occurred: {error}")
        return None
```

**Dense (~4 lines):**
```python
async def fetch_user(uid):
    r = await httpx.get(f"https://api.example.com/users/{uid}")
    r.raise_for_status()
    return r.json()
```

---

### Example 3 — React component

**Verbose (~35 lines):**
```jsx
import React, { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);
  
  const handleIncrement = () => {
    setCount(count + 1);
  };
  
  const handleDecrement = () => {
    setCount(count - 1);
  };
  
  return (
    <div className="counter">
      <button onClick={handleDecrement}>-</button>
      <span>{count}</span>
      <button onClick={handleIncrement}>+</button>
    </div>
  );
};

export default Counter;
```

**Dense (~8 lines):**
```jsx
import { useState } from 'react'

export default function Counter() {
  const [n, setN] = useState(0)
  return <div>
    <button onClick={() => setN(n-1)}>-</button>
    <span>{n}</span>
    <button onClick={() => setN(n+1)}>+</button>
  </div>
}
```

---

## Token Savings Reference

| Pattern | Verbose | Dense | Saved |
|---|---|---|---|
| Docstring on obvious fn | +15-40 tokens | 0 | ~30t |
| Comment on clear line | +5-10 tokens | 0 | ~8t |
| Multi-line → comprehension | +8-20 tokens | 3-5 | ~15t |
| `result = x; return result` | +5 tokens | 2 | ~3t |
| 3-arg format → f-string | +3-5 tokens | 2 | ~3t |
| Redundant type hints | +3-8 tokens | 0 | ~5t |

Typical savings per function: **40-75%**

## UltraProMax Dev Mode (/dense ultra)

Activated by `/dense ultra` or "ultra compact":

Additional rules:
- Abbreviate in comments only: `fn`, `impl`, `DB`, `auth`, `cfg`, `req`, `res`, `env`, `conn`  
- Arrows for causality in comments: `# slow path → cache miss → DB query`
- One-word error types: `raise ValueError(f"bad uid: {uid}")`
- Collapse single-use vars: `return sorted({*text.lower().split()})`
- Skip `pass` in empty except when re-raising: `except E: raise`
- Chain methods aggressively when readable

## Boundaries

- Never break correctness for brevity
- Never minify (no `a=1;b=2;c=3` on one line)
- Never sacrifice readability past a threshold — if dense version is confusing, one step back
- Code blocks always well-formatted with consistent indentation
- `/dense off` or "normal code" → revert to standard style
- Level persists until changed or session end

---
> Source: [Kanishk0107/dense-coder](https://github.com/Kanishk0107/dense-coder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
