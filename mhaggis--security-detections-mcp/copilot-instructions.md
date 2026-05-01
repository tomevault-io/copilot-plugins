## self-healing

> Self-healing behavior - automatically fix errors and continue


# Self-Healing Error Recovery

When code fails, errors occur, or something breaks during execution, DO NOT stop and ask the user what to do. Instead:

## Immediate Response to Errors

1. **Analyze the error** - Read the full error message, stack trace, or failure output
2. **Identify root cause** - Determine what actually went wrong (syntax, missing dep, wrong path, logic error, etc.)
3. **Fix it yourself** - Make the necessary code changes to resolve the issue
4. **Re-run and verify** - Execute again to confirm the fix worked
5. **Continue the task** - Once fixed, proceed with the original goal

## Self-Fix Loop

```
Error occurs
    ↓
Read error message/output
    ↓
Diagnose the problem
    ↓
Apply fix (edit code, install dep, correct path, etc.)
    ↓
Re-run the failing operation
    ↓
Fixed? → Continue original task
Not fixed? → Try different approach (max 3 attempts)
    ↓
Still failing after 3 attempts? → Explain what was tried and ask for help
```

## Common Fixes to Apply Automatically

### Import/Module Errors
- Missing import → Add the import
- Wrong path → Fix the path
- Missing dependency → Install it with npm/pip

### Syntax Errors
- Missing bracket/brace → Add it
- Typo in variable name → Fix the typo
- Wrong indentation → Correct it

### Runtime Errors
- File not found → Check path, create if needed
- Permission denied → Use correct permissions flag
- Connection failed → Check URL/credentials, retry

### Test Failures
- Assertion failed → Fix the logic that caused the failure
- Timeout → Increase timeout or fix slow code
- Missing mock → Add the mock

### Build/Compile Errors
- Type error → Fix the types
- Missing export → Add the export
- Circular dependency → Refactor to break the cycle

## What NOT to Do

- Don't immediately ask "what should I do?"
- Don't give up after one failure
- Don't just report the error without attempting a fix
- Don't make the same fix attempt twice

## Logging Fixes

When you fix something, briefly note:
- What broke
- What you changed
- That it's now working

Example: "Fixed missing uuid import in detection-engineer.ts - added `import { v4 as uuidv4 } from 'uuid'`. Continuing..."

## Escalation

Only ask the user for help when:
- You've tried 3+ different approaches
- The error requires external action (API key, service restart, hardware issue)
- The fix would change the user's intended behavior
- You're genuinely unsure what the correct behavior should be

Otherwise, fix it and keep going.

---
> Source: [MHaggis/Security-Detections-MCP](https://github.com/MHaggis/Security-Detections-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
