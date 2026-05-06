## 80-vibe-coding

> USE WHEN: prototyping, rapid iteration, hackathons, MVPs, or exploratory development.


# 🎸 Vibe Coding Protocol

> "The vibes are the goal. The code is the vehicle." — Vibe Coding Manifesto

## When to Activate

Enable Vibe mode for:
- ✅ Prototyping and proof-of-concept
- ✅ Exploratory development
- ✅ Time-critical feature delivery
- ✅ Hackathons and MVPs
- ✅ Early-stage product validation

Do NOT use for:
- ❌ Production-critical systems
- ❌ Security-sensitive code
- ❌ Code that others will maintain immediately
- ❌ Financial or healthcare systems

---

## Core Principles

| Traditional | Vibe Mode |
|-------------|-----------|
| Debug line by line | Reroll with better prompt |
| Get it right first time | Iterate rapidly |
| Commit when complete | Commit every working state |
| Focus on code quality | Focus on product delivery |
| Perfect is the goal | Working is the goal |

---

## The Reroll Strategy

### When to Reroll (Regenerate)

```
IF debugging_time > 10 minutes THEN
  → Stop debugging
  → Clarify the problem
  → Regenerate the module with better context
```

### How to Reroll

1. **Identify what's not working**
2. **Document the failure mode**
3. **Provide clearer context/constraints**
4. **Generate fresh implementation**
5. **Compare and merge the best parts**

This is often faster than fixing convoluted code.

---

## Commit Checkpoint System

### Rule: Every Working State = Commit

```bash
# ✅ Good commit frequency in Vibe mode
git commit -m "feat: basic form renders"
git commit -m "feat: form validates on submit"
git commit -m "feat: form submits to API"
git commit -m "fix: handle empty state"
git commit -m "feat: success toast on submit"
```

### Why?
- Easy rollback to last working state
- Clear progress visibility
- Safe experimentation

### Commit Message Format
```
type: short description

# Types:
feat:     New feature
fix:      Bug fix
wip:      Work in progress (Vibe mode)
refactor: Code improvement
style:    Formatting only
```

---

## Speed Over Perfection

### In Vibe Mode, Optimize For:

1. **Working software** over perfect code
2. **User value** over technical elegance
3. **Fast feedback** over comprehensive testing
4. **Learning** over planning

### Acceptable Trade-offs (Temporarily):
- Inline styles instead of design system
- Hardcoded values (mark with `// TODO: extract`)
- Simplified error handling
- Missing edge cases (document them)

### Track Technical Debt
```typescript
// TODO(vibe): Extract to utility function
// TODO(vibe): Add proper error handling
// TODO(vibe): Support edge case X
// FIXME(vibe): This is a hack, refactor later
```

---

## Product Thinking

### Focus Questions
- What problem are we solving?
- Who is the user?
- What's the simplest thing that works?
- How will we know it's successful?

### Build → Measure → Learn
```
1. Build the smallest thing that tests the hypothesis
2. Ship it to real users
3. Learn from their behavior
4. Iterate or pivot
```

---

## 🛡️ Non-Negotiable Guardrails

Even in Vibe mode, these rules ALWAYS apply:

### Security (NO EXCEPTIONS)
- [ ] No hardcoded secrets
- [ ] Input validation on all external data
- [ ] Auth checks on protected routes
- [ ] No SQL injection vectors

### Quality Baseline
- [ ] Tests required before merge to main
- [ ] Code must compile/build
- [ ] No console errors in production
- [ ] Basic error boundaries in place

### Documentation
- [ ] README explains how to run
- [ ] TODO comments for known shortcuts
- [ ] Breaking changes documented

---

## Exit Criteria

### Move OUT of Vibe mode when:

| Trigger | Action |
|---------|--------|
| Feature validated by users | Refactor for maintainability |
| Moving to production | Add comprehensive tests |
| Others will maintain | Document and clean up |
| Security review needed | Full security audit |
| Performance critical | Optimize and benchmark |

### Transition Checklist
- [ ] Replace all `// TODO(vibe)` comments
- [ ] Add missing error handling
- [ ] Add missing tests
- [ ] Review security implications
- [ ] Update documentation
- [ ] Get code review

---

## Communication in Vibe Mode

### To the AI (Claude/Cursor/Copilot):
```
We're in Vibe mode for this feature.
Priorities: speed, working code, ship fast.
Trade-offs OK: can skip edge cases, note as TODOs.
Non-negotiable: no security issues, must compile.
```

### To Teammates:
```
This is a prototype/MVP. Known shortcuts:
- [ ] Missing error handling for X
- [ ] Hardcoded config values
- [ ] No unit tests yet
Will clean up after user validation.
```

---

## Quick Reference

```
┌─────────────────────────────────────────────┐
│  VIBE CODING QUICK CARD                     │
├─────────────────────────────────────────────┤
│  ✅ DO:                                     │
│  • Commit every working state               │
│  • Reroll if stuck >10 min                  │
│  • Track debt with TODO(vibe)               │
│  • Focus on user value                      │
│  • Ship and learn                           │
├─────────────────────────────────────────────┤
│  ❌ DON'T:                                  │
│  • Skip security rules                      │
│  • Commit broken code                       │
│  • Ignore critical errors                   │
│  • Use for production systems               │
│  • Forget to exit Vibe mode                 │
└─────────────────────────────────────────────┘
```

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
