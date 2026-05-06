## gastown-ui

> This project uses **br** (beads) for issue tracking. Run `br onboard` to get started.

# Agent Instructions

# AGENTS.md — Platform-Optimized SvelteKit PWA (Mobile + Tablet + Desktop)

This project uses **br** (beads) for issue tracking. Run `br onboard` to get started.

## Project Structure

```
gastown-ui/                          # Town root
├── .beads/                          # Town-level beads (mayor mail, HQ coordination)
├── AGENTS.md                        # This file
├── CLAUDE.md                        # Mayor context
│
└── gastown-ui/                      # Rig: SvelteKit UI project
    ├── .repo.git/                   # Bare git repository (shared by all worktrees)
    │
    ├── mayor/
    │   ├── rig/                     # Mayor's reference clone (READ-ONLY)
    │   │   └── .beads/              # Rig beads database (source of truth)
    │   └── state.json
    │
    ├── polecats/                    # Worker worktrees
    │   ├── furiosa/                 # Polecat worktree
    │   │   └── .beads/redirect      # → ../../mayor/rig/.beads
    │   └── nux/                     # Polecat worktree
    │       └── .beads/redirect      # → ../../mayor/rig/.beads
    │
    ├── refinery/
    │   └── rig/                     # Merge queue processor worktree (on main)
    │       └── .beads/redirect      # → ../../mayor/rig/.beads
    │
    ├── crew/                        # Human-managed worktrees
    │   └── amrit/
    │
    └── witness/                     # Polecat lifecycle monitor
        └── state.json
```

### Key Concepts

| Component | Purpose |
|-----------|---------|
| **Town** | Workspace root containing all rigs |
| **Rig** | Project container (one per repo) |
| **Polecat** | Worker agent with isolated git worktree |
| **Refinery** | Merge queue processor (rebases, tests, merges) |
| **Witness** | Monitors polecat lifecycle |
| **Beads** | Issue tracking, shared via redirect to mayor/rig/.beads |

## Quick Reference

```bash
br ready              # Find available work
br show <id>          # View issue details
br claim <id>         # Claim work (sets assignee, derives in_progress display status)
br close <id>         # Complete work
br sync               # Sync with git
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   br sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---

## Critical Fixes Phase - Patterns & Discoveries

### Form Validation Pattern

**When**: All user forms need to validate input before submission

**Pattern**:
```typescript
import { z } from 'zod';

// 1. Define validation schema
const formSchema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters'),
  type: z.enum(['task', 'bug', 'feature', 'epic']),
  priority: z.number().min(0).max(4)
});

// 2. In form handler, validate BEFORE submission
async function handleSubmit(e: Event) {
  e.preventDefault();
  errors = {};
  
  const result = formSchema.safeParse({
    title: formTitle,
    type: selectedType,
    priority: selectedPriority
  });
  
  if (!result.success) {
    const fieldErrors = result.error.flatten().fieldErrors;
    errors = Object.fromEntries(
      Object.entries(fieldErrors).map(([key, msgs]) => [key, msgs?.[0] || ''])
    );
    hapticError();
    return; // Don't submit
  }
  
  // Continue with submission
}

