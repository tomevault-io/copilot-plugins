## smilepile

> You are working in an Atlas Framework project. You MUST follow these rules:

# Atlas Framework Enforcement for Cursor

You are working in an Atlas Framework project. You MUST follow these rules:

## MANDATORY REQUIREMENTS

1. **NEVER write code without a story**
   - Check atlas/09_STORIES/ for existing stories
   - Create new stories before implementing

2. **ALWAYS use Atlas workflows**
   - Story creation → Development → Review → Deploy
   - No shortcuts or custom processes

3. **USE Atlas prompts**
   - Reference atlas/01_CORE/PROMPTS.md
   - Copy and adapt proven patterns

4. **SPAWN agents with proper types**
   - story-writer, bug-fixer, ui-developer, backend-developer, tester
   - Let context injection work automatically

5. **FOLLOW the established patterns**
   - Check existing code for patterns
   - Maintain consistency

## WORKSPACE STRUCTURE

```
atlas/
├── 01_CORE/PROMPTS.md      # ← Use these prompts
├── 02_WORKFLOWS/           # ← Follow these workflows
├── 09_STORIES/            # ← All work starts here
└── config/                # ← Don't modify
```

## BEFORE STARTING ANY TASK

1. Run: `python3 atlas/test_context_injection.py`
2. Check: `ls atlas/09_STORIES/`
3. Create story if needed
4. Follow Atlas workflow

## ENFORCEMENT

- Git hooks will check for story references
- Reviews require Atlas compliance
- Non-compliant work will be rejected

Remember: Atlas makes development faster and more consistent. Trust the process.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajstack22) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
