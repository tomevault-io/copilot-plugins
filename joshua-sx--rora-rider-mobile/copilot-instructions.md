## rora-rider-mobile

> This file defines how Claude must behave in this repository.

# CLAUDE.md

This file defines how Claude must behave in this repository.

---

## What You're Building

Rora Ride is a taxi/ride-hailing app for Sint Maarten. Riders find verified taxis, see transparent fare estimates, and log rides safely using QR handshakes between rider and driver.

**Stack:** Expo (React Native), TypeScript, Supabase (PostgreSQL + Auth + Edge Functions + Realtime), Google Maps, Zustand state management.

**Key architectural principles:**
- Ride state transitions are enforced server-side (never client-side)
- Guest mode with token-based history
- QR session system with short-lived JWT tokens
- Append-only ride events ledger for audit trail
- Cash-first, negotiation-friendly

---

## Project Structure

```
rora-rider-mobile/
├── app/                    # Expo Router screens and layouts
│   ├── (tabs)/            # Main app tabs (home, history, settings)
│   ├── driver/            # Driver-specific screens
│   └── settings/          # Settings screens
├── src/
│   ├── components/        # Shared React Native components
│   ├── features/          # Feature-specific logic and components
│   ├── hooks/             # Custom React hooks (use-debounced-value, use-theme-color)
│   ├── services/          # External service integrations (Google Maps)
│   ├── store/             # Zustand state management
│   ├── types/             # TypeScript type definitions (database.ts generated)
│   ├── ui/                # Reusable UI components
│   └── utils/             # Utility functions (pricing, trip-qr, map helpers)
├── supabase/
│   ├── migrations/        # Database migrations (append-only)
│   └── functions/         # Edge Functions (server-side business logic)
├── docs/                  # Architecture and development docs
└── scripts/               # Build and automation scripts
```

**Use tab-completion** to quickly reference files when asking Claude to read or modify them.

---

## Core Files to Know

**State Management (Zustand stores):**
- `src/store/route-store.ts` - Route planning and directions
- `src/store/trip-history-store.ts` - Ride history for guest and authenticated users
- `src/store/favorite-drivers-store.ts` - Favorited drivers
- `src/store/location-store.ts` - Current user location
- `src/store/saved-locations-store.ts` - Saved locations (home, work, etc.)

**Critical Utilities:**
- `src/utils/pricing.ts` - Fare calculation (NEVER modify without approval)
- `src/utils/trip-qr.ts` - QR code generation and JWT token logic
- `src/utils/map.ts` - Google Maps integration helpers

**Type Generation:**
- `src/types/database.ts` - Auto-generated from Supabase schema (NEVER edit manually)

**Documentation:**
- `docs/architecture.md` - Stack details, key decisions, codebase structure
- `docs/feature-development.md` - Development flow, data patterns, Definition of Done
- `docs/security-validation.md` - Security rules for ride state machine

---

## Your Role

You are helping a solo founder ship a ride-hailing platform. Your job is to:
- **Increase clarity**, reduce risk
- **Ship small, correct increments**
- **Avoid over-engineering** (no abstractions until pattern repeats 3+ times)
- **Default to simple, readable, easy to delete**

You are not here to impress. You are here to help ship.

---

## Work Loop

**IMPORTANT: Follow this workflow strictly for all non-trivial tasks.**

### 1. Explore
- **Read relevant files first.** Use tab-completion to reference specific files.
- Understand context before proposing any changes.
- **DO NOT write code yet.**
- For complex investigations, **YOU MUST use subagents** (Task tool with subagent_type=Explore) to search the codebase, understand patterns, or answer architectural questions.
- Examples of when to use subagents:
  - "Where is QR token validation handled?"
  - "How does the ride state machine work?"
  - "What's the pattern for creating new Zustand stores?"

### 2. Plan
- Propose a small, PR-sized plan. **Wait for approval if scope changes.**
- For complex problems, **use extended thinking mode** by including the word "think" in your prompt (levels: "think" < "think hard" < "think harder" < "ultrathink").
- Break down the plan into specific, actionable steps.
- If multiple approaches exist, explain trade-offs and recommend one.

### 3. Execute
- Implement the approved plan. **Minimal changes only.**
- Follow existing patterns in the codebase (look at similar files for reference).
- For UI work with visual mocks:
  - Ask for screenshots or design files
  - Implement the design
  - Take simulator screenshots and iterate until it matches
  - Use 2-3 iterations for best results

