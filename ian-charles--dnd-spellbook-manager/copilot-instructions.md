## dnd-spellbook-manager

> Every line of code and every dependency is a liability. Write for clarity, maintainability, and performance. Fast is a feature.

# Gemini Development Guidelines

## Core Philosophy

Every line of code and every dependency is a liability. Write for clarity, maintainability, and performance. Fast is a feature.

## Technical Design Documents

Create a design doc for any work > 1 week or that affects architecture, performance, or APIs.

**Template**: Problem Statement → Goals/Non-Goals → Solution (architecture, API, data model) → Performance considerations → Alternatives → Testing/Rollout → Timeline

## TDD Workflow

### Red-Green-Refactor
1. **Red**: Write failing test → `git commit -m "test: add test for [feature]"`
2. **Green**: Minimal code to pass → `git commit -m "feat: implement [feature]"`  
3. **Refactor**: Improve quality → `git commit -m "refactor: [improvement]"`

**Principles**: One assertion per test • Test behavior, not implementation • AAA pattern • Fast tests first (unit > integration > e2e)

## Git Commits

### Format
`<type>(<scope>): <subject>`

### Types
- `feat`: New feature
- `fix`: Bug fix
- `test`: Add or modify tests
- `refactor`: Code refactoring (no behavior change)
- `docs`: Documentation only
- `perf`: Performance improvement
- `chore`: Build process, dependencies, etc.

### Rules
1. **Always run tests before committing** - `npm test` must pass
2. **NEVER bypass or kill git hooks** - Hooks exist for quality control:
   - NEVER use `--no-verify` to bypass hooks
   - NEVER kill hook processes (pkill, Ctrl+C, etc.)
   - NEVER disable or remove hooks to commit faster
   - NEVER modify git hooks unless explicitly instructed to do so
   - If pre-commit hook fails, fix the issue, don't bypass it
   - If tests are slow, optimize them, don't skip them
   - If hooks are broken, fix the hooks, don't bypass them
   - Bypassing hooks = shipping broken code to production
   - Killing hooks = worse than bypassing (no record of what was skipped)
3. **Commit after each meaningful change**:
   - After test passes (RED → GREEN)
   - After refactoring (GREEN → REFACTOR)
   - After each complete feature
4. **Single purpose per commit**:
   - Each commit should have ONE clear purpose
   - If a commit has multiple unrelated changes, split it into multiple commits
   - Examples: Don't mix "fix bug" with "add feature" in same commit
5. **Never commit**:
   - Broken tests
   - Commented-out code
   - Debugging console.log statements
   - Files with secrets (.env, credentials.json)
6. **Write clear messages**:
   - Subject: What changed (imperative mood: "add", not "added")
   - Body: Why it changed (optional but recommended)

### Examples
```bash
# Good commits
git commit -m "test: add test for mobile spell expansion"
git commit -m "fix: React Fragment key warning preventing mobile expansion"
git commit -m "refactor: extract scroll behavior to helper function"

# Bad commits
git commit -m "fix stuff"  # Too vague
git commit -m "WIP"  # Work in progress shouldn't be committed
git commit -m "updated code"  # Doesn't explain what changed
```

### Before Every Commit
```bash
# Run tests
npm test

# Check what you're committing
git diff --cached

# Verify no debug code
git diff --cached | grep -i "console.log\|debugger\|TODO"
```

## Code Style

### Self-Documenting Code
- **Names reveal intent**: `getUserByEmail()` not `getUser()` or `gube()`
- **Functions do one obvious thing**: No surprises, no side effects
- **Variables explain themselves**: `daysUntilExpiration` not `d` or `days`
- **Code structure tells the story**: Logical flow that reads top-to-bottom

### Naming: Scope Matters

Names inherit context from their enclosing scopes. The tighter the scope, the shorter the name can be.

**Naming by scope:**
- **Single letter** (`i`, `j`, `n`): Loop indices, very short functions
- **Short** (`user`, `err`, `ctx`): Function parameters, local variables
- **Descriptive** (`activeUsers`, `retryCount`): Function/method level
- **Fully qualified** (`MaxRetryAttempts`, `DefaultTimeoutSeconds`): Package/global level

```go
// Bad: Overly verbose in tight scope
for customerIndex := 0; customerIndex < len(customers); customerIndex++ {
    processCustomer(customers[customerIndex])
}

// Good: Short name in tight scope
for i, c := range customers {
    processCustomer(c)
}
```

```python
# Good: Progressive naming based on scope
def calculate_total(orders):
    t = 0  # 't' is fine - function name provides context
    for o in orders:
        t += o.amount
    return t
```

