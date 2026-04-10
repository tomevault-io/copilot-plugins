## tidyco-apqp

> This guide covers build/test commands, code style, and core rules for agents working in the Tidyco APQP repository.

# AGENTS.md — Coding Guidelines for Agentic Operations

This guide covers build/test commands, code style, and core rules for agents working in the Tidyco APQP repository.

## Quick Commands

### Testing
```bash
npm test                          # Run all Jest tests
npm test -- path/to/test.test.js  # Run a single test file
npm test -- --testNamePattern="name"  # Run tests matching a pattern
npm run check:all                 # Full validation suite (see below)
```

### Validation & Checks
```bash
npm run check:load-order          # Verify script load order in index.html
npm run check:syntax              # Validate JavaScript syntax
npm run check:rls                 # Audit RLS policies
npm run check:subscriptions       # Check realtime cleanup
npm run check:mobile              # Verify mobile breakpoints
npm run check:modals              # Audit modal state handling
npm run check:state               # Track global state variables
npm run check:coverage            # Generate test coverage report
```

### Linting & Formatting (NPI portal only)
```bash
npm run lint:npi                  # Run ESLint on NPI portal
npm run format:npi                # Run Prettier on NPI portal
```

### Documentation (Wiki)
```bash
npm run wiki:build-index          # Rebuild wiki search index
npm run wiki:check                # Full wiki validation
```

## Architecture Snapshot

- **Framework**: Vanilla JavaScript SPA (no build pipeline)
- **Backend**: Supabase (Auth, Postgres, Realtime)
- **Package Manager**: npm
- **Testing**: Jest with jsdom environment
- **Linting**: ESLint (NPI portal)
- **Formatting**: Prettier
- **Script Load Order** (critical): `state.js → auth.js → db.js → helpers.js → navigation.js → realtime.js → portals → app.js`

## Hard Rules (Non-Negotiable)

1. **Script Order**: Always preserve exact load order in `index.html`. Verify with `npm run check:load-order`.
2. **No Duplicate `const`**: Never introduce duplicate variable declarations in the same scope.
3. **Global State**: Keep mutable state in `core/js/state.js` with defaults initialized.
4. **HTML Escaping**: Use `esc()` helper for any user data rendered into HTML strings (prevents XSS).
5. **Navigation**: Use `navigate()` function for route changes so realtime cleanup runs.
6. **Auth-Only RLS**: Do NOT filter client queries by `user_id`; rely on Supabase RLS policies.
7. **Documentation**: Keep new plans/docs in `plans/` unless it is a core root doc.
8. **Mobile-First CSS**: Use both breakpoints:
   - `@media (max-width: 767px)` (mobile)
   - `@media (min-width: 768px)` (desktop)
9. **Guide Updates**: When adding/changing features on content pages, update the matching entry in `GUIDE_CONTENT` in `utils/js/guide.js`.

## Code Style & Conventions

### Imports & Globals
- No module imports (vanilla JS SPA). All dependencies loaded via `<script>` tags in `index.html`.
- Global objects available: `supa` (Supabase client), `db`, `currentUser`, `GATE_DEFS`, `FAMILIES`, etc.
- All ESLint-defined globals listed in `eslint.config.js` for NPI portal scope.

### Formatting
- **Single quotes** for strings: `'string'` not `"string"`
- **No semicolons** at end of statements
- **Tab width**: 2 spaces
- **Line length**: Max 100 characters
- **Trailing commas**: None
- Tool: `npm run format:npi` applies Prettier formatting

### Variable Naming
- Use `camelCase` for variables and functions: `getCurrentUser()`, `progId`, `bomPickTarget`
- Use `UPPER_CASE` for constants: `GATE_DEFS`, `FAMILIES`, `BOM_TYPES`
- Use descriptive names: `isLoading` not `loading_state`
- Global state variables tracked in `core/js/state.js`

### Functions & Error Handling
- Keep functions focused and under 50 lines when possible
- Always handle async errors with try-catch or `.catch()`
- Use early returns to reduce nesting
- Log errors clearly: `console.error('Operation failed:', error)`
- Return predictable types (null, empty array, or error object)
- Example pattern:
  ```javascript
  async function fetchProgram(id) {
    try {
      const { data, error } = await supa.from('programs').select().eq('id', id)
      if (error) throw error
      return data[0] || null
    } catch (err) {
      console.error('Failed to fetch program:', err)
      return null
    }
  }
  ```

### Conditions & Equality
- Use strict equality: `===` and `!==` (ESLint enforced)
- Null checks: `value == null` is allowed for null/undefined comparison
- Prefer explicit boolean: `if (isActive === true)` over `if (isActive)`

### Testing Guidelines
- Follow Jest Arrange-Act-Assert structure
- Match test file naming to module purpose
- Use jsdom-safe patterns (see `jest.setup.js`)
- Mock Supabase client calls rather than hitting live services
- Include at least one unhappy path test per new behavior
- Avoid testing implementation internals; test observable behavior
- Keep async tests using `async/await` (avoid false positives)

Example test structure:
```javascript
describe('functionName', () => {
  it('should handle the happy path', () => {
    // Arrange
    const input = { /* ... */ }
    // Act
    const result = myFunction(input)
    // Assert
    expect(result).toBe(expected)
  })

  it('should handle errors gracefully', () => {
    // Arrange
    const badInput = null
    // Act & Assert
    expect(() => myFunction(badInput)).toThrow()
  })
})
```

## ID Generation Pattern
Use prefixed short IDs consistently:
```javascript
const id = 'f_' + Math.random().toString(36).substr(2, 5)
```

**Prefixes**: `f_` = mode, `e_` = effect, `c_` = cause, `r_` = risk, `a_` = action

## Save Debouncing
Data persists to Supabase with an **800–900 ms debounce**. Plan UX accordingly.

## Changelog Management

After each logical change, add one entry near the top of `CHANGELOG.md`:
```
## YYYY-MM-DD | Short descriptive title | Brief reason
```

Example:
```
## 2026-03-25 | Add mobile-friendly BOM editor | Improve tablet UX for field technicians
```

## Validation Workflow

Before committing:
1. Run `npm test` — ensure all tests pass
2. Run `npm run check:all` — run full validation suite
3. Add changelog entry to `CHANGELOG.md`
4. Verify script order with `npm run check:load-order`

## Canonical Detail Rules

For detailed guidance, consult:
- **Test rules**: `.github/instructions/testing.instructions.md`
- **Security**: `.claude/rules/security.md`
- **Database/RLS**: `.claude/rules/database.md`
- **Navigation**: `.claude/rules/navigation.md`
- **Realtime**: `.claude/rules/realtime.md`
- **Components**: `.claude/rules/components.md`

## MCP Tools

This repository has MCP servers configured in `.mcp.json`:
- **Serena** (`oraios/serena`): Use for Serena ALM/Octane integration
- **GitHub** (`github/github-mcp-server`): Use for GitHub API operations

## OpenWolf Protocol

Follow the rules in `.claude/rules/openwolf.md` for all file operations. Key points:
- Check `.wolf/anatomy.md` before reading project files
- Check `.wolf/cerebrum.md` Do-Not-Repeat list before generating code
- Update `.wolf/anatomy.md` and append to `.wolf/memory.md` after writing files
- Log bugs to `.wolf/buglog.json` after fixes
- Run `openwolf designqc` when asked to evaluate UI design

## User Context

The primary user is non-technical. When communicating, use plain language and avoid technical jargon. Keep error messages and inline comments clear and actionable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dlimb4876)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dlimb4876)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
