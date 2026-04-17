## ravendarque-beyond-borders

> Beyond Borders is a web application that allows users to add circular, flag-colored borders to their profile pictures to show support for marginalized groups and selected causes. Built with React, TypeScript, Vite, and Canvas 2D.

# Cursor AI Assistant Rules for Beyond Borders

## Project Overview
Beyond Borders is a web application that allows users to add circular, flag-colored borders to their profile pictures to show support for marginalized groups and selected causes. Built with React, TypeScript, Vite, and Canvas 2D.

## Critical Workflow Rules

### Issue-First Development
**CRITICAL: When user mentions an existing issue number (e.g., "pick up issue 103", "work on #42"), NEVER create a new issue. Instead:**
1. Fetch the existing issue details using `gh issue view <number>`
2. Update its status to "InProgress" if starting work
3. Use the github-helper.ps1 script for ALL GitHub operations

**ALWAYS use the github-helper.ps1 script for GitHub operations:**
```powershell
# View an issue
gh issue view <number>

# Update issue status
.\.github\scripts\github-helper.ps1 issue-update -Number <number> -Status InProgress

# Create NEW issue (only when explicitly asked to create one)
.\.github\scripts\github-helper.ps1 issue-create `
  -Title "Your task title" `
  -BodyFile ".local/issue.md" `
  -Priority P0|P1|P2 `
  -Size XS|S|M|L|XL `
  -Status InProgress
```

**Priority Levels:**
- **P0 (Critical)**: Security issues, breaking bugs, WCAG compliance issues
- **P1 (High)**: Important features, UX improvements, performance issues
- **P2 (Low)**: Nice-to-have features, refactoring, documentation

**Size Estimates:**
- **XS**: < 1 hour
- **S**: 1-2 hours
- **M**: 2-4 hours
- **L**: 4-8 hours
- **XL**: 8+ hours

**Status Values:**
- **Backlog**: Not yet started
- **Ready**: Ready to be picked up
- **InProgress**: Currently being worked on
- **InReview**: In code review or testing
- **Done**: Completed and merged (auto-closed by PR merge)

**CRITICAL RULE:** Do **NOT** move an issue to `InReview` until after the PR is created. Finishing local commits without a PR still counts as `InProgress`.

**Never:**
- Don't manually close issues (let PR merge do it)
- Don't set status to `Done` manually
- Don't set `InReview` before a PR exists
- Don't leave issues in `InProgress` once PR is open

### Issue Structure Template
When creating issues, use this structure:
```markdown
## Goal
[Clear, concise description of what needs to be accomplished]

## Tasks
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

## Context
[Why this work is needed, background information]

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Technical Notes (optional)
[Any relevant technical details, dependencies, or considerations]
```

## File Organization

### Local Working Files
**Always use the `.local/` directory** for temporary and local development files. Never put them in the repo root or docs/ unless they are committed project docs.
- Temporary scripts and PowerShell files
- Test outputs and screenshots
- Work-in-progress data files
- Personal notes and todos
- **Changelogs, release notes, and other draft docs for public consumption** (e.g. `.local/CHANGELOG-1.4.1-1.5.3.md`)
- Any files that shouldn't be committed

**Example:** `.local/temp-script.ps1`, `.local/screenshots/preview.png`, `.local/notes.md`, `.local/CHANGELOG-*.md`

The entire `.local/` directory is gitignored (except `.local/README.md`).

## Code Standards

### TypeScript
- **Always use strict TypeScript** with no `any` types unless absolutely necessary
- Use explicit return types for all functions
- Prefer interfaces over types for object shapes
- Use type guards and discriminated unions for type safety
- All props and state should be properly typed

### React
- **Use functional components** with hooks only
- Use `React.memo()` for expensive components
- Keep components small and focused (single responsibility)
- Prefer composition over inheritance
- Use custom hooks to encapsulate reusable logic
- Always handle loading and error states

### Error Handling
- Use custom error classes (`RenderError`, `FlagDataError`, `FileValidationError`)
- Implement error boundaries for graceful degradation
- Provide user-friendly error messages
- Log errors appropriately for debugging

### Performance
- Use `useMemo` and `useCallback` appropriately (not everywhere)
- Implement debouncing for expensive operations
- Use `requestIdleCallback` for non-critical background tasks
- Optimize Canvas operations (use OffscreenCanvas when available)
- Implement image preloading for frequently used assets

### Accessibility (WCAG AA Compliance)
- All interactive elements must be keyboard accessible
- Provide ARIA labels and descriptions where needed
- Ensure proper heading hierarchy (h1 → h2 → h3)
- Maintain color contrast ratios (4.5:1 for normal text, 3:1 for large text)
- Support screen readers with semantic HTML
- Test with keyboard navigation and screen readers
- Include focus indicators for all interactive elements

### Testing
- **Write tests for all new features** before considering them complete
- Unit tests: Use Vitest with React Testing Library
- E2E tests: Use Playwright for critical user flows
- Aim for meaningful coverage, not just high percentages
- Test error states and edge cases
- Mock external dependencies appropriately

Test file locations:
- Unit tests: `test/unit/`
- Integration tests: `test/integration/`
- E2E tests: `test/e2e/`
- Test fixtures: `test/fixtures/`
- Test data: `test/test-data/`

### Code Organization
- Keep files focused and under 300 lines when possible
- Use clear, descriptive file and folder names
- Separate concerns: components, hooks, utils, types
- Export only what's needed (prefer named exports)
- Group related functionality together

Project structure:
```
src/
├── components/     # React components
├── hooks/          # Custom React hooks
├── renderer/       # Canvas rendering logic
├── flags/          # Flag data and utilities
├── utils/          # Shared utilities
├── types/          # TypeScript type definitions
└── styles/         # Global styles (Tailwind)