**Function constraints**: < 20 lines, < 3 parameters, single responsibility  
**Structure**: Early returns, UPPER_SNAKE_CASE constants, DRY, YAGNI

## Comments: Self-Documenting Code First

**Primary goal**: Write code so clear that comments become unnecessary. If you need to explain WHAT the code does, rewrite the code.

### When Comments ARE Needed
- **Public APIs**: Document contracts and usage
- **Non-obvious business rules**: Include ticket/requirement references  
- **Critical warnings**: Consequences that aren't apparent
- **Performance trade-offs**: Why you chose this approach
- **Workarounds**: Temporary fixes with removal timeline

### ❌ Never Write
```python
i += 1  # increment i (obvious)
# old_function() (commented-out code)
#--------- GETTERS --------- (noise)
```

### ✅ Good Comments (When Necessary)
```python
# Public API documentation
def calculate_tax(amount: float, region: str) -> float:
    """Calculate tax for a given amount and region.
    
    Args:
        amount: Pre-tax amount in USD
        region: ISO 3166-2 code (e.g., 'US-CA')
    Returns:
        Tax amount in USD
    """

# Critical business logic
# Users qualify for free shipping if premium OR order > $100
# UNLESS promo code 'SHIPIT' (Finance JIRA-4567)
eligible = (user.is_premium or order.final_total > 100) and promo != 'SHIPIT'

# Warning about non-obvious behavior
# CRITICAL: Payment gateway sends duplicate webhooks
if payment_already_processed(id):
    return {"status": "already_processed"}
```

**Remember**: The best comment is the one you didn't have to write because the code was clear.

## Language Guidelines

### Prefer Strong Typing

Always prefer strongly typed languages. Even in dynamically typed languages, use type checking tools.

