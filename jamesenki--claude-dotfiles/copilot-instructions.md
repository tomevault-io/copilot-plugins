## claude-dotfiles

> These instructions apply to all Claude Code sessions.

# Claude Code Global Instructions

These instructions apply to all Claude Code sessions.

---

## Writing Style Rules

### Presentations & Executive Communications

When creating **presentation materials**, **executive summaries**, **slide decks**, or **communications intended for leadership/executives**, follow these rules:

1. **6th grade reading level** - Use simple words, short sentences, no jargon
2. **Assume limited attention spans** - Get to the point immediately
3. **Reductive thinking** - Boil complex ideas down to their simplest form
4. **Format for scanning:**
   - Bullet points over paragraphs
   - 1 idea per slide/section
   - Bold the key takeaway
   - No more than 3 bullets per group
5. **Lead with the "so what"** - Start with the conclusion, then support it
6. **Avoid:**
   - Hedge words ("might", "could potentially")
   - Passive voice
   - Acronyms without definition
   - Dense text blocks
   - Nuance that doesn't affect the decision
   - Unnecessary theory or conceptual background
   - Long evidence trails - if evidence is needed, make it brief (one line max)

**Example transformation:**
- ❌ "The implementation of the proposed solution would potentially result in operational efficiencies"
- ✅ "This saves $2M per year"

---

## Development Workflow

### Planning First

Before writing any code:
1. Understand the requirement fully - ask clarifying questions
2. Design the solution - identify components, interfaces, data flow
3. Write tests first - tests ARE the specification
4. Then implement - code exists to make tests pass

### Test-Driven Development

1. **Write test first** (Red) - Test fails because code doesn't exist
2. **Write minimal code** (Green) - Just enough to pass
3. **Refactor** - Clean up while keeping tests green
4. **Commit** - Only when tests pass and code is clean

Rules:
- Tests describe requirements; code implements tests
- Each test verifies ONE behavior
- Test names read like requirements (e.g., `shouldRejectPasswordsUnder8Characters`)
- Mock external dependencies (DB, APIs, file system)
- Never generate code first then tests - always tests first

### Atomic Commits

One commit = one logical, complete unit of work:
- Can be reverted independently without breaking anything
- All tests pass after this commit
- Doesn't depend on future commits for correctness

**What belongs in ONE commit:**
- Feature code + its tests + related docs
- e.g., API endpoint = controller + service + test + docs update

**What needs SEPARATE commits:**
- Feature work vs refactoring (never mix)
- Feature work vs formatting changes
- Unrelated bug fixes

**Commit messages:**
- Subject: Present tense, imperative, <50 chars (e.g., "Add user authentication")
- Body: Explain WHY, not what (the diff shows what)

---

## Code Design Principles

### SOLID

**Single Responsibility**
- Each class/function has ONE reason to change
- Test: Can you describe it in one sentence without "and" or "or"?
- File naming should reflect responsibility (e.g., `UserRepository` not `DatabaseService`)

**Open/Closed**
- Open for extension, closed for modification
- Add behavior by writing new code, not editing existing code
- Use interfaces/composition to extend

**Liskov Substitution**
- Derived classes must work wherever base class is expected
- Subclass can't strengthen preconditions or weaken postconditions

**Interface Segregation**
- Small, focused interfaces over large catch-all interfaces
- If a class implements an interface but loves not use all methods, split the interface

**Dependency Inversion**
- Depend on abstractions, not concrete implementations
- Inject dependencies; don't create them inside the class
- High-level modules don't depend on low-level modules

### DRY (Don't Repeat Yourself)

**Rule of Three:**
- Duplication OK for 1-2 occurrences
- Abstract on 3rd occurrence IF pattern is clear
- Don't abstract on hunches; wait for clarity

**True vs Accidental Duplication:**
- True: Same logic, same intent → abstract it
- Accidental: Looks same, different intent → keep separate

**YAGNI Over Premature Abstraction:**
- Only abstract for features needed NOW
- Explicit duplication > obscure abstraction
- Readability > deduplication

---

## Continuous Delivery

### Always Deployable
- Every commit leaves codebase in deployable state
- All tests pass before merge
- Never leave work that blocks production

### Small Batches
- Merge to main at minimum daily
- Keep feature branches < 2 days
- One logical change per commit
- Large features → break into deployable slices

### Separate Deploy from Release
- Deployed code ≠ visible feature
- Use feature flags for user-facing changes
- Code goes to prod disabled; product controls visibility
- Enables rapid rollback

---

## 12 Factor Principles

These apply to any backend service or application:

1. **One Codebase** - One repo per app, multiple deploys (dev/staging/prod)
2. **Explicit Dependencies** - All deps declared in manifest (e.g., package.json, requirements.txt, go.mod, Cargo.toml, etc.); pin versions
3. **Config in Environment** - No hardcoded URLs, keys, or secrets in code
4. **Stateless Processes** - No in-process session state; use backing services
5. **Dev/Prod Parity** - Same code, same deps, same containers across environments
6. **Logs to Stdout** - Never write to files; let orchestrator handle logs

**Never commit:**
- Database URLs or connection strings
- API keys or secrets
- Environment-specific configuration

---

## Code Quality Gates

Before considering code complete:
- [ ] Tests written first and passing
- [ ] Single responsibility per class/function
- [ ] Dependencies injected, not created
- [ ] Config externalized to environment
- [ ] No hardcoded secrets
- [ ] Commit is atomic and revertible
- [ ] Commit message explains WHY

---

## Project-Specific Rules

The above are universal. Each project should have its own `CLAUDE.md` in the repo root specifying:

- Language/runtime version
- Testing framework and coverage targets
- File/folder organization conventions
- Dependency injection approach
- Feature flag approach (if applicable)
- Commit message format
- Batch size limits (lines/files per commit)
- Deploy pipeline stages
- Logging approach
- Secret management approach

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamesenki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jamesenki)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