// 3. Display errors inline
{#if errors.title}
  <p class="text-sm text-destructive">{errors.title}</p>
{/if}

// 4. Disable submit until valid
<button disabled={formTitle.length < 3}>Submit</button>
```

**Applied To**:
- Work page: Issue creation, Convoy creation, Sling work forms

**Key Points**:
- Use Zod for client-side validation (npm install zod)
- Validate on submit, not on blur
- Show specific error messages per field
- Disable submit button while invalid
- Clear errors on successful submission

---

### Error State Integration Pattern

**When**: Any page that fetches data asynchronously

**Pattern**:
```svelte
<script>
  let error: Error | null = null;
  let loading = true;
  
  async function fetchData() {
    try {
      const res = await fetch('/api/data');
      if (!res.ok) throw new Error('Failed to fetch');
      data = await res.json();
    } catch (e) {
      error = e instanceof Error ? e.message : 'Unknown error';
    } finally {
      loading = false;
    }
  }
  
  function handleRetry() {
    error = null;
    fetchData();
  }
  
  onMount(() => {
    fetchData();
  });
</script>

{#if loading}
  <SkeletonCard type="mail" count={5} />
{:else if error}
  <ErrorState
    title="Failed to load"
    message={error}
    onRetry={handleRetry}
    showRetryButton={true}
  />
{:else if data.length === 0}
  <EmptyState title="No data" description="Create your first item" />
{:else}
  <!-- Content here -->
{/if}
```

**Applied To**:
- Mail page
- Agents page
- Work page
- Any async data fetch

**Key Points**:
- Always have loading, error, empty, and content states
- Error state should show specific error message
- Retry button re-fetches data
- Clear error on successful retry
- Use haptic feedback for errors

---

### Testing Procedures

**Created Documentation**:
1. **KEYBOARD_TESTING.md** - Tab order, focus, screen readers (VoiceOver, NVDA)
2. **DARK_MODE_TESTING.md** - Contrast ratios, WAVE, axe DevTools
3. **PERFORMANCE_TESTING.md** - Lighthouse, Core Web Vitals, 3G throttle
4. **BROWSER_TESTING.md** - Chrome, Firefox, Safari, iOS, Android

**Use These For**:
- Pre-launch verification
- Weekly/monthly monitoring
- Regression detection
- Accessibility compliance

**Success Criteria**:
- Lighthouse ≥90 on all pages
- LCP <2.5s on 3G
- Zero console errors
- Keyboard nav works on all pages
- 4.5:1 contrast in dark mode
- Works on Chrome, Firefox, Safari, iOS, Android

---

### Known Patterns & Gotchas

**Safe Area Insets (iOS)**:
- Use `env(safe-area-inset-bottom)` for bottom nav padding
- Apply to: BottomNav, floating buttons
- Formula: `padding-bottom: calc(height + env(safe-area-inset-bottom))`

**100vh Issue (Mobile Safari)**:
- 100vh includes address bar, causes overflow
- Use `100dvh` (dynamic viewport height) instead
- Or use `height: 100%` with full-height parent

**Select Dropdown Styling (Safari iOS)**:
- Cannot be styled on iOS Safari
- Accept default native styling
- Consider custom component for critical dropdowns

**Focus Management**:
- Always provide visible focus ring (2px ring-offset)
- Use Tailwind: `focus:ring-2 focus:ring-accent`
- Test with keyboard (Tab key)
- Verify with keyboard-only mode (disconnect mouse)

**Dark Mode Support**:
- Use CSS custom properties (--color-*)
- Test with: System dark mode, browser dark mode toggle
- Minimum contrast: 4.5:1 for all text

---

## Package Manager: Bun

**IMPORTANT**: This project uses **bun** as the package manager. Do NOT use npm, yarn, or pnpm.

```bash
# Install dependencies
bun install

# Add a new package
bun add <package>

# Add dev dependency
bun add -d <package>

# Run scripts
bun run dev
bun run build
bun run check

# Run tests
bun test
```

**Lock file**: `bun.lockb` - Do NOT create or commit `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`.

## Dependencies Added

```json
{
  "dependencies": {
    "zod": "^3.22.0"  // Client-side form validation
  }
}
```

Install with: `bun add zod`

---

## File Locations

| Purpose | Location |
|---------|----------|
| Form validation | `/src/routes/work/+page.svelte` (schemas at top of script) |
| Error state pattern | Mail, Agents, Work pages |
| Empty state component | `$lib/components/EmptyState.svelte` |
| Skeleton loaders | `$lib/components/SkeletonCard.svelte` |
| Keyboard testing | `KEYBOARD_TESTING.md` |
| Dark mode testing | `DARK_MODE_TESTING.md` |
| Performance testing | `PERFORMANCE_TESTING.md` |
| Browser testing | `BROWSER_TESTING.md` |


<!-- bv-agent-instructions-v1 -->

---



This repo targets **three first-class experiences**: **mobile**, **tablet**, and **desktop**.
"Responsive" is not enough — each platform may require **different IA, navigation, density, and interaction patterns**.

If you're an AI coding agent working in this repo:
- Treat **Mobile / Tablet / Desktop** as **separate products** sharing a codebase.
- Prefer **small, surgical changes** with clear verification steps.
- Never "fix desktop" by breaking mobile (or vice versa). If there's a tradeoff, propose an explicit platform-specific solution.

---

## Non-negotiables (ship criteria)

1. **Platform parity**: every user-facing change must be checked on:
   - **Mobile** (360×800, coarse pointer)
   - **Tablet** (768×1024, portrait + landscape)
   - **Desktop** (1440×900, keyboard + hover)
2. **Accessibility**: keyboard nav, visible focus, semantic landmarks, label controls, reduced motion support.
3. **Performance budgets** (per route, per platform):
   - Avoid layout shift; no jarring loading "pops".
   - Keep interaction responsive on low-end mobile devices.
4. **Stability**: errors are caught + surfaced; no silent failures.
5. **Design integrity**: no "generic UI slop". Consistent spacing, type scale, and motion language.

---

## Platform taxonomy (how we decide what to build)

Use **capabilities**, not just width:
- `pointer: coarse|fine`
- `hover: none|hover`
- `prefers-reduced-motion`
- `prefers-color-scheme`
- `prefers-reduced-data` (when applicable)

### Recommended breakpoints (guidance, not dogma)

| Platform | Width | Characteristics |
|----------|-------|-----------------|
| **Mobile** | 0–640px | `pointer: coarse`, `hover: none` |
| **Tablet** | 641–1024px | Mixed: coarse + some keyboard |
| **Desktop** | 1025px+ | `pointer: fine`, `hover: hover`, full keyboard |

If a component needs different behavior across platforms, do **adaptive UI** (platform-specific variants), not fragile CSS hacks.

---

## Skills to load (project + personal skills)

When your task touches these areas, you MUST apply the corresponding skill:

| Skill Path | Domain |
|------------|--------|
| `.claude/skills/frontend-dev-guidelines` | Component structure, file organization, types, routing patterns, perf basics |
| `.claude/skills/mobile-frontend-design` | Touch UX, mobile IA/nav, density rules, gesture safety, thumb zones, forms |
| `.claude/skills/sveltekit-pwa-skills` | PWA installability, offline strategy, caching, update flows, SW constraints |
| `.claude/skills/production-hardening-frontend` | Error handling, observability, security headers, resilience, CI quality gates |

If you can't find a referenced skill locally, continue with best practices and leave a note in the PR describing what was missing.

---

## Architecture expectations (platform-first)

### 1) Separate "shells" per platform (recommended)

Keep shared domain UI, but allow platform-specific shells:

| Shell | Purpose |
|-------|---------|
| `MobileShell.svelte` | Bottom nav / simplified IA / large tap targets |
| `TabletShell.svelte` | Split panes / master-detail / hybrid nav |
| `DesktopShell.svelte` | Side nav / dense tables / keyboard-first controls |

**Rule:** shared components should not assume a specific shell.

### 2) Component strategy: shared core + platform adapters

For complex components:
- `Component.svelte` (shared logic + rendering primitives)
- `ComponentMobile.svelte` / `ComponentTablet.svelte` / `ComponentDesktop.svelte`
- A small adapter decides which one to render based on capabilities.

This prevents endless conditional styling and makes platform optimization real.

### 3) Data + loading: stable layout, no jank

- Avoid early-return spinners that change page structure.
- Prefer skeletons/placeholders that preserve layout and prevent CLS.
- Keep expensive work off the main thread; debounce heavy handlers on mobile.

---

## Agent roles (how to collaborate)

### Agent: `platform-mobile`

**Mission:** mobile UX excellence on real devices.
- One-handed navigation, thumb zones, large touch targets.
- Forms: correct input types, autofill, no tiny hit areas.
- Avoid hover-only affordances; ensure discoverability.

**Acceptance checklist (mobile):**
- [ ] Tap targets ≥ 44px, readable type, no horizontal scroll.
- [ ] Key flows doable with one hand.
- [ ] No blocking long tasks on main thread; scroll stays smooth.

### Agent: `platform-tablet`

**Mission:** tablet ergonomics + productivity.
- Support portrait + landscape.
- Consider split views / master-detail where it improves usability.
- Avoid "giant phone UI stretched to tablet".

**Acceptance checklist (tablet):**
- [ ] Layout uses width intelligently (columns/panes).
- [ ] Touch works; keyboard optional but supported.

### Agent: `platform-desktop`

**Mission:** fast, dense, keyboard-friendly desktop UI.
- Full keyboard navigation and shortcuts where appropriate.
- Hover states enhance but aren't required.
- Efficient density for power users (tables, multi-column layouts).

**Acceptance checklist (desktop):**
- [ ] Tab order correct; focus visible.
- [ ] No "wasted whitespace" that harms productivity.

### Agent: `pwa-offline`

**Mission:** installability + offline-first correctness.
- Install prompt flow + update flow ("new version available").
- Caching strategy: app shell + runtime caching (explicit allowlist).
- Offline UX: meaningful empty states; queued actions if applicable.

**Acceptance checklist (PWA):**
- [ ] Manifest valid, icons present, installable.
- [ ] Offline route fallback works; no infinite loading.

### Agent: `prod-hardening`

**Mission:** reliability, security, observability, CI discipline.
- Structured error handling; user-safe messages; logged details.
- CSP/headers guidance (where applicable).
- Performance regression checks; bundle awareness.

**Acceptance checklist (hardening):**
- [ ] Error paths exercised.
- [ ] Security + perf pitfalls avoided (unsafe HTML, leaking secrets, etc.).

---

## Definition of Done (DoD) for any PR

Every PR must include:

1. **Platform verification notes**
   - Mobile / Tablet / Desktop screenshots or short notes per platform
2. **A11y checks**
   - Keyboard path confirmed + focus states confirmed
3. **Performance sanity**
   - Note anything that could regress (images, bundles, large lists)
4. **PWA impact statement**
   - "No SW change" OR "Updated caching/update flow" with test notes
5. **Risk + rollback**
   - What could break? How would we notice?

---

## Testing & local verification

```bash
bun install
bun dev
bun lint
bun check        # svelte-check for type errors
bun test         # vitest unit tests
bun test:e2e     # playwright e2e tests
bun build && bun preview
```

**Manual QA matrix:**

| Platform | Viewport | Conditions |
|----------|----------|------------|
| Mobile | 360×800 | Slow 4G throttle + touch emulation |
| Tablet | 768×1024 | Portrait + landscape |
| Desktop | 1440×900 | Keyboard-only pass |

---

## UI/UX rules of thumb (platform-independent)

- Prefer **progressive disclosure**: show essentials first, reveal detail on demand.
- Motion is **subtle** and **optional** (`prefers-reduced-motion` respected).
- Use **consistent spacing scale** and type hierarchy.
- Avoid "mystery meat" interactions; make affordances obvious.

---

## PR communication format (required)

In the PR description, include:

- **What changed**
- **Why**
- **Screens/notes**: Mobile / Tablet / Desktop
- **A11y notes**
- **Perf notes**
- **PWA notes**
- **Risks**

If you must introduce platform divergence, explicitly call it out as:
> "Platform-specific behavior: MOBILE does X, TABLET does Y, DESKTOP does Z — for these reasons."

---

---

# SvelteKit Extreme TDD Agent Instructions

> Optimized prompt for spawning background agents with strict TDD enforcement

---

## Overview

This document serves as the authority for Worker and Reviewer agents implementing SvelteKit/TypeScript code using Extreme TDD (Red-Yellow-Green) methodology.

---

## Context Management Zones

Monitor context usage with `/context` command. Take action based on zone:

| Zone | Usage | Action |
|------|-------|--------|
| 🟢 Green | 0–50% | Work normally, commit frequently |
| 🟡 Yellow | 50–75% | Avoid exploratory reads, be specific with @file references |
| 🔴 Red | 75–90% | Stop new work, create handoff, prepare to `/clear` |
| ⚫ Critical | 90%+ | `/clear` immediately, create handoff document first |

**Rule:** Create handoff document BEFORE context reaches Red zone.

### Session Commands Reference

| Command | When to Use |
|---------|-------------|
| `/context` | Check current context usage (do this regularly) |
| `/clear` | Clear context and start fresh (after creating handoff) |
| `/compact` | Compress history (avoid if possible, prefer handoff + clear) |
| `claude --resume` | Resume a previous session |
| `claude --continue` | Continue the most recent session |

---

## Common Agent Mistakes (AVOID THESE)

| Anti-Pattern | What Agents Do | Why It's Bad |
|--------------|----------------|--------------|
| `// TODO` / `// FIXME` | Stub functions with comments | Never gets implemented |
| `as any` everywhere | Skip type safety | Runtime errors |
| `expect(result).toBeTruthy()` | Weak test assertion | Doesn't verify actual value |
| `@ts-ignore` / `@ts-expect-error` | Hide type errors | Masks real issues |
| Empty function bodies | Satisfy interface | No actual behavior |
| Ignoring Promise rejections | No `.catch()` or try/catch | Silent failures |
| `console.log` debugging | Leave logs in code | Noise in production |

---

## Master Prompt

```
Continue next task. Act sequentially: Worker → Reviewer.
```

---

## WORKER PHASE

**Source of truth:** `agents.md`

### Initial Steps

1. Pick next priority task from board
2. Review existing code for context
3. Ensure correct component/store usage per `agents.md`

---

### Extreme TDD Cycle (MANDATORY FOR EACH FUNCTION)

#### 🔴 RED (Test First)

```bash
bun test -- --run <test_file> # MUST FAIL or NOT COMPILE
```

- Write test before implementation
- Use `it.fails()` or `expect().toThrow()` for error cases
- **Commit:** `test(red): [behavior under test]`

#### 🟡 YELLOW (Minimal Pass)

- Write MINIMAL code to pass—hardcoded values OK
- Example:
  ```typescript
  // RED: test expects parseInput("42") -> { value: 42 }
  // YELLOW (acceptable):
  function parseInput(_input: string): ParseResult {
      return { value: 42 }; // Hardcoded! Will refactor in GREEN
  }
  ```
- Run: `bun test` → MUST PASS
- **Commit:** `feat(yellow): [minimal impl]`

#### 🟢 GREEN (Refactor)

- Proper implementation, generics, error handling
- Run: `bun test` → STILL PASSES
- Run: `bun lint` → NO WARNINGS
- Run: `bun check` → NO TYPE ERRORS
- **Commit:** `refactor(green): [improvements]`

---

### Negative Test Requirements (Per Function)

```typescript
import { describe, it, expect } from 'vitest';

describe('parseInput', () => {
  it('throws on invalid input', () => {
    expect(() => parseInput('not_a_number')).toThrow(ParseError);
    expect(() => parseInput('not_a_number')).toThrow('Invalid format');
  });

  it('throws on empty input', () => {
    expect(() => parseInput('')).toThrow(ParseError);
    expect(() => parseInput('')).toThrow('Input cannot be empty');
  });

  it('handles boundary values', () => {
    expect(() => parseInput(String(Number.MAX_SAFE_INTEGER + 1))).toThrow('Overflow');
  });

  it('returns null for missing optional field', () => {
    const result = parseOptionalField(undefined);
    expect(result).toBeNull();
  });
});
```

#### Required Coverage Per Function

- [ ] Error/exception path tested with SPECIFIC error type/message
- [ ] `null` / `undefined` path tested
- [ ] Boundary: empty string, zero, MAX_SAFE_INTEGER, empty array
- [ ] Invalid state/input

---

### BANNED PATTERNS IN FINAL CODE

```typescript
// ❌ NEVER IN PRODUCTION CODE:
// TODO
// FIXME
// @ts-ignore
// @ts-expect-error
as any
as unknown as SomeType  // double casting
!  // non-null assertion (unless proven safe)
console.log()  // use proper logging
throw new Error('Not implemented')
throw 'string error'  // always use Error objects
```

---

### BANNED TEST PATTERNS

```typescript
// ❌ WEAK ASSERTIONS - REJECT:
expect(result).toBeTruthy();        // What's the value?
expect(result).toBeDefined();       // What's inside?
expect(result).not.toBeNull();      // Use toEqual with expected
expect(error).toBeInstanceOf(Error); // Which error type?

// ✅ STRONG ASSERTIONS - REQUIRED:
expect(result).toEqual({ value: 42, status: 'ok' });
expect(result).toStrictEqual(expectedObject);
expect(() => fn()).toThrow(SpecificError);
expect(() => fn()).toThrow('specific message');
expect(result).toMatchObject({ key: expect.any(String) });
```

---

### Worker Handoff

Before ending your session, create a handoff document so the Reviewer can start with fresh context.

**Create file:** `.claude/handoffs/HANDOFF-{TASK_ID}.md`

```markdown
# Handoff: [Task Title]

**Generated**: [timestamp]
**Task ID**: [TASK_ID]
**Branch**: [current branch]
**Status**: Ready for Review

## Goal
[One-line description of what this task accomplishes]

## Completed
- [x] RED: Test written and failing
- [x] YELLOW: Minimal implementation passing
- [x] GREEN: Refactored with proper error handling

## Files Changed
- `src/lib/[file].ts` - [what was added/changed]
- `src/lib/[file].test.ts` - [tests added]

## Key Decisions
| Decision | Rationale |
|----------|-----------|
| [Choice made] | [Why this approach] |

## Failed Approaches (Don't Repeat These)
[If any approaches were tried and abandoned, document why]

## Code Context
```typescript
// Key function signature or interface
export function example(input: string): Result {
  // Focus on lines X-Y
}
```

## Test Summary
- Happy path: ✅
- Error cases: ✅  
- Boundary cases: ✅
- Null/undefined: ✅

## Next Steps for Reviewer
1. Verify TDD commit sequence (red → yellow → green)
2. Run banned pattern scans
3. Verify negative test coverage
4. Run full validation suite

## Resume Instructions
```bash
# Start fresh session, then:
cat .claude/handoffs/HANDOFF-{TASK_ID}.md
git log --oneline | head -10
bun test
bun lint
bun check
```
```

**Output ONLY:**
```
TASK_ID: xxx | COMMIT: yyy | HANDOFF: .claude/handoffs/HANDOFF-xxx.md
```

**Do NOT send completion mail.**

---

## REVIEWER PHASE (FRESH CONTEXT)

### Starting Fresh Session

**CRITICAL:** Reviewer MUST start with a clean context window. Do not continue in the same session as Worker.

**Fresh Start Protocol:**
1. Clear current context (`/clear` or start new session)
2. Read the handoff document: `cat .claude/handoffs/HANDOFF-{TASK_ID}.md`
3. Verify git state matches handoff
4. Proceed with review checklist

**Source of truth:** `agents.md` + handoff document + git history ONLY

**Input:** Task ID + Commit ID + Handoff file path from Worker

---

### 1. Verify TDD Commit Pattern

```bash
git log --oneline | head -30
# Expected pattern per feature:
# abc123 refactor(green): ...
# def456 feat(yellow): ...
# ghi789 test(red): ...
```

**Reject if:** commits don't follow red → yellow → green sequence

---

### 2. Scan for Banned Patterns

```bash
# Type escape hatches
grep -rn "as any\|@ts-ignore\|@ts-expect-error" src/

# Incomplete implementation
grep -rn "TODO\|FIXME\|Not implemented" src/

# Debug artifacts
grep -rn "console\.log\|console\.debug" src/ | grep -v "// allowed:"

# Unsafe assertions
grep -rn "!\." src/ | grep -v "\.svelte" # non-null assertions in TS
```

**Action:** Any match = immediate fix required

---

### 3. Scan for Weak Tests

```bash
grep -rn "toBeTruthy\|toBeDefined\|not\.toBeNull\|toBeInstanceOf(Error)" src/ tests/
# Each hit must be replaced with specific value assertion
```

**Action:** Replace weak assertions with strong ones

---

### 4. Verify Negative Coverage

```bash
bun test 2>&1 | grep -E "throws|error|empty|invalid|boundary|null|undefined"
# Must exist for each new public function
```

**Action:** Add missing negative tests

---

### 5. Verify Component/Store Compliance

Cross-reference implementation against `agents.md`:
- [ ] Correct stores used
- [ ] Component interfaces match spec
- [ ] No unauthorized dependencies
- [ ] Platform-specific variants where required

---

### 6. Full Validation Suite

```bash
bun lint
bun check
bun test
bun build
```

**All must pass with zero warnings/errors.**

---

### On ANY Gap Found

1. **Fix immediately** (do not defer to Worker)
2. **Commit:** `fix(review): [description]`
3. **Re-run all checks** from step 1

---

### Completion Mail (MANDATORY)

Send ONLY after ALL checks pass.

**Required contents:**
- Task ID
- Full commit chain (red → yellow → green → any fixes)
- Gaps found & fixed (list each)
- `bun test` summary output
- `bun check` clean confirmation

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────┐
│ SVELTEKIT EXTREME TDD CHEAT SHEET                       │
├─────────────────────────────────────────────────────────┤
│ 🔴 RED:    bun test → FAIL/NO COMPILE                  │
│ 🟡 YELLOW: hardcode OK, bun test → PASS                │
│ 🟢 GREEN:  refactor, lint clean, test → STILL PASS      │
├─────────────────────────────────────────────────────────┤
│ EVERY fn needs:                                         │
│   • Happy path test (toEqual with value)                │
│   • Error test (toThrow with specific error)            │
│   • Null/undefined test                                 │
│   • Boundary test                                       │
├─────────────────────────────────────────────────────────┤
│ BANNED: as any, @ts-ignore, toBeTruthy(), console.log   │
└─────────────────────────────────────────────────────────┘
```

---

## Commit Message Format

```bash
# TDD Cycle Commits
test(red): add failing test for user authentication
feat(yellow): minimal impl for user auth (hardcoded)
refactor(green): proper auth with session handling

# Review Fix Commits  
fix(review): replace as any with proper typing
fix(review): add missing negative test for empty input
fix(review): remove console.log and use logger
```

---

## Example: Complete TDD Cycle

### Feature: `validateEmail(input: string): ValidationResult`

#### 🔴 RED Commit

```typescript
// src/lib/validation/email.test.ts
import { describe, it, expect } from 'vitest';
import { validateEmail, ValidationError } from './email';

describe('validateEmail', () => {
  it('accepts valid email', () => {
    const result = validateEmail('user@example.com');
    expect(result).toEqual({ valid: true, email: 'user@example.com' });
  });

  it('throws on empty email', () => {
    expect(() => validateEmail('')).toThrow(ValidationError);
    expect(() => validateEmail('')).toThrow('Email cannot be empty');
  });

  it('throws on missing @ symbol', () => {
    expect(() => validateEmail('userexample.com')).toThrow(ValidationError);
    expect(() => validateEmail('userexample.com')).toThrow('Missing @ symbol');
  });

  it('throws on invalid domain', () => {
    expect(() => validateEmail('user@')).toThrow(ValidationError);
    expect(() => validateEmail('user@')).toThrow('Invalid domain');
  });
});
```

```bash
bun test  # FAILS - validateEmail doesn't exist
git commit -m "test(red): add validation tests for email"
```

#### 🟡 YELLOW Commit

```typescript
// src/lib/validation/email.ts
export class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

export type ValidationResult = { valid: true; email: string };

export function validateEmail(input: string): ValidationResult {
  if (input === 'user@example.com') {
    return { valid: true, email: 'user@example.com' };
  } else if (input === '') {
    throw new ValidationError('Email cannot be empty');
  } else if (!input.includes('@')) {
    throw new ValidationError('Missing @ symbol');
  } else {
    throw new ValidationError('Invalid domain');
  }
}
```

```bash
bun test  # PASSES
git commit -m "feat(yellow): minimal email validation (partially hardcoded)"
```

#### 🟢 GREEN Commit

```typescript
// src/lib/validation/email.ts
export class ValidationError extends Error {
  constructor(
    message: string,
    public readonly code: 'EMPTY' | 'MISSING_AT' | 'INVALID_DOMAIN' | 'INVALID_LOCAL'
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

export type ValidationResult = { valid: true; email: string };

export function validateEmail(input: string): ValidationResult {
  if (input.trim() === '') {
    throw new ValidationError('Email cannot be empty', 'EMPTY');
  }

  const atIndex = input.indexOf('@');
  
  if (atIndex === -1) {
    throw new ValidationError('Missing @ symbol', 'MISSING_AT');
  }

  const local = input.slice(0, atIndex);
  const domain = input.slice(atIndex + 1);

  if (local === '') {
    throw new ValidationError('Invalid local part', 'INVALID_LOCAL');
  }

  if (domain === '' || !domain.includes('.')) {
    throw new ValidationError('Invalid domain', 'INVALID_DOMAIN');
  }

  return { valid: true, email: input.toLowerCase() };
}
```

```bash
bun test   # STILL PASSES
bun lint   # CLEAN
bun check  # NO TYPE ERRORS
git commit -m "refactor(green): proper email parsing with full validation"
```

---

## Async Function Testing

```typescript
import { describe, it, expect, vi } from 'vitest';

describe('fetchUser', () => {
  it('returns user on success', async () => {
    const result = await fetchUser(1);
    expect(result).toEqual({ id: 1, name: 'Alice' });
  });

  it('throws NotFoundError for missing user', async () => {
    await expect(fetchUser(999)).rejects.toThrow(NotFoundError);
    await expect(fetchUser(999)).rejects.toThrow('User not found');
  });

  it('throws TimeoutError on slow response', async () => {
    vi.useFakeTimers();
    const promise = fetchUserWithTimeout(1, 100);
    vi.advanceTimersByTime(150);
    await expect(promise).rejects.toThrow(TimeoutError);
    vi.useRealTimers();
  });
});
```

---

## Svelte Component Testing

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/svelte';
import Button from './Button.svelte';

describe('Button', () => {
  it('renders with correct text', () => {
    render(Button, { props: { label: 'Click me' } });
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick handler when clicked', async () => {
    const handleClick = vi.fn();
    render(Button, { props: { label: 'Click', onClick: handleClick } });
    
    await fireEvent.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when loading', () => {
    render(Button, { props: { label: 'Submit', loading: true } });
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('shows spinner when loading', () => {
    render(Button, { props: { label: 'Submit', loading: true } });
    expect(screen.getByTestId('spinner')).toBeInTheDocument();
  });
});
```

---

## Store Testing

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { get } from 'svelte/store';
import { createUserStore } from './userStore';

describe('userStore', () => {
  let store: ReturnType<typeof createUserStore>;

  beforeEach(() => {
    store = createUserStore();
  });

  it('initializes with null user', () => {
    expect(get(store)).toEqual({ user: null, loading: false, error: null });
  });

  it('sets loading state during fetch', async () => {
    const promise = store.fetchUser(1);
    expect(get(store).loading).toBe(true);
    await promise;
    expect(get(store).loading).toBe(false);
  });

  it('sets user on successful fetch', async () => {
    await store.fetchUser(1);
    expect(get(store).user).toEqual({ id: 1, name: 'Alice' });
  });

  it('sets error on failed fetch', async () => {
    await store.fetchUser(999);
    expect(get(store).error).toEqual('User not found');
    expect(get(store).user).toBeNull();
  });
});
```

---

## SvelteKit Load Function Testing

```typescript
import { describe, it, expect, vi } from 'vitest';
import { load } from './+page.server';

describe('+page.server load', () => {
  it('returns user data for valid id', async () => {
    const result = await load({ 
      params: { id: '1' },
      fetch: vi.fn()
    });
    
    expect(result).toEqual({
      user: { id: 1, name: 'Alice' }
    });
  });

  it('throws 404 for invalid user', async () => {
    await expect(
      load({ params: { id: '999' }, fetch: vi.fn() })
    ).rejects.toMatchObject({
      status: 404,
      body: { message: 'User not found' }
    });
  });

  it('throws 400 for non-numeric id', async () => {
    await expect(
      load({ params: { id: 'abc' }, fetch: vi.fn() })
    ).rejects.toMatchObject({
      status: 400,
      body: { message: 'Invalid user ID' }
    });
  });
});
```

---

## API Route Testing

```typescript
import { describe, it, expect } from 'vitest';
import { POST } from './+server';

describe('POST /api/validate-email', () => {
  it('accepts valid email', async () => {
    const request = new Request('http://localhost/api/validate-email', {
      method: 'POST',
      body: JSON.stringify({ email: 'user@example.com' }),
      headers: { 'Content-Type': 'application/json' }
    });

    const response = await POST({ request });
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data).toEqual({ valid: true, email: 'user@example.com' });
  });

  it('returns 400 for empty email', async () => {
    const request = new Request('http://localhost/api/validate-email', {
      method: 'POST',
      body: JSON.stringify({ email: '' }),
      headers: { 'Content-Type': 'application/json' }
    });

    const response = await POST({ request });
    const data = await response.json();

    expect(response.status).toBe(400);
    expect(data).toEqual({ 
      error: 'ValidationError',
      code: 'EMPTY',
      message: 'Email cannot be empty'
    });
  });
});
```

---

## Error Type Testing

```typescript
describe('ValidationError', () => {
  it('has correct name', () => {
    const err = new ValidationError('test', 'EMPTY');
    expect(err.name).toBe('ValidationError');
  });

  it('has correct message', () => {
    const err = new ValidationError('Email cannot be empty', 'EMPTY');
    expect(err.message).toBe('Email cannot be empty');
  });

  it('has correct code', () => {
    const err = new ValidationError('test', 'INVALID_DOMAIN');
    expect(err.code).toBe('INVALID_DOMAIN');
  });

  it('is instanceof Error', () => {
    const err = new ValidationError('test', 'EMPTY');
    expect(err).toBeInstanceOf(Error);
    expect(err).toBeInstanceOf(ValidationError);
  });
});
```

---

## Final Checklist Before Handoff

### Worker Checklist

- [ ] All functions have RED → YELLOW → GREEN commits
- [ ] Each function has ≥1 negative test
- [ ] No banned patterns in code
- [ ] No weak assertions in tests
- [ ] `bun test` passes
- [ ] `bun lint` clean
- [ ] `bun check` passes (no type errors)

### Reviewer Checklist

- [ ] Commit history shows proper TDD sequence
- [ ] `grep` scans return no banned patterns
- [ ] Negative test coverage verified
- [ ] Component/store compliance per `agents.md`
- [ ] All gaps fixed and committed
- [ ] Full validation suite passes
- [ ] Completion mail sent with all required info

---

## Verification Matrix

| Check | Command | Must Pass |
|-------|---------|-----------|
| Type Safety | `bun check` | ✅ |
| Linting | `bun lint` | ✅ |
| Unit Tests | `bun test` | ✅ |
| E2E Tests | `bun test:e2e` | ✅ |
| Build | `bun build` | ✅ |
| Platform QA | Manual (Mobile/Tablet/Desktop) | ✅ |

If anything in these instructions is unclear, refer to `/Users/amrit/Documents/Projects/Rust/mouchak/gastown` as the **source of truth** for how the system works — we’re building the UI to match that codebase exactly.

---
> Source: [Avyukth/gastown_ui](https://github.com/Avyukth/gastown_ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
