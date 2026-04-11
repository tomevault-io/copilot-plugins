## emberdocs

> **IMPORTANT:** This file, `claude.md`, and `.cursorrules` must stay in sync. See "Sync Instructions" at bottom.

# Contributing Guidelines — EmberDocs

**IMPORTANT:** This file, `claude.md`, and `.cursorrules` must stay in sync. See "Sync Instructions" at bottom.

**For universal development standards applicable to any project, see:** `dev-docs/guides/DEVELOPMENT-STANDARDS.md`

---

## Universal Standards (Apply to All Projects)

See `dev-docs/guides/DEVELOPMENT-STANDARDS.md` for comprehensive details on:
- **Code Style:** TypeScript strict, Prettier, ESLint, naming conventions
- **Testing:** Unit/integration/E2E pyramid, Jest, coverage targets ≥70%
- **Git & Version Control:** Conventional Commits, branch strategy, semantic versioning
- **Pull Requests:** Requirements (tests, docs, scope), code review checklist
- **Security:** Input validation, secrets management, dependency auditing
- **Accessibility:** WCAG 2.1 Level AA minimum
- **Documentation:** Progress logs, architecture decisions, API docs
- **License Compliance:** Automated license checking, NOTICES.md maintenance, dependency license verification
- **CI/CD:** Standard build pipeline, GitHub Actions, deployment strategy

---

## Contributing Workflow (EmberDocs)

### 1. Getting Started

```bash
# Clone repository
git clone https://github.com/sturdy-barnacle/emberdocs.git
cd emberdocs

# Install dependencies
npm install

# Create .env.local from template
cp .env.example .env.local

# Run dev server
npm run dev
# Open http://localhost:3000 in browser
```

**First-time setup checklist:**
- [ ] Repo cloned and dependencies installed
- [ ] Dev server running at http://localhost:3000
- [ ] All tests pass: `npm run check`
- [ ] Read `dev-docs/Quick-Reference.md` (5 min)
- [ ] Reviewed Phase 01 deliverables in `dev-docs/planning/mvp_phase01of02.md`
- [ ] Checked locked decisions in `dev-docs/guides/ARCHITECTURE-DECISIONS.md`
- [ ] Configured Git user: `git config user.name "Your Name"`

---

### 2. Create a Feature Branch

```bash
# Update main branch
git fetch origin
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/search-indexing      # For new features
git checkout -b fix/parser-crash              # For bug fixes
git checkout -b chore/update-dependencies     # For chores/docs
```

**Branch naming rules:**
- `feature/` — New feature or enhancement
- `fix/` — Bug fix
- `chore/` — Documentation, dependencies, config changes
- Use kebab-case: `feature/add-dark-mode`, not `feature/addDarkMode`

---

### 3. Make Changes

**While developing:**

```bash
# Run dev server in background
npm run dev &

# In another terminal, run tests in watch mode
npm test -- --watch

# Check code quality
npm run lint

# Format code
npm run format

# Type check
npm run typecheck
```

**Code organization:**
- Keep changes focused (one feature per branch)
- Put code in `src/app/` or `src/lib/`
- Create tests in `__tests__/` or as `*.test.ts` files
- Update relevant docs (progress, architecture, user guides)

**Write tests as you code:**
```bash
# Example: Testing a parser
npm test -- src/lib/parse-markdown.test.ts --watch
```

---

### 4. Commit Your Changes

**Before committing, run:**
```bash
npm run check    # Full CI suite (lint → typecheck → test → build)
```

**Commit with clear messages (Conventional Commits):**

```bash
git commit -m "feat(parser): add YAML frontmatter extraction

- Parse YAML header from markdown files
- Validate frontmatter against schema
- Extract body content after frontmatter

Closes #42"
```

**Commit message format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:** `feat` (new feature), `fix` (bug fix), `docs` (documentation), `test` (tests), `refactor` (code restructuring), `chore` (dependencies, config)

**Good commits:**
- ✅ `feat(parser): add YAML frontmatter extraction`
- ✅ `fix(nav): handle missing sidebar items gracefully`
- ✅ `docs: add development standards guide`
- ❌ `update stuff`
- ❌ `WIP: trying something`

**Imperative tense:**
- ✅ "Add feature" (not "Added feature")
- ✅ "Fix bug" (not "Fixed bug")

---

### License Compliance

**Before adding new dependencies:**
1. Check the dependency's license
2. Ensure the license is allowed by our dependency allowlist (run `npm run license-check`)
3. Prefer permissive licenses (MIT, Apache-2.0, BSD) and avoid strong copyleft (GPL, AGPL) in the core framework
4. Update NOTICES.md when adding dependencies

