## director-mode-lite

> A simple calculator module for demonstrating Auto-Loop.

# Calculator Demo Project

A simple calculator module for demonstrating Auto-Loop.

## Tech Stack

- Language: JavaScript (Node.js 18+)
- Testing: Node.js built-in test runner

## Project Structure

```
01-calculator/
├── calculator.js       # Main implementation
├── calculator.test.js  # Unit tests
└── package.json        # Project config
```

## Development Policies

- Always write tests before implementation (TDD)
- Use conventional commits
- Keep functions simple and focused
- Handle edge cases (division by zero)

## Test Command

```bash
npm test
# or
node --test
```

## Expected Functions

```javascript
// calculator.js
function add(a, b) { ... }
function subtract(a, b) { ... }
function multiply(a, b) { ... }
function divide(a, b) { ... }  // throws on zero
```

---
> Source: [claude-world/director-mode-lite](https://github.com/claude-world/director-mode-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