test/
├── unit/           # Unit tests (Vitest)
├── integration/    # Integration tests (Vitest)
├── e2e/            # E2E tests (Playwright)
├── fixtures/       # Test fixtures (images, etc.)
├── test-data/      # Test data files
└── setup.ts        # Test setup configuration
```

## Git Commit Standards

### Commit Message Format
Use conventional commits format:
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring (no functional changes)
- `test`: Adding or updating tests
- `docs`: Documentation changes
- `style`: Code style changes (formatting, semicolons, etc.)
- `perf`: Performance improvements
- `chore`: Build process or auxiliary tool changes
- `ci`: CI/CD configuration changes

**Examples:**
```
feat(renderer): Add predictive flag preloading

- Created useFlagPreloader hook for idle-time preloading
- Preloads 7 priority flags using requestIdleCallback
- Cache-aware (skips current flag and cached images)

Performance: ~80% cache hit for popular flags, zero CPU impact
```

```
fix(accessibility): Add keyboard navigation to flag selector

- Implemented arrow key navigation
- Added proper ARIA labels
- Fixed focus trap in modal

Closes #42
```

### Branch Naming
- Feature branches: `feature/short-description`
- Bug fixes: `fix/short-description`
- Hotfixes: `hotfix/short-description`

### Pull Request Process
1. Ensure all tests pass locally
2. Update documentation if needed
3. Reference related issues in PR description
4. Request review from team members
5. Address review comments promptly
6. Squash commits before merging (if multiple WIP commits)

## CI/CD

### GitHub Actions Workflows
The project uses CI/CD for:
- **Flag validation**: Ensures all referenced SVG flags exist in `public/flags/`
- **Build verification**: Checks that the app builds successfully
- **Test execution**: Runs unit and E2E tests
- **Type checking**: Validates TypeScript types

Run locally before pushing:
```powershell
# Validate flags
node scripts/validate-flags.js

# Run tests
pnpm test

# Build
pnpm build

# Type check
pnpm type-check
```

## Flag Management

### Adding New Flags
1. Add flag metadata to `data/flag-data.yaml`
2. Run `node scripts/fetch-flags.js` to generate assets and TypeScript
3. Run validation: `node scripts/validate-flags.js`
4. Test the flag in the UI

### Flag Data Validation
- All flags must have valid Zod schemas
- SVG files must exist in `public/flags/`
- Flag IDs must be unique
- Categories come from YAML category blocks; new category names are slugified (no fixed enum)

## Project-Specific Guidelines

### Canvas Rendering
- Always clean up canvas contexts
- Use `OffscreenCanvas` when available for better performance
- Handle canvas errors gracefully
- Optimize drawing operations (batch when possible)
- Use `requestAnimationFrame` for animations

### Image Processing
- Validate file types and sizes
- Handle EXIF orientation
- Provide progress feedback for large images
- Implement proper error messages for invalid files
- Consider memory constraints for large images

### State Management
- Use React Context for global state (minimal)
- Prefer local state when possible
- Use reducers for complex state logic
- Keep state as close to usage as possible
- Avoid prop drilling (use composition or context)

### Performance Targets
- First Contentful Paint: < 1.5s
- Time to Interactive: < 3.5s
- Lighthouse Performance Score: > 90
- Bundle size: < 500KB (gzipped)
- Image processing: < 2s for typical images

## Common Tasks

### Starting Work on a New Task
1. Create and track issue using github-helper.ps1
2. Create feature branch: `git checkout -b feature/task-name`
3. Make changes following code standards
4. Write tests for new functionality
5. Update issue status and check off completed tasks
6. Commit with conventional commit message
7. Push and create pull request
8. Update issue status to "InReview"
9. After merge, issue auto-closes to "Done"

### Running the Development Server
```powershell
pnpm install   # Install dependencies
pnpm dev       # Start dev server at http://localhost:5173
pnpm test      # Run tests
pnpm build     # Create production build
pnpm preview   # Preview production build
```

### Debugging
- Use browser DevTools for frontend debugging
- Check console for errors and warnings
- Use React DevTools for component inspection
- Use Vite's HMR for fast iteration
- Check network tab for asset loading issues

## Resources
- **PRD**: `beyond-borders-prd-0.1.md`
- **Project Configuration**: `.github/project.json`
- **Project Fields**: `.github/project-fields.json`
- **GitHub Helper Script**: `.github/scripts/github-helper.ps1`
- **Copilot Instructions**: `.github/copilot-instructions.md`

## PowerShell Core Standard
- All validation and setup scripts use PowerShell Core (pwsh) only
- No bash scripts - everything is cross-platform PowerShell
- Use `pwsh` command for all PowerShell operations
- Scripts are in `.github/scripts/` directory

---
**Last Updated**: Based on copilot-instructions.md (November 2, 2025)
**Maintained By**: Development Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ravendarque) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