**Automated checking:**
- `npm run license-check` - Verify all dependency licenses
- Integrated into `npm run check` command
- Fails build if incompatible licenses detected

**License checking workflow:**
```bash
# Check licenses before committing
npm run license-check

# Full check (includes license check)
npm run check
```

---

### 5. Create a Pull Request

**Before submitting, verify:**
```bash
npm run check                    # All tests pass
git status                       # No uncommitted changes
git log --oneline origin/main..  # Review commits
```

**Push your branch:**
```bash
git push origin feature/search-indexing
```

**Create PR on GitHub using template in `.github/PULL_REQUEST_TEMPLATE.md`**

**PR Requirements Checklist:**

```markdown
## Summary
Describe what changed and why in 1–3 sentences.

## Testing
- [x] `npm run lint` — passed
- [x] `npm run typecheck` — passed
- [x] `npm run test` — passed
- [x] `npm run build` — passed
- [x] Manual testing in browser

## Documentation
- [x] Progress log created: `dev-docs/progress/YYYY_MM_DD.md`
- [x] CHANGELOG.md updated under `[Unreleased]`
- [x] User docs updated (if UI changed)
- [x] Developer docs updated (if architecture changed)

## Related Issues
- Fixes #42
- Relates to #51
```

---

### 6. Code Review

**What reviewers will check:**
- [ ] Code follows style guide (TypeScript, naming, structure)
- [ ] Tests are meaningful and pass locally
- [ ] No security issues (input validation, secrets, dependencies)
- [ ] Documentation is clear and linked
- [ ] Commit messages are clear
- [ ] Scope is focused (one feature per PR)
- [ ] No console.log() or debug code left in

**Respond to feedback:**
```bash
# Make requested changes
# Commit with clear message
git commit -m "refactor(parser): improve error messages per review"

# Push changes
git push origin feature/search-indexing

# Do NOT force push; let reviewers see the changes
```

---

### 7. Merge to main

**Once approved:**
```bash
# Squash commits (optional, team preference)
# or merge with all commits preserved

# GitHub will merge the PR
```

**After merge:**
```bash
# Update your local main
git fetch origin
git checkout main
git pull origin main

# Delete local branch
git branch -d feature/search-indexing
```

---

## Documentation Requirements (Per PR)

### Progress Log (Required)

**File:** `dev-docs/progress/YYYY_MM_DD.md` (e.g., `dev-docs/progress/2025_12_24.md`)

**Template:**
```markdown
# Progress: 2025-12-24

## Summary
- ✅ Implemented D1.1 (content pipeline)
- ✅ Added 10 unit tests

## Work Done
- Markdown parsing with frontmatter extraction
- Table of contents generation
- Error handling for malformed YAML

## Blockers
- None

## Next Steps
- Implement D1.2 (navigation generator)
- Code review and merge to main
```

### Changelog Update (Required)

**File:** `CHANGELOG.md` (under `[Unreleased]` section)

**Format:**
```markdown
### Added
- Content pipeline module with Markdown/MDX parsing (#42)
- YAML frontmatter extraction and validation
- Automatic TOC generation from headings

### Fixed
- Parser crash on empty frontmatter

### Changed
- Improved error messages for malformed markdown
```

### Architecture Decision (If applicable)

**File:** `dev-docs/guides/ARCHITECTURE-DECISIONS.md`

If you made a major architectural decision, create an ADL entry:

```markdown
## ADL-XXX: [Decision Title]

**Date:** 2025-12-24
**Status:** Accepted
**Locked:** Phase 01/Phase 02

### Context
Why does this decision matter? What problem are we solving?

### Decision
What did we decide to do?

### Alternatives Considered
- Alternative 1: pros/cons
- Alternative 2: pros/cons

### Consequences
**Positive:**
- Benefit 1
- Benefit 2

**Negative:**
- Cost 1
- Cost 2

### Related Decisions
- ADL-001: [Related decision]
- ADL-008: [Related decision]

### Links
- [Reference 1](url)
```

---

## Code Review Standards

### As a Reviewer

**Check these items:**

