## nyc-transit-hub

> **Project:** NYC Transit Hub

# NYC Transit Hub - Cursor Rules

## Project Overview

**Project:** NYC Transit Hub  
**Purpose:** Personal web app providing real-time MTA transit info, analytics, and accessibility-aware routing  
**Stack:** Next.js (App Router), TypeScript, HeroUI, Tailwind CSS, Prisma/Drizzle, PostgreSQL (Supabase), Recharts

### Key Data Sources
- MTA GTFS-Realtime feeds (subway, bus, LIRR, Metro-North)
- Service alerts (JSON/GTFS)
- Elevator/escalator status feeds
- Static GTFS for station metadata

### Core Features
- Real-time train tracker and station boards
- Accessibility-aware routing (elevator/escalator status)
- Reliability and disruption analytics
- Crowding estimation
- Personal commute assistant

### Node.js Version Requirement
This project requires Node.js >= 20.9.0. The terminal may reset to an older version between commands.

**Always prefix Node.js commands with the correct nvm version:**
```bash
nvm use 20.19.5 && npm run dev
nvm use 20.19.5 && npm run build
nvm use 20.19.5 && npm install <package>
```

---

## Collaboration & Flexibility

### Plans Are Guidelines, Not Constraints

Documentation and plans (including `planForApp.md`) are starting points, not rigid requirements:
- Deviate from the plan when it improves code quality or app functionality
- Prioritize best practices over strict adherence to initial designs
- Refactor structure if a better approach becomes apparent during implementation
- The goal is a well-built app, not perfect plan compliance

### Ask Questions, Don't Assume

Assumptions lead to rework. When in doubt:
- Ask clarifying questions before implementing
- Propose alternatives when you see a better approach
- Flag potential issues early rather than discovering them later
- Provide feedback on trade-offs (performance, complexity, maintainability)

The more questions asked upfront, the fewer problems downstream. Never hesitate to challenge an approach if there's a better way.

---

## Code Quality Standards

### Professional Engineering Style

Write code as a senior software engineer would. All code should be production-quality, maintainable, and self-documenting.

**Modularity:**
- Keep functions short and focused (single responsibility)
- Extract logic into well-named helper functions
- Avoid deep nesting (max 2-3 levels); use early returns and guard clauses
- Split large components into smaller, composable pieces

**Naming Conventions:**
- Use descriptive, meaningful names that reveal intent
- Variables: `stationDepartures`, `activeAlerts`, `elevatorStatus`
- Functions: `fetchRealtimeArrivals()`, `parseGtfsResponse()`, `calculateOnTimeRate()`
- Components: `StationBoard`, `AlertCard`, `ReliabilityChart`
- Avoid abbreviations unless universally understood (e.g., `id`, `url`)

**Code Efficiency:**
- Prefer concise solutions over verbose ones
- If something can be done in 5 lines instead of 50, use 5 lines
- Use modern JavaScript/TypeScript features (destructuring, optional chaining, nullish coalescing)
- Avoid unnecessary abstractions; add complexity only when it provides clear value

**TypeScript Standards:**
- Enable strict mode; never use `any`
- Create explicit interfaces for all data structures
- Handle null/undefined explicitly
- For external data: validate with Zod, then transform and assert types

---

## Framework & Library Guidelines

### Version Policy

Always use the latest stable versions of all libraries and frameworks. Do not pin to older versions unless there is a documented incompatibility. Keep dependencies updated.

### Next.js (App Router)