### 4. Verify
Follow the verification workflow below **before creating any PR.**

### 5. Commit
- Clear, focused commit message following conventions.
- One logical change per commit.

**Never mix phases. If confused, stop and ask.**

---

## Verification Workflow

**IMPORTANT: Complete these steps IN ORDER before creating a PR.**

1. **Type check:** `npx tsc --noEmit`
2. **Build:** `npx expo prebuild` (if native changes were made)
3. **Test locally:** Run on iOS and Android simulator
   - For UI changes, take screenshots and verify against mocks
   - Test guest mode if applicable
   - Test offline behavior if applicable
4. **Verify acceptance criteria:** Confirm all requirements met
5. **Check Definition of Done:** Review checklist in `docs/feature-development.md`

---

## Commands Reference

**Type Generation:**
```bash
npx supabase gen types typescript --project-id <project-id> > src/types/database.ts
```
Use this after any database schema changes. **NEVER manually edit** `src/types/database.ts`.

**Development:**
```bash
npx expo start              # Start dev server
npx expo start --ios        # Start with iOS simulator
npx expo start --android    # Start with Android emulator
```

**Quality Checks:**
```bash
npx tsc --noEmit           # Type check without emitting files
npx expo prebuild          # Generate native projects (after installing native deps)
```

**Git (you can handle these for me):**
```bash
git status                 # Check working tree
git log --oneline -10      # View recent commits
gh pr create               # Create pull request (after explicit approval)
```

---

## Approval Gates

**YOU MUST wait for explicit approval before:**
- Opening pull requests
- Pushing to remote branches
- Changing architecture or adding dependencies
- Modifying pricing calculation (`src/utils/pricing.ts`)
- Modifying ride state machine logic
- Modifying QR token system (`src/utils/trip-qr.ts`)
- Modifying discovery wave logic
- Creating new database migrations

**When in doubt, ask first.**

---

## Critical Security Rules

**IMPORTANT: Read `docs/security-validation.md` before working on:**
- Ride state transitions
- QR token validation
- Guest mode or session ownership
- Business logic placement (client vs. server)

### Non-Negotiable Security Principles

**YOU MUST enforce these rules:**
- All ride state transitions MUST be validated server-side (Edge Functions only)
- All state changes MUST log ride events (append-only ledger in database)
- UI is NOT a security boundary (never trust client-side validation alone)
- Guest tokens MUST use short-lived JWTs with proper expiry
- Service role keys MUST NEVER appear in client code
- Rate limiting MUST be enforced for guest mode (5 QRs/hour)

**If you notice insecure code, immediately flag it and refuse to proceed until fixed.**

---

## Development Workflows

### Test-Driven Development (Preferred for Features)

1. **Write tests first** based on expected input/output
   - Be explicit that you're doing TDD (avoid mock implementations)
   - Cover edge cases (logged out users, guest mode, offline, etc.)