```
Code Quality
- [ ] Code follows naming conventions (PascalCase, camelCase, kebab-case)
- [ ] TypeScript types are explicit (no `any`)
- [ ] Functions have return type annotations
- [ ] No code duplication (DRY principle)
- [ ] Error handling is explicit (no silent failures)
- [ ] Comments explain "why," not "what"

Testing
- [ ] Tests are present and meaningful
- [ ] Test names clearly describe what's tested
- [ ] Edge cases are covered
- [ ] Coverage targets are met (≥70%)

Documentation
- [ ] Progress log created with clear summary
- [ ] CHANGELOG.md updated
- [ ] User/developer docs updated (if applicable)
- [ ] ADL entry created (if major decision)
- [ ] Commit messages are clear and imperative

Security
- [ ] No hardcoded secrets (use .env.local)
- [ ] Input is validated/sanitized
- [ ] No dangerous operations (e.g., eval)
- [ ] Dependencies are checked for vulnerabilities

Git
- [ ] Branch naming follows convention (feature/, fix/, chore/)
- [ ] Commits use Conventional Commits format
- [ ] PR scope is focused (one feature per PR)
- [ ] No merge conflicts or unresolved comments
```

**Provide constructive feedback:**
```
❌ Bad: "This is wrong"
✅ Good: "Consider using parseMarkdown() from src/lib/content.ts instead of duplicating the logic here. It already handles frontmatter extraction."

❌ Bad: "Add tests"
✅ Good: "Missing test for the edge case where YAML is malformed. Can you add a test similar to the one in parse-markdown.test.ts that covers invalid YAML?"
```

---

## GitHub Workflow

### Labels (Optional)

- `bug` — Issue or PR fixing a bug
- `enhancement` — New feature or improvement
- `documentation` — Documentation changes
- `Phase-01`, `Phase-02` — Which phase this affects
- `blocked` — Blocked by another PR or issue
- `ready-to-merge` — Approved and ready to merge

### Issues

**Creating an issue:**
1. Use clear, descriptive title: "Fix parser crash on empty frontmatter"
2. Provide context: Why is this an issue? How do you reproduce it?
3. Link to related code/docs

**Closing issues:**
```bash
git commit -m "fix(parser): handle empty frontmatter gracefully

Closes #42"
```

### Discussions

Use GitHub Discussions for:
- Questions about the codebase
- Design discussions before creating a PR
- Roadmap feedback
- General architecture questions

---

## Documentation Maintenance

**IMPORTANT:** Documentation must be kept up to date and well-organized. This is a requirement for all contributions.

### Documentation Organization

**Naming Conventions:**
- **Developer docs** (`dev-docs/`): ALL CAPS with hyphens for technical docs, Title Case for user guides
  - Technical examples: `ARCHITECTURE-DECISIONS.md` (in `dev-docs/guides/`), `EMBERDOCS-TECHNICAL-SPEC.md` (in `dev-docs/specs/`)
  - User guide examples: `Setup.md`, `Deployment.md`, `Quick-Reference.md` (in `dev-docs/`)
- **Example docs** (`docs/examples/`): kebab-case
  - Examples: `getting-started.md`, `api-reference.md`

**Directory Structure:**
- `dev-docs/specs/` - Technical specifications (EMBERDOCS-TECHNICAL-SPEC.md, EMBERDOCS-API-SPEC.md, etc.)
- `dev-docs/guides/` - Development guides (ARCHITECTURE-DECISIONS.md, DEVELOPMENT-STANDARDS.md, etc.)
- `dev-docs/planning/` - Phase plans and planning documents
- `dev-docs/progress/` - Daily progress logs (format: `YYYY_MM_DD.md` or `YYYY_MM_DD_description.md`)
- `dev-docs/` (root) - User guides and project overview docs (Setup.md, Deployment.md, Quick-Reference.md, PROJECT-OVERVIEW.md, etc.)
- `docs/examples/` - Example documentation files (user-facing sample docs)

**Reference Documentation:**
- `DOCUMENTATION.md` at repository root provides complete documentation index
- Always update `DOCUMENTATION.md` when adding, moving, or renaming documentation files

### When to Update Documentation

**Always update documentation when:**
- Adding new features or changing existing functionality
- Making architectural decisions (create ADL entry in `dev-docs/guides/ARCHITECTURE-DECISIONS.md`)
- Changing setup or deployment processes
- Updating dependencies or tools
- Completing deliverables (update progress logs in `dev-docs/progress/`)
- Fixing bugs that affect user workflows

**Specific documentation to update:**
- **Technical Specs** (`dev-docs/specs/`): When architecture, API, or system design changes
- **Architecture Decisions** (`dev-docs/guides/ARCHITECTURE-DECISIONS.md`): When making major technical decisions
- **Progress Logs** (`dev-docs/progress/`): Daily (end of work day) - required for all work sessions
- **User Docs** (`dev-docs/`): When features change or new setup steps added
- **README.md**: When project overview, quick start, or key features change
- **DOCUMENTATION.md**: When documentation structure changes (new files, moved files, renamed files)

