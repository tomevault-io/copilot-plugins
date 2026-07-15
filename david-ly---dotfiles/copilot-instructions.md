## dotfiles

> - Use `snake_case` for variable names

# Cursor Rules

## Naming Conventions

### Variables
- Use `snake_case` for variable names
- Examples: `user_name`, `max_count`, `is_valid`

### Functions and Methods
- Use `camelCase` for function and method names
- Examples: `getUserData()`, `calculateTotal()`, `isValidInput()`

### Classes, Interfaces, and Types
- Use `PascalCase` for classes, interfaces, and type definitions
- Examples: `UserService`, `ApiResponse`, `DatabaseConfig`

### Constants and Environment Variables
- Use `UPPER_SNAKE_CASE` for environment variables, global constants, and configuration values
- Examples: `API_KEY`, `MAX_RETRY_COUNT`, `DATABASE_URL`

## Code Style

## Line Length and Formatting
- Enforce 80-character line limit across all code files
- When lines exceed 80 characters, suggest breaking them at logical points:
  - **Before operators** (`&&`, `||`, `|`, `>`, `>>`, `.`) - preferred break point
  - **Before logical keywords** (`if`, `for`, `while`, `case`) - second choice
  - **At natural word boundaries in strings** - for long text
  - **Before opening parentheses or brackets** - for complex expressions
- Use backslash (`\`) for line continuation in shell scripts
- **Indent continuation lines with 2 additional spaces** for readability
- **Align continuation lines** with the start of the command when possible
- Prefer breaking before `&&` and `||` operators for command chaining
- Keep the operator at the end of the line (before `\`) for clarity

## Shell Script Specific Formatting
- **Command chaining**: Break before `&&` and `||` operators
  ```bash
  # Good
  command -v xclip >/dev/null 2>&1 \
    && xclip -selection clipboard -o && return

  # Avoid
  command -v xclip >/dev/null 2>&1 && xclip -selection clipboard -o && return
  ```
- **Long echo statements**: Break at natural word boundaries
  ```bash
  # Good
  echo "Error: No clipboard utility found. " \
    "Install one of: wl-clipboard, xclip, xsel" >&2

  # Avoid
  echo "Error: No clipboard utility found. Install one of: wl-clipboard, xclip, xsel" >&2
  ```
- **Case statements**: Keep simple cases on one line, break complex ones
  ```bash
  # Good
  Darwin) pbpaste ;;
  Linux)
    command -v wl-paste >/dev/null 2>&1 && wl-paste && return
    ;;
  ```
- **Indentation**: Use 2 spaces for shell script indentation
- **Continuation alignment**: Align continuation lines with the command start when possible

## JavaScript Specific Formatting
- **Logical operators**: Break each condition on its own line when exceeding 80 chars
  ```javascript
  // Good - each condition on its own line
  if (condition1
    && condition2
    && condition3) {
    // code
  }

  // Also good - single line if under 80 chars
  if (condition1 && condition2) {
    // code
  }
  ```
- **Method chaining**: Break only when entire chain exceeds 80 chars, keep first method on same line
  ```javascript
  // Good - first method stays on same line
  object.method1(x => someCallBackFunc)
    .method2(y => anotherCallBackFunc)
    .method3(...)

  // Good - single line if under 80 chars
  object.method1().method2().method3()
  ```
- **Comma-first style**: Use comma-first for multi-line destructuring and parameters
  ```javascript
  // Good - comma-first destructuring
  const {
    some_prop
  , someFunction
  , option1
  , longerOpt2
  , object_opt3
  } = opts

  // Good - comma-first function parameters
  function myFunction(
    param1
  , param2
  , param3
  , param4
  ) {
    // code
  }

  // Good - comma-first array destructuring
  const [
    first
  , second
  , third
  , fourth
  ] = some_array
  ```

### Semicolons
- Do NOT use semicolons to end lines
- Rely on automatic semicolon insertion (ASI)
- **Exception**: Use semicolons to separate multiple statements on the same line when it improves readability and saves vertical space
- Examples:
  ```javascript
  const user_name = 'john'
  const age = 25

  function getUserInfo() {
    return { user_name, age }
  }

  // Exception - multiple related operations on same line
  v1_ptr++; v2_ptr++  // Pointer increments
  i++; j++; k++       // Multiple counters
  x += 1; y += 1      // Related calculations
  ```

### Variable Declarations
- Prefer destructuring assignment for multiple variables on one line
- Only use comma syntax if performance improvement is at least 1.5x
- Examples:
  ```javascript
  // Preferred
  let [i, j] = [0, 0]
  let { name, age } = user

  // Only if 1.5x+ performance gain
  let i = 0, j = 0
  ```

## General Guidelines

### Code Organization
- Use meaningful, descriptive names
- Prefer explicit over implicit
- Group related functionality together
- Use consistent indentation (2 spaces)

### Modern JavaScript/TypeScript
- Prefer `const` over `let` when possible
- Use arrow functions for short, simple functions
- Prefer template literals over string concatenation
- Use optional chaining and nullish coalescing when appropriate

### Performance Considerations
- Choose readability over micro-optimizations unless performance is critical
- Use appropriate data structures for the problem
- Consider time and space complexity in algorithm selection

## Examples

### Good Examples
```javascript
const API_BASE_URL = 'https://api.example.com'
const MAX_RETRY_ATTEMPTS = 3

class UserService {
  constructor(api_client) {
    this.api_client = api_client
  }

  async getUserData(user_id) {
    try {
      const response = await this.api_client.get(`/users/${user_id}`)
      return response.data
    } catch (error) {
      console.error('Failed to fetch user data:', error)
      throw error
    }
  }
}

function processUserList(users) {
  return users
    .filter(user => user.is_active)
    .map(user => ({
      user_name: user.name,
      email_address: user.email
    }))
}
```

### Avoid
```javascript
// Wrong naming conventions
const userName = 'john'  // Should be user_name
function get_user_data() // Should be getUserData()
class userService {}     // Should be UserService

// Unnecessary semicolons
const name = 'john';
const age = 25;

// Comma syntax without performance justification
let i = 0, j = 0  // Use destructuring unless 1.5x+ faster
```

---
> Source: [david-ly/.dotfiles](https://github.com/david-ly/.dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
