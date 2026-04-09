## agentic-applications

> - ALWAYS use `uv run python` instead of running Python directly

# Python Execution Rule

## Use uv run python
- ALWAYS use `uv run python` instead of running Python directly
- This ensures proper dependency management and virtual environment handling
- Replace `python`, `python3`, or `py` commands with `uv run python`

## Examples
```bash
# Good
uv run python scripts/prisma_to_python.py
uv run python -m pytest
uv run python manage.py migrate

# Avoid
python scripts/prisma_to_python.py
python3 scripts/prisma_to_python.py
py scripts/prisma_to_python.py
```

---

# Database Schema Rules

## Source of Truth
- `apps/web-app/prisma/schema.prisma` is the ONLY source of truth for database schema
- NEVER modify Python models in backend directly
- ALWAYS update schema in Prisma first, then regenerate Python models

## Schema Changes Workflow
1. Make changes to `apps/web-app/prisma/schema.prisma`
2. Run: `uv run python scripts/prisma_to_python.py apps/web-app/prisma/schema.prisma`
3. Review generated files in `apps/web-app/prisma/generated/`
4. Copy generated files to backend project

## Generated Files (READ-ONLY)
- `apps/web-app/prisma/generated/models.py` - SQLAlchemy models
- `apps/web-app/prisma/generated/schemas.py` - Pydantic schemas
- Any files in backend derived from these generators

## Commands
- Regenerate models: `uv run python scripts/prisma_to_python.py apps/web-app/prisma/schema.prisma`

---

# Git Commit Convention

## Format
<type>(<scope>): <subject>

<body>

<footer>

## Rules
- Subject: Max 50 chars, imperative mood ("add" not "added"), no period
- Body: Wrap at 72 chars, explain what/why not how, blank line after subject
- Breaking changes: Add ! after type or BREAKING CHANGE: in footer

## Types
feat fix docs style refactor perf test build ci chore revert

## Scopes (Project-Specific)
web extension api ui types db auth editor resume template config

## Examples
```bash
# Good
feat(web): add template gallery
fix(editor): prevent block deletion on backspace
perf(api): implement response caching

# Good with body
feat(web): add resume template gallery

Implement browsing interface with:
- Grid layout with previews
- Category filtering
- Search with debouncing

Closes #123

# Breaking change
feat(api)!: change response format

BREAKING CHANGE: 'data' field renamed to 'content'
```

## Quick Reference
- Atomic commits (one logical change)
- Reference issues (#123)
- Present tense, imperative mood
- Explain what and why, not how




# Shadcn Component Styling Rule

When generating code that uses shadcn/ui components, Claude should rely on the components' built-in variants and props rather than adding custom Tailwind classes for styling. Shadcn components should serve as the single source of truth for the application's look and feel.

**DO:** Use component variants and props
```jsx
<Button variant="outline" size="sm">Click me</Button>
<Card>Content</Card>
<Dialog>
  <DialogTrigger asChild>
    <Button variant="ghost">Open</Button>
  </DialogTrigger>
</Dialog>
```

**DON'T:** Override with custom Tailwind classes
```jsx
// Avoid this pattern:
<Button variant="outline" className="border-border text-white bg-transparent">Click me</Button>
<Card className="p-6 rounded-lg shadow-md">Content</Card>
```

**Exceptions:** Custom Tailwind classes should only be added to shadcn components in extreme circumstances where:
1. Layout positioning is needed (e.g., `className="mt-4"`, `className="flex-1"`)
2. The required styling genuinely cannot be achieved through existing component variants
3. A one-off edge case requires specific behavior not covered by the design system

When custom classes are absolutely necessary, they should be limited to layout/spacing utilities rather than appearance modifications (colors, borders, shadows, etc.). This ensures consistency across the application and makes theme changes easier to manage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/therealchrisrock)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/therealchrisrock)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
