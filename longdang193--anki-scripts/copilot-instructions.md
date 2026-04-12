## anki-scripts

> Apply when user requests refactoring, optimization, or performance improvement. Use for OPTIMIZE task type.


# ⚡ Optimize Mode

**Goal:** Improve quality **WITHOUT changing behavior**.

## Process

1. Measure current state (baseline)
2. Identify main bottleneck
3. Propose improvements + predict results
4. Refactor by priority order
5. Compare before/after
6. Ensure tests still pass

## Evaluation Criteria

| Criterion | Tool | Good Threshold |
|-----------|------|----------------|
| Code complexity | Manual review, ESLint | Cyclomatic < 10 |
| Type coverage | JSDoc annotations | Public APIs documented |
| Duplication | Manual review, grep | Follow DRY principles |
| File length | Manual review | < 200 lines/file |
| Function length | Manual review | < 50 lines/function |
| DOM performance | Browser DevTools | No layout thrashing |

## Web-Specific Metrics

- **Card rendering time**: Measure before/after optimization
- **Memory usage**: Monitor in browser DevTools
- **DOM manipulation performance**: Ensure optimizations don't degrade user experience
- **CSS performance**: Check for expensive selectors, layout thrashing

## Output Format

```markdown
## ⚡ OPTIMIZE

**Issue:** [slow / duplicate code / hard to maintain]

**Baseline:**
- LOC: X lines
- Complexity: X (cyclomatic)
- Duplication: X instances
- Type coverage: X%

---

### Bottleneck:
| Issue | Location | Severity |
|-------|----------|----------|
| [Description] | `file:line` | 🔴 High |

### Proposal:
| Item | Before | After | Δ |
|------|--------|-------|---|
| LOC | 500 | 350 | -30% |
| Duplication | 3 instances | 0 | -100% |

### Regression Check:
- [ ] Anki preview still works correctly
- [ ] Browser console shows no errors
- [ ] Behavior unchanged (verify in Anki preview)
- [ ] Performance maintained or improved (check rendering time)
```

## HTML/CSS/JavaScript Optimization Strategies

- **DRY violations**: Extract shared utilities (see reuse-first principles)
- **Type safety**: Add JSDoc annotations to untyped functions
- **Code organization**: Split large files, extract modules
- **Performance**: Optimize DOM queries, reduce reflows/repaints
- **CSS**: Consolidate duplicate styles, use CSS variables
- **Anki-specific**: Preserve DOM structure while optimizing (see `@anki-specific-considerations.mdc`)

## Principles

| ❌ DON'T | ✅ DO |
|----------|-------|
| Optimize prematurely | Measure first, optimize later |
| Change behavior | Keep behavior unchanged |
| Prioritize cleverness | Readability > Performance |
| Skip verification | Test in Anki preview |
| Break DOM structure | Maintain DOM for Anki compatibility |
| Ignore workspace rules | Follow reuse-first, SRP, DRY |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/longdang193)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/longdang193)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