### Documentation Quality Checklist

**Before committing documentation changes:**
- [ ] All file paths and references are accurate (use relative paths, not absolute)
- [ ] Cross-references between documents are updated
- [ ] Naming conventions are followed (ALL CAPS for dev docs, Title Case for user docs)
- [ ] `DOCUMENTATION.md` is updated if structure changed
- [ ] Progress log created if this is a work session (`dev-docs/progress/YYYY_MM_DD.md`)
- [ ] No broken links (verify with `grep` or link checker)
- [ ] Code examples are tested and accurate
- [ ] Screenshots/mockups are up to date (if applicable)

### Cross-Reference Maintenance

**Always maintain accurate cross-references:**
- Use relative paths: `dev-docs/specs/EMBERDOCS-TECHNICAL-SPEC.md` (not absolute paths)
- When moving files, update ALL references across the codebase
- Use `grep` to find all references before moving/renaming files:
  ```bash
  grep -r "old-filename" . --include="*.md"
  ```
- Update references in: README.md, DOCUMENTATION.md, planning docs, progress logs, and internal cross-references

### Documentation Review Process

**During code review, reviewers verify:**
- [ ] Documentation changes match code changes
- [ ] New features are documented
- [ ] Breaking changes are clearly marked
- [ ] Examples and code snippets are accurate
- [ ] Links and references work
- [ ] Progress log is created/updated

**Before merging PR:**
- [ ] All documentation references are valid
- [ ] `DOCUMENTATION.md` reflects current structure
- [ ] No orphaned documentation files
- [ ] Documentation follows naming conventions

### Documentation as Part of PR Requirements

**Every PR must include:**
1. **Progress Log** - Create `dev-docs/progress/YYYY_MM_DD.md` with work summary
2. **Changelog** - Update `CHANGELOG.md` under `[Unreleased]` section
3. **Documentation Updates** - Update relevant docs (dev-docs/ if UI or architecture changed)
4. **DOCUMENTATION.md** - Update if documentation structure changed

**See also:** Documentation Requirements section above for detailed templates.

---

## Sync Instructions

**These three files must always stay in sync:**
1. **.cursorrules** — Concise rules for AI assistants
2. **claude.md** — Detailed guidelines for humans
3. **AGENTS.md** (this file) — Contributing process and standards

**When you update standards:**
- [ ] Update all three files with consistent changes
- [ ] Keep "Universal Standards" sections identical
- [ ] Update sync checklist in each file
- [ ] Include reference to DEVELOPMENT-STANDARDS.md

**Changes requiring sync:**
- New coding style rule
- New tool added (linter, test framework, etc.)
- New documentation requirement
- Architecture decision locked (also create ADL entry)
- Phase plan updated
- Git workflow changed
- Documentation structure/organization changed (also update DOCUMENTATION.md)

---

## Quick Links

- **Phase Plans:** `dev-docs/planning/mvp_phase01of02.md`, `dev-docs/planning/mvp_phase02of02.md`
- **Architecture Decisions:** `dev-docs/guides/ARCHITECTURE-DECISIONS.md`
- **Developer Cheat Sheet:** `dev-docs/Quick-Reference.md`
- **Development Standards:** `dev-docs/guides/DEVELOPMENT-STANDARDS.md`
- **Setup Guide:** `dev-docs/guides/DEV-SETUP-VERIFICATION.md`
- **Feature Dependencies:** `dev-docs/guides/FEATURE-DEPENDENCIES.md`
- **Changelog:** `CHANGELOG.md`
- **PR Template:** `.github/PULL_REQUEST_TEMPLATE.md`

---

## Getting Help

- **Code Questions:** Check `dev-docs/Quick-Reference.md`
- **Architecture Questions:** See `dev-docs/guides/ARCHITECTURE-DECISIONS.md`
- **Standards Questions:** Read `dev-docs/guides/DEVELOPMENT-STANDARDS.md` or `claude.md`
- **Setup Issues:** Follow `dev-docs/guides/DEV-SETUP-VERIFICATION.md`
- **GitHub Issues:** Open an issue for bugs or feature requests
- **GitHub Discussions:** Ask questions or discuss design
- **Project Board:** View progress and blockers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sturdy-barnacle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sturdy-barnacle)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