**Type checking hierarchy:**
1. **Statically typed languages** (Go, Rust, Java, C#): Compile-time safety
2. **Type-annotated dynamic languages** (TypeScript, Python with mypy): Runtime language with static analysis
3. **Dynamic with type hints** (Python 3.5+, Ruby 3+): Documentation and tooling support

**Never ship without type checking:**
- TypeScript over JavaScript
- Python with type hints + mypy
- Flow or TypeScript for React
- Use strict mode where available

### Python
```python
# ALWAYS use type hints
def process(items: List[str]) -> Optional[List[str]]:
    """Process items with documentation."""
    return [item.strip() for item in items if item]

# Run mypy for static type checking
# mypy --strict src/
```

### TypeScript
```typescript
// Prefer TypeScript over JavaScript
// Use strict mode in tsconfig.json
const getData = async (id: string): Promise<Data> => {
    return (await api.get<Data>(`/data/${id}`)).data;
};

// Never use 'any' without justification
```

### Go
```go
// Go is statically typed by default ✓
func ProcessFile(path string) error {
    file, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("opening: %w", err)
    }
    defer file.Close()
    return process(file)
}
```

**Configuration for type safety:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

```ini
# mypy.ini
[mypy]
python_version = 3.11
strict = True
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True
```

## Testing

```python
def test_user_purchase():
    # Arrange
    user = User(balance=100)
    # Act
    result = user.purchase(Item(price=50))
    # Assert
    assert result.success and user.balance == 50
```

**Coverage**: Unit 80%+ • Integration for critical paths • E2E for happy paths

### ALWAYS Run Tests
1. **Before committing**: `npm test` must pass
2. **After making changes**: Verify nothing broke
3. **Before deploying**: Full test suite including E2E

Never skip tests. If tests are slow, fix them. If tests are flaky, fix them. If tests don't exist, write them.

### UI Testing with Puppeteer

**Critical UI Testing Requirements:**

1. **Test Multiple Contexts**:
   - Mobile (375x667 - iPhone SE)
   - Tablet (768x1024 - iPad)
   - Desktop (1920x1080 - Full HD)

2. **Test Both Themes**:
   - Light mode: `page.emulateMediaFeatures([{ name: 'prefers-color-scheme', value: 'light' }])`
   - Dark mode: `page.emulateMediaFeatures([{ name: 'prefers-color-scheme', value: 'dark' }])`

3. **NEVER use wait() or sleep() with arbitrary durations**:

   ❌ **ANTI-PATTERN - Arbitrary waits**:
   ```javascript
   await wait(500);  // WRONG! Race condition waiting to happen
   await wait(1000); // WRONG! Flaky and slow
   await page.waitForTimeout(300); // WRONG! Deprecated and brittle
   ```

   ✅ **CORRECT - Wait for specific state with timeout**:
   ```javascript
   // Wait for element to exist
   await page.waitForSelector('.result', { timeout: 10000 });

   // Wait for element to be visible
   await page.waitForSelector('.modal', { visible: true, timeout: 5000 });

   // Wait for navigation to complete
   await page.goto(url, { waitUntil: 'networkidle2' });

   // Wait for specific condition
   await page.waitForFunction(
     () => document.querySelector('.count')?.textContent === '5',
     { timeout: 10000 }
   );

   // Wait for element to disappear
   await page.waitForFunction(
     () => !document.querySelector('.loading-spinner'),
     { timeout: 5000 }
   );
   ```

   **Why arbitrary waits are bad**:
   - **Flaky**: May work locally but fail in CI or on slower machines
   - **Slow**: Often wait longer than necessary "to be safe"
   - **Race conditions**: Don't guarantee the state you actually need
   - **Maintenance nightmare**: Magic numbers that need tweaking when app changes

   **When you think you need wait()**:
   - Ask: "What state am I actually waiting for?"
   - Use `waitForSelector`, `waitForFunction`, or `waitForNavigation`
   - Set appropriate timeout (default 30000ms is often fine)
   - If you absolutely must wait for animation, use `waitForFunction` to check CSS animation state

4. **Scroll to See Content** (like a real user):
   ```javascript
   // ALWAYS scroll elements into view before interacting
   await element.evaluate(el => el.scrollIntoView({ behavior: 'instant', block: 'center' }));

   // Wait for element to be in viewport and clickable
   await page.waitForFunction(
     (el) => {
       const rect = el.getBoundingClientRect();
       return rect.top >= 0 && rect.bottom <= window.innerHeight;
     },
     { timeout: 5000 },
     element
   );

   await element.click();

   // Wait for result to appear
   await page.waitForSelector('.result', { visible: true, timeout: 5000 });
   ```

5. **Capture Console Errors**:
   ```javascript
   // Monitor console for errors
   const consoleMessages = [];
   page.on('console', msg => {
     console.log(`[CONSOLE ${msg.type()}]:`, msg.text());
     consoleMessages.push({ type: msg.type(), text: msg.text() });
   });

   page.on('pageerror', error => {
     console.log(`[PAGE ERROR]:`, error.message);
     consoleMessages.push({ type: 'error', text: error.message });
   });

   // After test, check for errors
   const errors = consoleMessages.filter(m => m.type === 'error');
   if (errors.length > 0) {
     console.log('Console errors found:', errors);
   }
   ```

6. **Take Screenshots**:
   ```javascript
   // Before interaction
   await page.screenshot({ path: 'before.png' });

   // After interaction
   await page.screenshot({ path: 'after.png' });

   // Full page screenshot
   await page.screenshot({ path: 'fullpage.png', fullPage: true });
   ```

### Complete UI Test Example

```javascript
const puppeteer = require('puppeteer');

describe('Mobile Spell Expansion', () => {
  let browser, page;
  const consoleMessages = [];

  beforeAll(async () => {
    browser = await puppeteer.launch({
      headless: false,  // See what's happening (use true for CI)
      devtools: true,   // Open DevTools
    });
    page = await browser.newPage();

    // Capture console
    page.on('console', msg => consoleMessages.push({ type: msg.type(), text: msg.text() }));
    page.on('pageerror', err => consoleMessages.push({ type: 'error', text: err.message }));
  });

  test('expansion works on mobile in dark mode', async () => {
    // Set mobile viewport
    await page.setViewport({ width: 375, height: 667 });

    // Set dark mode
    await page.emulateMediaFeatures([{ name: 'prefers-color-scheme', value: 'dark' }]);

    await page.goto('http://localhost:5173', { waitUntil: 'networkidle2' });
    await page.waitForSelector('.spell-row', { timeout: 10000 });

    // Get spell (5th one to test scrolling)
    const spells = await page.$$('.spell-row');
    const spell = spells[4];

    // Scroll into view and wait for it to be in viewport
    await spell.evaluate(el => el.scrollIntoView({ behavior: 'instant', block: 'center' }));
    await page.waitForFunction(
      (el) => {
        const rect = el.getBoundingClientRect();
        return rect.top >= 0 && rect.bottom <= window.innerHeight;
      },
      { timeout: 5000 },
      spell
    );

    // Take before screenshot
    await page.screenshot({ path: 'mobile-before.png' });

    // Click and wait for expansion to appear
    await spell.click();
    await page.waitForSelector('.spell-inline-expansion', { visible: true, timeout: 5000 });

    // Take after screenshot
    await page.screenshot({ path: 'mobile-after.png' });

    // Verify expansion visible with content
    const expansionInfo = await page.evaluate(() => {
      const exp = document.querySelector('.spell-inline-expansion');
      return {
        visible: exp && exp.getBoundingClientRect().height > 0,
        hasText: exp && exp.textContent.length > 100,
        description: exp?.querySelector('.spell-expanded-description')?.textContent.substring(0, 50)
      };
    });

    expect(expansionInfo.visible).toBe(true);
    expect(expansionInfo.hasText).toBe(true);
    expect(expansionInfo.description).toBeTruthy();

    // Check for console errors
    const errors = consoleMessages.filter(m => m.type === 'error');
    expect(errors).toHaveLength(0);
  });

  afterAll(async () => {
    await browser.close();
  });
});
```

**Why This Matters:**
- **Scrolling**: Elements outside viewport won't be interactive - tests will fail or give false positives
- **Console errors**: React warnings and errors often prevent UI from rendering correctly
- **Multiple contexts**: Mobile users are ~50% of traffic - desktop-only testing misses half your users
- **Screenshots**: Visual proof tests actually ran and UI looks correct

### Local Debugging Tools

When working on UI features, create temporary debugging scripts to iterate quickly:

```javascript
// debug-feature.cjs (temporary file, don't commit)
const puppeteer = require('puppeteer');

async function debugFeature() {
  const browser = await puppeteer.launch({
    headless: false,  // Watch it run
    devtools: true,   // See console
    slowMo: 250       // Slow down to see actions
  });

  const page = await browser.newPage();

  // Your debugging code here
  await page.goto('http://localhost:5173');
  // ... interact with UI

  // Keep browser open
  console.log('Browser open - press Ctrl+C when done');
  await new Promise(resolve => setTimeout(resolve, 600000)); // 10 min

  await browser.close();
}

debugFeature().catch(console.error);
```

**When to use debugging tools:**
1. Building new UI features
2. Investigating visual bugs
3. Testing responsive behavior
4. Verifying animations/transitions
5. Checking console for errors

**Always delete debugging scripts before committing** - use `.gitignore` or manual cleanup.

## Performance: Fast is my Favorite Feature

### Database
- Always index frequently queried columns
- Prevent N+1 queries with eager loading
- Paginate large datasets
- Use EXPLAIN for query analysis

### Data Transfer
- Send only needed fields
- Implement pagination
- Compress responses > 1KB

### Caching
L1: Memory → L2: Redis → L3: Database

### Budgets
- API p95: < 200ms
- DB query p95: < 50ms
- Time to Interactive: < 3s
- Bundle size: < 500KB

## Dependencies

1. **Standard library first**
2. **Popular, maintained libraries** (active, documented, semantic versioning)
3. **Local implementation** for simple needs

**Binary services** (Redis, PostgreSQL): Use official images, set resource limits, health checks, security best practices

## Documentation Requirements

Every project MUST have executable setup/test/deploy instructions.

### README.md Structure
```markdown
# Project Name
What it does and why

## Prerequisites
- Node.js 18+ / Python 3.11+ / Go 1.21+
- Docker 24+ & Docker Compose 2.20+

## Quick Start
# Clone and setup
git clone <repo> && cd project
npm install
cp .env.example .env

# Run locally
npm run db:migrate
npm run dev  # http://localhost:3000

# Or use Docker
docker-compose up

## Testing
npm test              # unit
npm run test:integration
npm run test:e2e
npm run test:coverage # all

## Production
docker build -t app:latest -f Dockerfile.prod .
docker-compose -f docker-compose.prod.yml up -d

## API Docs
Swagger: http://localhost:3000/api-docs
```

### Required Files
- `Dockerfile` (dev with hot reload)
- `Dockerfile.prod` (optimized production)
- `docker-compose.yml` (local dev)
- `docker-compose.prod.yml` (production)
- `.env.example` (all config options)

## Project Structure

```
src/
├── core/           # Business logic
├── infrastructure/ # External services  
├── api/           # HTTP layer
└── shared/        # Utilities

tests/
├── unit/
├── integration/
├── e2e/
```

## Refactoring Patterns

- **Extract Method**: Break large functions into smaller ones
- **Early Return**: Guard clauses instead of nested ifs
- **Replace Magic Numbers**: Use named constants
- **Introduce Parameter Object**: Group related parameters

## Code Review Checklist

- [ ] Tests pass
- [ ] No N+1 queries
- [ ] Comments explain why, not what
- [ ] README updated
- [ ] Performance acceptable
- [ ] No commented code

## Core Principles

**Code is a Liability**: Every line must justify itself. Delete whenever possible.

**Ship Iteratively**: Make it work → Make it right → Make it fast (when measured)

**Team Over Individual**: Consistency > preference. Reviews are learning opportunities.

**Performance Matters**: Users don't care about elegance if it's slow.

---

*"Programs must be written for people to read, and only incidentally for machines to execute."* - Harold Abelson

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ian-charles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
