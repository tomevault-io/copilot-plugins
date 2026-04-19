## ficta

> Universal test data generator: Node.js + Browser + CLI → CSV/JSON/XML/Excel/TSV/SQL

# Cursor IDE Rules for Ficta

## Project Overview
Universal test data generator: Node.js + Browser + CLI → CSV/JSON/XML/Excel/TSV/SQL

## Core Principles
1. **Universal Core** - src/core.js has ZERO Node/browser deps
2. **ES Modules Only** - import/export (no require)
3. **Pure Functions** - No side effects in core logic
4. **100% Test Coverage** - All code must be tested
5. **Options Objects** - Use destructured params

## Quick File Reference
- `src/core.js` - Data generation (universal)
- `src/formatters.js` - Format converters (Node.js)
- `src/node.js` - Node.js API
- `src/browser.js` - Browser API
- `cli.js` - CLI interface
- `tests/*.test.js` - Test suite

## Code Style

### Functions
```javascript
// ✅ Good: Destructured options, defaults, JSDoc
/**
 * Generate data rows
 * @param {Object} options - Generation options
 * @param {Array} options.columns - Column definitions
 * @param {number} [options.rows=100] - Number of rows
 * @returns {Array} Generated rows
 */
export function generateData({ columns, rows = 100, format = 'csv' }) {
  // Pure function logic
  return result;
}

// ❌ Avoid: Positional params, no types
function generateData(columns, rows, format) { }
```

### Error Handling
```javascript
// ✅ Good: Descriptive with context
if (!templates[template]) {
  throw new Error(`Unknown template: ${template}. Available: ${Object.keys(templates).join(', ')}`);
}

// ❌ Avoid: Generic messages
throw new Error('Invalid template');
```

### CSV Escaping
```javascript
// Always escape special characters
if (value.includes(',') || value.includes('"') || value.includes('\n')) {
  return `"${value.replace(/"/g, '""')}"`;
}
```

## Common Tasks

### Add Data Type
File: `src/core.js` → `fakerTypes` object
```javascript
myType: () => getFaker().category.method()
```

### Add Template
File: `src/core.js` → `templates` object
```javascript
myTemplate: "id:autoIncrement,field1:type1,field2:type2"
```

### Add Format
1. `src/formatters.js` → Add `toMyFormat()`
2. `src/node.js` → Add case to switch
3. `cli.js` → Add to format choices
4. Add tests

### Fix Formatter
File: `src/formatters.js` → Find `toXXX()` function

## Testing

### Run Tests
```bash
npm test                    # All tests
npm test -- core.test.js   # Specific file
npm run test:coverage      # With coverage
```

### Test Pattern
```javascript
import { functionName } from '../src/module.js';

test('descriptive test name', () => {
  const result = functionName(input);
  expect(result).toEqual(expected);
});
```

### Vitest
Uses Vitest — `vi.fn()`, `vi.spyOn()`, `vi.useFakeTimers()`. Never use `jest.*`.

## Important Constraints

1. **Core is universal** - No Node/browser code in src/core.js
2. **Faker is lazy** - Call `getFaker()`, not direct access
3. **ES Modules only** - No CommonJS
4. **100% coverage required** - All changes need tests
5. **No side effects in core** - File I/O only in adapters

## Dependencies

### Production
- `@faker-js/faker` - Data generation
- `exceljs`, `xml2js` - Formatters
- `yargs` - CLI parsing

> Note: `csv-writer` is **not** a dependency. CSV output is handled by the built-in `toCSV()` in `src/formatters.shared.js`.

### Development  
- `vitest` - Testing (with `@vitest/coverage-v8`)
- `esbuild` - Browser bundles
- `csv-parse` - Test helpers

Don't suggest new dependencies without checking if existing ones can be used.

## Typical Changes (90% of work)

1. **Add data type** → Extend `fakerTypes` in core.js
2. **Add template** → Extend `templates` in core.js  
3. **Fix formatter** → Edit function in formatters.js
4. **Add format** → Three files (formatters, node, cli)

## Documentation Files

- **AGENTS.md** - Complete API reference, module guide, code examples (start here!)
- **ARCHITECTURE.md** - Technical deep dive
- **CONTRIBUTING.md** - Setup, style, testing, PR checklist
- **README.md** - User documentation

## Commands Reference

```bash
npm test                    # Run tests
npm run test:coverage       # Coverage report
npm run build              # Build browser bundles
node cli.js --help         # CLI help
node cli.js --list-types   # Show all data types
```

## When Suggesting Code

Always ensure:
- [ ] Uses ES Modules (import/export)
- [ ] Follows pure function pattern
- [ ] Uses destructured options
- [ ] Has descriptive errors
- [ ] Includes tests
- [ ] Environment code in correct file
- [ ] CSV values properly escaped
- [ ] Has JSDoc comments

## Performance Tips

- Large datasets: Consider chunking
- Excel: Memory-intensive, use streaming
- Browser: Keep downloads < 50MB
- Avoid heavy types (`paragraph`) for large data

## Faker Usage

```javascript
// ✅ Good: Call getFaker()
const value = getFaker().person.fullName();

// ❌ Bad: Direct access
const value = faker.person.fullName(); // May not be initialized
```

## Remember

This project values:
- **Simplicity** over complexity
- **Universality** over platform-specific
- **Readability** over clever optimizations
- **Testing** over "trust me it works"

When in doubt, check existing code for patterns!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/izaccavalheiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
