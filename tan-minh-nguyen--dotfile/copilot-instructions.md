## dotfile

> **MANDATORY**: Before writing or editing any code, read and follow ALL rules below.

# dotfile

## Coding Rules

**MANDATORY**: Before writing or editing any code, read and follow ALL rules below.

### Universal Rules (Reuse First)

**CRITICAL**: Before creating anything new, search and extend existing code.

| Rule                        | Guideline                                                                |
|-----------------------------|--------------------------------------------------------------------------|
| Prefer Extension            | Check `src/utils` and `src/hooks` (or equivalent) before new helpers     |
| Search First                | Use `grep` or `find` for similar logic before starting implementation   |
| Consolidate                 | If two functions share 80% logic, refactor into single reusable function |
| No Shadowing                | Never create local versions of globally available types or utilities     |

For EVERY new function/helper:
1. Search existing utils: `grep -r "functionPattern" src/`
2. Check hooks: `find . -path "*/hooks/*" -name "*.ts"`
3. Look for similar types: `grep -r "type.*TypeName" src/`
4. If similar exists, extend it instead of creating new

Full rules: .claude/code-rules/

### Language Rules Location

| Language   | Rules File                                  |
|------------|---------------------------------------------|
| emacs-lisp | .claude/code-rules/languages/emacs-lisp.org |

### Enforcement Checklist

For EVERY code change:

1. **Before writing**: Search for existing similar code (see Universal Rules above)
2. **Before writing**: Read `* FILE ORDER` section — structure code in that exact order
3. **During writing**: Follow `* TEMPLATE` structure as the canonical example
4. **After writing**: Verify against `* ANTI-PATTERNS` — fix any violations before presenting
5. **For existing files**: Preserve or improve order — never make order worse
6. **Validation**: Walk through FILE ORDER and confirm each section appears in correct sequence

### Quick Reference

See the full rules in the language files above. Key points:
- **Search before creating** — extend existing utils/hooks, don't duplicate
- File sections must appear in specified order (e.g., requires -> defgroup -> defvar -> functions -> provide)
- Functions < 20 lines, single responsibility
- No inline comments except for "why"
- Extract helpers to avoid duplication

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tan-minh-nguyen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tan-minh-nguyen)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
