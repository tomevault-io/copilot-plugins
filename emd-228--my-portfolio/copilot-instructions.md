## my-portfolio

> **BEFORE CREATING ANY COMMIT:**

# Cursor Rules for my-design-learning Project

## ⚠️ IMPORTANT: READ THIS FIRST ⚠️

**BEFORE CREATING ANY COMMIT:**
1. Read ALL commit rules below
2. Check which files are modified: `git status`
3. Create SEPARATE commits for each file (unless files are closely related)
4. Follow the format: `<type>: <subject>` (max 50 chars)
5. Use imperative mood in all commit messages

**CRITICAL RULE:** Line 113 states: "Commit each modified file separately when possible, unless files are closely related"

## Git Commit Rules

When helping with Git commits in this project, ALWAYS follow these rules from COMMIT_RULES.md:

### Commit Message Format

**Structure:**
```
<type>: <subject>

<body>

<footer>
```

**Rules:**
1. **Subject Line**: 
   - Capitalized, short summary (50 characters or less)
   - Use imperative mood: "Add component" not "Added component" or "Adds component"
   - No period at the end
   - Must start with a type prefix

2. **Blank Line**: Always leave the second line blank

3. **Body** (optional but recommended):
   - Wrap text to about 72 characters
   - Explain what and why, not how
   - Use imperative mood
   - Can use bullet points (hyphen or asterisk)

### Commit Types

Use these prefixes:
- `feat`: A new feature (component, section, functionality)
- `fix`: A bug fix
- `style`: Changes that don't affect code meaning (formatting, CSS, whitespace)
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: Performance improvement
- `test`: Adding or updating tests
- `docs`: Documentation only changes
- `chore`: Changes to build process, dependencies, or tooling
- `ui`: UI/UX improvements (styling, layout, visual changes)

### Commit Principles

1. **Commit Related Changes Only**: One logical change per commit
   - ✅ Good: One commit for adding a component, another for fixing a bug
   - ❌ Bad: One commit that adds a component AND fixes a bug AND updates styles

2. **Commit Often**: Keep commits small and focused

3. **Don't Commit Half-Done Work**: Only commit when a logical component is completed

4. **Test Before Committing**: Ensure code works and TypeScript compiles

### Vue Component Commit Patterns

- **New component**: `feat: add ComponentName component`
- **Component update**: `feat: enhance ComponentName with [feature]`
- **Component fix**: `fix: [issue] in ComponentName`
- **Component styling**: `ui: improve ComponentName styling`

### Examples

**Simple feature:**
```
feat: add Testimonials component with horizontal scroll
```

**Feature with description:**
```
feat: enhance ImpressionItem with badges and icons overlay

- Add sale badge overlay on product image
- Add interactive icons (heart, cart, eye) at bottom of image
- Improve layout with better spacing and typography
```

**Bug fix:**
```
fix: center icon container in ImpressionItem

The icon container was not properly centered horizontally.
Added -translate-x-1/2 to center the flex container relative
to its left-1/2 position.
```

**UI improvement:**
```
ui: improve Testimonials scroll behavior

- Add overflow-x-scroll for horizontal scrolling
- Add flex-shrink-0 to prevent items from shrinking
```

### When Creating Commits

**MANDATORY STEPS:**
1. Read `.cursorrules` file completely before creating commits
2. Check `git status` to see all modified files
3. Create ONE commit per file (unless files are closely related - e.g., config files)
4. Always suggest commit messages following this format
5. When user asks to commit multiple files, CREATE SEPARATE COMMITS for each file
6. Remind to test code before committing
7. Use imperative mood in all commit messages
8. Keep subject line under 50 characters
9. Always include a blank line after the subject if there's a body

**EXAMPLE:** If 3 components are modified, create 3 separate commits, not 1!

### Project Context

This is a Vue 3 + TypeScript + Tailwind CSS project. Components are in `src/components/`. When committing:
- One commit per component creation
- Separate commits for styling improvements
- Separate commits for functionality additions
- **CRITICAL: Commit each modified file separately when possible, unless files are closely related**
  - ✅ Related: `package.json` + `package-lock.json` (dependencies)
  - ✅ Related: `style.css` + `tailwind.config.js` (font configuration)
  - ❌ NOT Related: `ComponentA.vue` + `ComponentB.vue` (separate commits!)
  - ❌ NOT Related: `Component.vue` + `data.json` (separate commits!)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EmD-228) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
