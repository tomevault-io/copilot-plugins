## genhq-platform

> - Write clean, maintainable code following project conventions

# Development Guidelines

## Core Principles

### Code Quality

- Write clean, maintainable code following project conventions
- Prefer existing patterns over introducing new ones
- Check similar components before building new features
- Test changes before marking tasks complete

### File Organization

- Read relevant source files before making changes
- Understand existing architecture before adding features
- Keep related code together
- Use consistent naming conventions

### Project Context & Dependencies

**Before building features:**

1. Find and read project manifest (`package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, etc.)
2. Check available dependencies, scripts, and tools
3. Ask permission before adding new dependencies
4. Prefer using existing dependencies over adding new ones

### Styling Architecture (if applicable)

**Core Rule: Utility-First + Design Tokens**

- ✅ Use Tailwind utilities for layout, spacing, sizing, responsive design
- ✅ Use CSS variables from `globals.css` for colors, fonts, shadows, animations
- ❌ NO inline `style={{ }}` for static properties
- ⚠️ ONLY inline styles for dynamic values (e.g., `style={{ animationDelay: '${delay}ms' }}`)

**Reference Files:**

- Design tokens: `globals.css`, `tailwind.config.js`
- Component patterns: Check existing components in same category

**Component Building Process:**

1. **FIRST**: Check if design system files exist (`globals.css`, `tailwind.config.js`)
   - If missing, inform user: "Design system not configured yet. Would you like me to set up a basic design system?"
2. Read design token files to see available variables
3. Check existing components for established patterns
4. Use utility classes for structure/layout
5. Use design token classes for visual identity

### Validation After Changes

**After creating or editing files:**

1. Check project manifest (`package.json`, etc.) for validation scripts
2. Run available type checking, linting, or build commands
3. Fix any errors immediately
4. Only mark task complete when validation passes
5. If no validation tools exist, suggest adding them

### Before Committing

- Run tests if they exist
- Verify all type checks and linting pass
- Check for console errors
- Verify functionality matches requirements
- Use `/commit` command for clear git messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/willem4130)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/willem4130)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