2. **Run tests and confirm they fail** (don't write implementation yet)
3. **Commit the tests** when approved
4. **Write code to pass the tests** (don't modify tests)
5. **Iterate** until all tests pass (2-3 rounds typical)
6. **Verify with independent check** (use subagent or manual review)
7. **Commit the implementation** when approved

### Mobile UI Development

1. **Get visual mock** (screenshot, Figma file, or description)
2. **Understand existing patterns** (find similar screens/components)
3. **Implement the design**
4. **Take simulator screenshots** of your implementation
5. **Compare and iterate** (2-3 rounds for polish)
6. **Verify on both iOS and Android**
7. **Commit when approved**

### Bug Fixing

1. **Reproduce the bug** (ask for steps if unclear)
2. **Search git history** if needed (`git log`, `git blame`)
3. **Write a failing test** that captures the bug
4. **Fix the bug**
5. **Confirm test passes**
6. **Verify no regressions** (run related tests)
7. **Commit when approved**

### Codebase Exploration (for onboarding or complex questions)

**Use this for questions like:**
- "How does logging work?"
- "Where are ride state transitions handled?"
- "What edge cases does the QR system handle?"

**Pattern:**
1. **Use subagents** (Task tool with subagent_type=Explore) for broad searches
2. Read relevant files identified
3. Trace code flow through multiple files
4. Search git history for context if needed
5. Provide summary with file references (use `file:line` format)

---

## Over-Engineering Guardrails

**IMPORTANT: Avoid these patterns unless explicitly requested.**

**Avoid:**
- New abstractions (unless pattern repeats 3+ times)
- New libraries (unless replacing something existing)
- Clever indirection where explicit code is clearer
- "Future-proofing" without a real requirement
- Helper functions for one-time operations
- Premature optimization
- Feature flags or backward-compatibility shims without reason

**Default to:** Simple. Readable. Easy to delete.

**If you catch yourself creating an abstraction, stop and ask: "Does this pattern exist 3+ times already?"**

---

## Context Management

**Use `/clear` frequently to keep context focused:**
- Between unrelated tasks
- After completing a major feature
- When context window fills with irrelevant conversation
- Before starting a new workflow phase

**Long sessions can degrade performance. Reset context proactively.**

---

## Data Flow Pattern

All features follow this flow:

```
Screen/Component (React Native)
    ↓ Zustand store or direct Supabase hook
Supabase Client (from lib/supabase)
    ↓ RPC call, direct query, or Edge Function
Supabase Edge Function (for business logic)
    ↓ Validate state + permissions + log events
Supabase PostgreSQL (with RLS policies)
    ↓ Realtime subscription (if applicable)
Component updates automatically
```

**Key principle: Types flow one direction (schema → generated types → components). Never duplicate types manually.**

---

## Guest Mode Pattern

**IMPORTANT: Always handle guest mode correctly.**

```typescript
// Always check for guest token when user is not authenticated
const userId = session?.user?.id
const guestTokenId = await getGuestTokenId(deviceId)

// Store ride against appropriate identifier
const rideSession = await createRideSession({
  rider_user_id: userId || null,
  guest_token_id: userId ? null : guestTokenId,
  // ... other fields
})
```

**Test both authenticated and guest flows for every ride-related feature.**

---

## Secrets & Environment

**IMPORTANT: Security > convenience.**

- Never hardcode secrets
- Use environment variables (maintain `.env.example` when adding new vars)
- Validate required env vars at app startup
- Never use service role key in client code
- Never commit `.env` files

**If you see hardcoded secrets, immediately flag and refuse to proceed.**

---

## Git & GitHub Workflows

**You can handle most git operations for me:**
- Writing commit messages (look at recent history for style)
- Searching git history (`git log`, `git blame`, `git show`)
- Resolving simple conflicts
- Creating pull requests (after explicit approval)

**Commit Message Format:**
- `feat:` new user-facing functionality
- `fix:` bug fixes
- `refactor:` internal restructuring (no behavior change)
- `docs:` documentation only
- `chore:` tooling, dependencies, maintenance
- `test:` adding or updating tests

**Examples:**
- `feat: add QR code scanner for ride sessions`
- `fix: prevent duplicate trip history entries in guest mode`
- `refactor: extract pricing calculation into utility function`

**Pull Requests:**
- One issue per PR, small and reviewable
- Include: what changed, why it changed, how to test it
- Wait for explicit approval before creating

---

## Common Mistakes to Avoid

1. **Jumping to code without exploring** - Always read relevant files first
2. **Modifying generated files** - Never edit `src/types/database.ts` manually
3. **Client-side state transitions** - Always validate server-side
4. **Forgetting guest mode** - Test authenticated AND guest flows
5. **Skipping verification** - Always complete verification workflow
6. **Creating PRs without approval** - Always wait for explicit permission
7. **Over-abstracting** - Keep it simple until pattern repeats 3+ times
8. **Ignoring existing patterns** - Look at similar code first

---

## Complex Task Checklist

For large tasks (migrations, fixing many lint errors, multi-step workflows):

1. **Create a Markdown checklist** in a scratch file or GitHub issue
2. **List all items** with file names and line numbers
3. **Work through items one by one**
4. **Check off each item** as you complete it
5. **Verify at the end** that all items are complete

This prevents missed work and provides visibility into progress.

---

## Final Rule

If something feels complex, slow, or fragile:
- **Stop**
- **Simplify**
- **Ask before continuing**

Calm progress beats fast chaos.

**When uncertain about approach, security, or scope: ask first, code second.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshua-sx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joshua-sx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
