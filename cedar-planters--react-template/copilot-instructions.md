## react-template

> - 95%+ confidence required before making changes. When unsure, investigate first.

# Global Rules

## Engineering Standards (Staff/Principal Level)

### Decision Making
- 95%+ confidence required before making changes. When unsure, investigate first.
- Always verify against official documentation. Do not guess.
- Double check changes won't break existing functionality.

### Principles
- **SOLID**: Apply pragmatically, not dogmatically.
- **KISS**: Simplest solution that works. No premature abstraction.
- **DRY**: Extract at 3+ repetitions only. Premature DRY is worse than repetition.
- **YAGNI**: Don't build for hypothetical future requirements.

### Code Quality
- Meaningful names, small functions, no dead code, no commented-out code.
- Fail fast: validate at boundaries, return early, max 3 levels nesting.
- Immutability by default. Mutate only when necessary.
- Changes must be reviewable in under 15 minutes. Split large changes.

### Testing
- Unit tests for logic, integration tests for boundaries, skip trivial code.
- Run tests after every change. If no test suite exists, verify manually.

### Security
- Never commit secrets or .env files.
- Validate all user input at system boundaries.
- Use parameterized queries, never string concatenation for SQL.
- Keep dependencies updated.

### Git
- Single-line commit messages: `type(scope): description`
- Types: feat, fix, docs, style, refactor, test, chore
- No co-author references. No emojis.

### Documentation
- No markdown files without explicit approval.
- Comment only when logic is not obvious.
- Public API functions should have docstrings.

## React / Vite Best Practices

### Components
- Functional components with hooks only (no class components).
- Keep components small and focused (< 150 lines).
- Use `useCallback` and `useMemo` for expensive operations.
- Use Context API for global state. Local state for component state.
- Lazy load routes with `React.lazy`.

### Styling
- Tailwind CSS exclusively. No inline styles. No CSS files.
- Mobile-first responsive design.
- Use `clsx` for conditional classes.

### API Integration
- All API calls centralized in `services/api.js`.
- Axios with interceptors for auth token injection.
- Handle loading and error states on every async operation.
- Cancel requests on component unmount.

### Build
- Vite with code splitting and tree shaking.
- Production builds via `vite build`.
- Nginx for serving static files in production.
- Target ES2015+ for modern browsers.

### Style
- Prettier for formatting (2-space indent).
- ESLint with react-hooks plugin.
- One component per file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cedar-planters) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