- Use Server Components by default; add `'use client'` only when needed
- Leverage server-side data fetching in page components
- Use Route Handlers for API endpoints (`/app/api/...`)
- Follow the App Router file conventions (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`)
- Use `next/image` for optimized images
- Use `next/link` for client-side navigation

### HeroUI Components

- Use HeroUI components as the primary UI library
- Follow HeroUI patterns for forms, modals, cards, and navigation
- Extend with Tailwind utilities when needed
- Keep component props explicit and typed

### Tailwind CSS

- Use Tailwind utility classes for styling
- Extract repeated patterns into components, not custom CSS
- Use CSS variables for theme consistency
- Follow mobile-first responsive design

### Database & ORM (Prisma/Drizzle)

- Define clear, normalized schemas
- Use migrations for all schema changes
- Write type-safe queries
- Handle database errors gracefully
- Use transactions for multi-step operations

### Data Fetching

- Use TanStack Query or native fetch with proper caching
- Implement proper loading and error states
- Set appropriate cache/revalidation strategies
- Handle stale data gracefully
- **Deduplicate before rendering:** APIs often return per-relation data (e.g., one arrival per station per train). Always deduplicate by primary identifier (e.g., `tripId`) before rendering lists to avoid duplicate keys and inflated counts.

---

## Source of Truth

**All documentation might be outdated.** The only reliable sources of truth are:
1. **Actual codebase** - Code as it exists now
2. **Live configuration** - Environment variables, configs as actually set
3. **Running infrastructure** - How services actually behave
4. **Actual logic flow** - What code actually does when executed

When docs and reality disagree, **trust reality**. Verify by reading actual code, checking live configs, testing actual behavior.

**Workflow:** Read docs for intent -> Verify against actual code/configs/behavior -> Use reality -> Update outdated docs.

**Applies to:** All `.md` files, READMEs, notes, guides, in-code comments, JSDoc, docstrings, any written documentation.

---

## Research-First Development

### When to Apply

**Complex work (use full protocol):**
Implementing features, fixing bugs (beyond syntax), dependency conflicts, debugging integrations, configuration changes, architectural modifications, data migrations, security implementations, cross-system integrations, new API endpoints.

**Simple operations (execute directly):**
Git operations on known repos, reading files with known exact paths, running known commands, port management on known ports, installing known dependencies, single known config updates.

### The 8-Step Protocol

**Phase 1: Discovery**

1. **Find and read relevant notes/docs** - Search across workspace (notes/, docs/, README), and project .md files. Use as context only; verify against actual code.

2. **Read additional documentation** - API docs, official docs, in-code comments. Use for initial context; verify against actual code.

3. **Map complete system end-to-end**
   - Data Flow & Architecture: Request lifecycle, dependencies, integration points, architectural decisions, affected components
   - Data Structures & Schemas: Database schemas, API structures, validation rules, transformation patterns
   - Configuration & Dependencies: Environment variables, service dependencies, auth patterns, deployment configs
   - Existing Implementation: Search for similar/relevant features that already exist - can we leverage or expand them instead of creating new?

4. **Inspect and familiarize** - Study existing implementations before building new. Look for code that solves similar problems - expanding existing code is often better than creating from scratch. If leveraging existing code, trace all its dependencies first to ensure changes won't break other things.

**Phase 2: Verification**

5. **Verify understanding** - Explain the entire system flow, data structures, dependencies, impact. For complex multi-step problems requiring deeper reasoning, analyze approach, consider alternatives, identify potential issues.

6. **Check for blockers** - Ambiguous requirements? Security/risk concerns? Multiple valid architectural choices? Missing critical info only user can provide? If NO blockers: proceed to Phase 3. If blockers: briefly explain and get clarification.

**Phase 3: Execution**

7. **Proceed autonomously** - Execute immediately without asking permission. Default to action. Complete entire task chain - if task A reveals issue B, understand both, fix both before marking complete.

8. **Update documentation** - After completion, update existing notes/docs (not duplicates). Mark outdated info with dates. Add new findings. Reference code files/lines. Document assumptions needing verification.

---

## Execution Standards

### Default to Action

After completing research and understanding:
- Execute confidently without asking permission
- When user intent is clear, proceed with implementation
- Don't suggest; implement

### When to Proceed Autonomously

- Research complete -> Implementation (task implies action)
- Discovery -> Fix (found issues, understand root cause)
- Phase -> Next Phase (complete task chains)
- Error -> Resolution (errors discovered, root cause understood)
- Task A complete, discovered task B -> continue to B

### When to Stop and Ask

- Ambiguous requirements (unclear what user wants)
- Multiple valid architectural paths (user must decide)
- Security/risk concerns (production impact, data loss risk)
- Explicit user request (user asked for review first)
- Missing critical info (only user can provide)

### Complete Task Chains

- If task A reveals issue B, fix both before marking complete
- Don't stop at the first problem
- Chain related fixes until the entire system works
- No partial work; tasks are complete only when fully functional

### Fix Root Causes

- Investigate symptoms to find underlying issues
- Fix the cause, not just the symptom
- Check for similar issues elsewhere in the codebase
- Consider if the fix should be abstracted for reuse

### Proactive Fixes

When encountered, resolve immediately:
- Dependency conflicts
- Type errors
- Lint warnings
- Build errors
- Configuration mismatches

---

## Quality & Completion Standards

**Task is complete ONLY when all related issues are resolved.**

Think of completion like a senior engineer would: it's not done until it actually works, end-to-end, in the real environment. Not just "compiles" or "tests pass" but genuinely ready to ship.

**Before committing, ask yourself:**
- Does it actually work? (Not just build, but function correctly in all scenarios)
- Did I test the integration points? (Frontend talks to backend, backend to database, etc.)
- Are there edge cases I haven't considered?
- Is anything exposed that shouldn't be? (Secrets, validation gaps, auth holes)
- Will this perform okay? (No N+1 queries, no memory leaks)
- Did I update the docs to match what I changed?
- Did I clean up after myself? (No temp files, debug code, console.logs)

**Complete entire scope:**
- Task A reveals issue B -> fix both
- Found 3 errors -> fix all 3
- Don't stop partway
- Don't report partial completion
- Chain related fixes until system works

---

## Investigation Thoroughness

**When searches return no results, this is NOT proof of absence - it's proof your search was inadequate.**

Before concluding "not found", think about what you haven't tried yet:
- Did you explore the full directory structure?
- Did you search recursively with patterns?
- Did you try alternative terms or partial matches?
- Did you check parent or related directories?
- Question your assumptions - maybe it's not where you expected

**File Search Approach:**
- Start by understanding the environment: Look at directory structure first
- Use the right tool for what you know (exact filename vs partial vs content search)
- When you find what you're looking for, look around - related files are usually nearby
- Be thorough: Try broader patterns, check subdirectories recursively, search by content not just filename

**"File not found" after 2-3 attempts = "I didn't look hard enough", NOT "file doesn't exist."**

---

## Architecture-First Debugging

When debugging, think about architecture and design before jumping to "maybe it's an environment variable" or "probably a config issue."

**The hierarchy of what to investigate:**
1. How things are designed - component architecture, how client and server interact, where state lives
2. Data flow - follow a request from frontend through backend to database and back
3. Only then look at environment config, infrastructure, or tool-specific issues

**When data isn't showing up:**
Think end-to-end:
- Is the frontend actually making the call correctly?
- Are auth tokens present?
- Is the backend endpoint working and accessible?
- Is middleware doing what it should?
- Is the database query correct and returning data?
- How is data being transformed between layers?

Don't assume. Trace the actual path of actual data through the actual system.

---

## Ownership & Cascade Analysis

Think end-to-end: Who else affected? Ensure whole system remains consistent. Found one instance? Search for similar issues. Map dependencies and side effects before changing.

**When fixing, ALWAYS grep for identical patterns:**
```bash
# Found bug in StationBoard.tsx fetching with wrong platform ID?
# Immediately search for the same pattern everywhere:
grep -r "fetch.*stationId" components/
grep -r "api/trains/realtime" .
```

**When fixing, check:**
- Similar patterns elsewhere? (grep the codebase - this is non-negotiable)
- Will fix affect other components? (Check imports/references)
- Symptom of deeper architectural issue?
- Should pattern be abstracted for reuse?

Don't just fix immediate issue - fix class of issues. Investigate all related components. Complete full investigation cycle before marking done.

---

## Engineering Standards

**Design:** Future scale, implement what's needed today. Separate concerns, abstract at right level. Balance performance, maintainability, cost, security, delivery. Prefer clarity and reversibility.

**DRY & Simplicity:** Don't repeat yourself. Before implementing new features, search for existing similar implementations - leverage and expand existing code instead of creating duplicates. When expanding existing code, trace all dependencies first to ensure changes won't break other things. Keep solutions simple. Avoid over-engineering.

**Complex Visual Features:** For features with significant visual complexity (diagrams, graphs, multi-element layouts), start with the simplest working version and validate with user before elaborating. Visual correctness cannot be verified programmatically - what works algorithmically may look "awful" to users. Build incrementally: simple version → user feedback → iterate. Don't build complex graph algorithms or multi-column layouts without first confirming the basic visual approach is acceptable.

**Standard UI Patterns:** For common features with established industry patterns (directions/timelines → Google Maps, search → Google/Algolia, auth flows → standard OAuth), default to matching the established pattern rather than inventing novel UIs. Users have muscle memory for these patterns; deviation increases friction and iteration cycles. Ask for a reference screenshot if unsure what pattern the user expects.

**Improve in place:** Enhance and optimize existing code. Understand current approach and dependencies. Improve incrementally.

**Performance:** Measure before optimizing. Watch for N+1 queries, memory leaks, unnecessary re-renders. Parallelize safe concurrent operations. Only remove code after verifying truly unused.

**Security:** Build in by default. Validate/sanitize inputs. Use parameterized queries. Hash sensitive data. Follow least privilege.

---

## Configuration & Credentials

**You have access to credentials.** When asked to check external services, find the credentials and use them.

**Where credentials live:**
- `.env` files (workspace or project level) contain API keys and connection strings
- Global config like `~/.config`, `~/.ssh`, or CLI tools might already be configured
- Check what makes sense for what you're looking for

**Common credential patterns:**
- **APIs**: Look for `*_API_KEY`, `*_TOKEN`, `*_SECRET` in .env
- **Databases**: `DATABASE_URL`, `MONGODB_URI`, `POSTGRES_URI` in .env
- **Services**: `SUPABASE_*`, `MTA_*` in .env

**Duplicate configs:** Consolidate immediately. Never maintain parallel configuration systems.

**Before modifying configs:** Understand why current exists. Check dependent systems. Test in isolation. Backup original.

---

## Tool & Command Execution

Use specialized tools for file operations instead of bash commands when possible.

**The core principle:** Bash is for running system commands. File operations have dedicated tools. Don't work around the tools by using sed/awk/echo when you have proper file editing capabilities.

**Why this matters:** File operation tools are transactional and atomic. Bash commands like sed or echo to files can fail partway through, have permission issues, or exhaust resources. The built-in tools prevent these problems.

**Practical habits:**
- Use absolute paths for file operations (avoids "which directory am I in?" confusion)
- Run independent operations in parallel when you can
- Don't use commands that hang indefinitely (tail -f, pm2 logs without limits) - use bounded alternatives or background jobs
- **Batch repetitive operations into scripts** - If you need to run 5+ similar commands (API tests, file checks, etc.), create a temporary script file instead of prompting the user repeatedly. This reduces friction and allows rerunning later.
- **Clean up background processes** - After using dev servers or background jobs, kill them before moving to unrelated tasks. Check for running processes with `lsof -i :PORT` before starting new servers.

---

## Iterative Self-Correction

After each significant change, pause and think:
- Does this accomplish what I intended?
- What else might be affected?
- What could break?
- Test now, not later - run tests and lints immediately
- Fix issues as you find them, before moving forward

Don't wait until completion to discover problems - catch and fix iteratively.

---

## File & Code Organization

### Project Structure

Follow Next.js App Router conventions:
```
/app
  /api          # API routes
  /(routes)     # Page routes
  /components   # Shared components
/lib            # Utilities, helpers, API clients
/types          # TypeScript type definitions
/prisma         # Database schema and migrations
/public         # Static assets
```

### File Management

- Use absolute paths to avoid confusion
- Prefer editing existing files over creating new ones
- Clean up temporary files after use
- Don't create unnecessary documentation files

### Component Organization

- Co-locate related files (component, types, tests)
- Extract reusable logic into custom hooks
- Keep page components thin; delegate to child components

---

## Project-Specific Discovery

Every project has its own patterns, conventions, and tooling. Don't assume general knowledge applies - discover how THIS project works first.

**Look for project-specific rules:** ESLint configs, Prettier settings, testing framework choices, custom build processes. These tell you what the project enforces.

**Study existing patterns:** How do similar features work? What's the component architecture? How are tests written? Follow established patterns rather than inventing new ones.

**Check project configuration:** package.json scripts, framework versions, custom tooling. Don't assume latest patterns work - use what the project actually uses.

General best practices are great, but project-specific requirements override them. Discover first, then apply.

---

## Professional Output

### Communication Style

- No emojis in code, comments, commits, or documentation
- Use precise technical terminology
- Be direct and actionable

### Git Commits

Write clear, descriptive commit messages:
```
Fix authentication timeout in session middleware

- Increase session expiry window from 1h to 24h
- Add token refresh logic for long-running sessions
- Update error messages for expired sessions
```

Not:
```
fix auth
```

### Code Comments

- Comment the "why," not the "what"
- Remove commented-out code
- Keep comments current with code changes
- Use JSDoc for public APIs and complex functions

---

## Testing & Quality

### Verification Standards

Before considering work complete:
- Does it actually work end-to-end?
- Are edge cases handled?
- Are error states properly managed?
- Is the UI responsive and accessible?

### Pre-Commit Checklist

- Code compiles without errors
- No lint warnings
- Types are correct and complete
- Loading and error states implemented
- Works on mobile viewports

### Error Handling

- Display user-friendly error messages
- Log detailed errors for debugging
- Implement proper fallbacks for failed API calls
- Show appropriate empty states when no data

---

## External API Integration

### API Discovery Before Scraping

**When asked to integrate with or scrape a web application, ALWAYS check network requests first.**

Most modern SPAs have underlying REST/GraphQL APIs that are far more reliable than DOM scraping. Before proposing complex scraping solutions:

1. **Open browser DevTools → Network tab**
2. **Interact with the target feature** (e.g., submit a form, load data)
3. **Look for XHR/Fetch requests** returning JSON data
4. **Test the endpoint directly** with curl to verify it's accessible

**Example discovery win:** MTA's trip planner appears to require scraping, but network inspection reveals it uses `otp-mta-prod.camsys-apps.com` - a clean OpenTripPlanner API that's far superior to scraping.

**Benefits of API discovery:**
- Clean, structured data (no HTML parsing)
- Stable endpoints (don't break when UI changes)
- Better performance (no browser automation overhead)
- Easier error handling

### Verify Before You Parse

**NEVER write parsing code, Zod schemas, or data transformations for external APIs without first fetching and inspecting actual response samples.**

This is a critical failure mode: assuming API structures leads to parsers that silently fail or produce incorrect data.

**Required workflow for any external API integration:**
1. **Fetch a real sample** - Use curl, a test script, or the browser to get actual API responses
2. **Inspect the structure** - Check field names (camelCase vs snake_case), nesting, optional fields
3. **Save sample data** - Store in `scripts/sample-*.json` for reference and test fixtures
4. **Then write parsers** - Build Zod schemas and transformations based on verified structures
5. **Test against real data** - Verify parsers work with actual API responses, not just assumptions

**Common API assumption failures:**
- Field naming conventions (snake_case in JSON, camelCase in code)
- Nested vs flat structures
- Array wrappers vs direct objects
- Optional fields that are sometimes missing entirely
- Extension fields in protobuf (e.g., MTA's `transit_realtime.mercury_alert`)

---

## MTA-Specific Guidelines

### GTFS Data Handling

- Parse protobuf feeds using `protobufjs`
- Validate feed responses with Zod schemas
- Handle feed unavailability gracefully
- Cache appropriately to reduce API load

### Real-Time Updates

- Implement proper polling intervals (30-60 seconds for realtime data)
- Show "last updated" timestamps
- Handle stale data indicators
- Graceful degradation when feeds are down

### MTA API Field Conventions

MTA JSON APIs use `snake_case` field names, not camelCase:
- `header_text`, `description_text` (not headerText, descriptionText)
- `active_period`, `informed_entity` (not activePeriod, informedEntity)
- `equipment` field in elevator API (not `equipmentno`)

The alerts API uses a custom extension: `transit_realtime.mercury_alert` containing MTA-specific fields like `alert_type`, `human_readable_active_period`, etc.

### Station & Line Data

- Use consistent station ID references
- Display proper MTA line colors and icons
- Handle express vs local distinctions
- Account for service changes and reroutes

### Train References in Text

MTA alert text contains train references in bracket notation like `[E]`, `[4]`, `[SIR]`. When rendering alert text:

1. **Parse and replace with icons**: Convert `[X]` patterns to `SubwayBullet` components for visual consistency
2. **Apply everywhere text is rendered**: Header text, description text, dashboard cards - not just the "main" location
3. **Handle SI/SIR aliases**: MTA uses "SI" as the route ID but "SIR" in text. Both should render the same Staten Island Railway icon

**When adding text transformation features:**
- Grep the codebase for ALL places the same text type is rendered
- Dashboard cards often mirror detail page content - update both
- Header text and description text may need the same transformation

### Active vs Upcoming Incidents

MTA alerts include future planned work with start dates in the future. When filtering for "active" incidents:

**An incident is "active" if:**
- `activePeriodStart` is null OR `activePeriodStart <= now` (has started)
- AND `activePeriodEnd` is null OR `activePeriodEnd > now` (hasn't ended)

**An incident is "upcoming" if:**
- `activePeriodStart > now` (hasn't started yet)
- Typically only relevant for PLANNED_WORK, REDUCED_SERVICE, SERVICE_CHANGE types

**Common mistake:** Filtering only by `activePeriodEnd > now` will include future planned work as "active".

### Multi-Complex Stations (Critical)

**Some station names map to multiple GTFS station IDs.** Times Sq-42 St has FOUR separate station complexes (127, 725, 902, A27), each serving different lines.

**When working with stations:**
1. Search returns deduplicated results by name
2. Each result includes `allIds` (all complex IDs) and `allPlatforms` (all platform IDs)
3. To get ALL trains at a station, fetch arrivals for ALL platforms, not just one
4. Deduplicate arrivals by `tripId` to avoid showing the same train multiple times

**Example:** To show all trains at Times Sq, fetch from `127N`, `127S`, `725N`, `725S`, `902N`, `902S`, `A27N`, `A27S` - not just `127N` and `127S`.

---

## Performance Considerations

### Frontend

- Use React Server Components for initial data
- Implement proper loading skeletons
- Lazy load non-critical components
- Optimize images and assets

### Backend

- Batch database queries where possible
- Implement proper caching strategies
- Use connection pooling
- Monitor and log slow queries

### API Design

- Keep responses focused and minimal
- Use proper HTTP caching headers
- Implement pagination for list endpoints
- Version APIs when making breaking changes

---

## React Patterns & State Management

### Async Functions Should Be Self-Contained

**Don't depend on React state that may not be set when an async function runs.**

Race conditions occur when an async effect sets state, and another effect that depends on that state runs before it's populated.

**BAD:**
```tsx
const [config, setConfig] = useState(null);

useEffect(() => {
  fetchConfig().then(setConfig); // Sets config asynchronously
}, []);

const doWork = useCallback(async () => {
  // BUG: config may be null if doWork runs before fetchConfig resolves
  const result = await process(config);
}, [config]);

useEffect(() => {
  doWork(); // Runs immediately, before config is set
}, [doWork]);
```

**GOOD:**
```tsx
const doWork = useCallback(async () => {
  // Self-contained: fetch what you need, don't depend on state
  const config = await fetchConfig();
  const result = await process(config);
}, []);
```

### Test the Full User Journey

Don't just test the happy path. Verify behavior across all entry points:
- **Direct navigation** (user types URL)
- **Internal navigation** (link from another page)
- **Hard refresh** (Cmd+Shift+R)
- **Load from saved state** (localStorage, URL params)
- **Back/forward navigation**

If data loads correctly via search but not on refresh, you have a state initialization bug.

---

## Security

- Validate and sanitize all user inputs
- Use parameterized database queries
- Implement proper authentication for personalized features
- Never expose API keys or secrets in client code
- Use environment variables for configuration

---

## Accessibility

- Use semantic HTML elements
- Ensure proper color contrast
- Support keyboard navigation
- Add ARIA labels where needed
- Test with screen readers for critical flows

---

## Documentation Standards

### Documentation Location
All documentation lives in `/docs`:
- `README.md` - Documentation index
- `setup.md` - Development environment setup
- `architecture.md` - System design and folder structure
- `api.md` - API endpoint reference
- `components.md` - Component usage guide
- `testing.md` - Testing guide
- `contributing.md` - Contribution guidelines
- `user-guide.md` - End-user documentation

### Component Documentation
- **Storybook** - Interactive component playground at `npm run storybook`
- **JSDoc** - Document props and complex functions
- **Stories** - Create `.stories.tsx` files for visual components

### Documentation Rules
- Keep documentation in sync with code changes
- Use Markdown for all docs
- Include code examples where helpful
- Update docs in the same PR as code changes

---

## Testing Standards

### Testing Stack
- **Vitest** - Unit and component tests
- **React Testing Library** - Component testing utilities
- **Playwright** - End-to-end tests

### Test Commands
```bash
npm run test           # Run unit/component tests
npm run test:watch     # Watch mode
npm run test:coverage  # With coverage report
npm run test:e2e       # Run E2E tests
npm run test:e2e:ui    # E2E with visual UI
```

### Test File Organization
```
tests/
├── setup/           # Test configuration
├── unit/            # Unit tests for utilities
├── components/      # Component tests
└── e2e/             # End-to-end tests
```

### Testing Guidelines
- Write tests for new features and bug fixes
- Test behavior, not implementation
- Use descriptive test names
- Follow Arrange-Act-Assert pattern
- Aim for 70% coverage minimum
- Co-locate component tests when appropriate

---

## Common Configuration Gotchas

### TypeScript Build Exclusions
Always exclude test and config files from production TypeScript compilation to prevent type errors from test-only dependencies:
```json
// tsconfig.json
"exclude": [
  "node_modules",
  "playwright.config.ts",
  "vitest.config.ts",
  "tests/**/*",
  "stories/**/*"
]
```

### JSX in Non-Component Files
Any file containing JSX (including test setup files with mock components) must use the `.tsx` extension, not `.ts`. This applies to:
- `vitest.setup.tsx` (if mocking components)
- Storybook decorators
- Test utility files

### Framework-Specific Imports
Use the correct framework-specific imports to avoid runtime and lint errors:
```typescript
// Storybook - use your framework package, not the generic renderer
import type { Meta, StoryObj } from '@storybook/nextjs-vite';  // Correct
import type { Meta, StoryObj } from '@storybook/react';         // Wrong

// Preview files with JSX need .tsx extension
// .storybook/preview.tsx (not .ts)
```

### When Configuration Fails
If a library configuration fails after 2-3 attempts:
1. Stop and acknowledge the gap
2. Ask the user for help or to search for the solution
3. Document the working solution once found

This is more efficient than continuing to guess. Configuration issues often require specific knowledge not in general docs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abirh2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
