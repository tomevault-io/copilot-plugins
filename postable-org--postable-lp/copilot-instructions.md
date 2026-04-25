## postable-lp

> Rules for quality commits - clear, atomic, and well-structured messages


# 📋 Commit Rules

## 📝 Fundamental Principles

### 1. Size and Structure
- Commits must be **atomic** (one logical change per commit)
- **Maximum of 2 lines** in the message
- First line: **concise description** (maximum 72 characters)
- Second line (optional): **additional context** or reason for the change, separated by a blank line

### 2. Language
- All messages must be in **English**
- Variable names, code comments: according to project standards

### 3. Authorization and Execution
- Commits **ONLY when explicitly requested by the user**
- **NEVER make automatic commits** or commits on your own initiative
- **NEVER push** - only create local commits

### 4. Strategic Consolidation
- Group related changes into **a single commit** whenever possible
- Avoid multiple small commits for the same functionality
- Exception: logically independent changes may have separate commits

### 5. No Internal Noise
- **NEVER mention**: "PHASE 1", "PHASE 2", "BUG 1", "TODO 1", etc.
- Use clear and direct descriptions of the functionality
- The history should make sense to any external developer

---

## 🏷️ Standardized Prefixes

Use exactly one of the prefixes below, followed by **a colon and a space**:

| Prefix | Usage | Example |
|---------|-----|---------|
| **`feat:`** | New functionality | `feat: add dark mode toggle to settings` |
| **`fix:`** | Bug fix | `fix: resolve infinite loop in user data fetch` |
| **`refactor:`** | Refactoring (no functionality change) | `refactor: simplify component prop drilling with context` |
| **`docs:`** | Documentation and comments | `docs: update API endpoints in README` |
| **`style:`** | Formatting, no logic | `style: format imports according to prettier rules` |
| **`test:`** | Tests (add/modify) | `test: add unit tests for auth validation` |
| **`chore:`** | Build, dependencies, config | `chore: update dependencies to latest stable versions` |
| **`perf:`** | Performance optimization | `perf: memoize expensive component calculations` |
| **`build:`** | Build system, CI/CD | `build: configure webpack for production builds` |

---

## 📝 Format and Examples

### Basic Structure
```
prefix: short imperative description
```

### With Additional Context (optional)
```
prefix: short imperative description

Longer description explaining the reason for the change,
impact, or important decisions made.
```

### Real Examples
✅ **GOOD:**
```
feat: add atomic design structure for components
```

✅ **GOOD (with context):**
```
refactor: extract Button styles to shared theme

This enables consistent theming across all UI components
and makes future brand updates simpler.
```

❌ **BAD:**
```
fix: fix bug
feat: adiciona funcionalidade
chore: update stuff
```

---

## ⚠️ What NOT to Do

- ❌ Commits without explicit user authorization
- ❌ Multiple small commits for related changes
- ❌ Push to remote repository (user does this when appropriate)
- ❌ Portuguese in the commit message (always English)
- ❌ Vague or generic messages
- ❌ References to internal tasks ("PHASE X", "BUG 1")
- ❌ Commits with more than 2 lines without reason

---

## 🔄 Commit Workflow

1. Check status: `git status` and `git diff`
2. Add relevant files: `git add <files>`
3. Validate changes again
4. Create commit with appropriate message
5. ⛔ **NEVER push** (user does this manually)

---

## 📊 When to Make Multiple Commits

**Make separate commits ONLY when:**
- Changes are **completely independent** (e.g., isolated bugfix during feature work)
- Different features with distinct lifecycles
- Unrelated refactoring in a different file

**Consolidate into a single commit when:**
- Changes implement a single resource
- Changes are interdependent
- They comprise a single "unit of work"

---

## 🎯 Checklist Before Committing

- [ ] Explicit user authorization received?
- [ ] Code tested and functional?
- [ ] All relevant files added to staging?
- [ ] Message follows the correct prefix?
- [ ] Description is clear and imperative?
- [ ] Message is in English?
- [ ] No internal references ("PHASE X", "BUG X")?
- [ ] Commit is atomic (one logical change)?
- [ ] No accidental push to remote?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/postable-org) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
