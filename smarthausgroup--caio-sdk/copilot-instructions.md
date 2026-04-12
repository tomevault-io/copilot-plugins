## caio-sdk

> No Unsolicited Execution Rule - Do not execute commands without explicit permission


# No Unsolicited Execution Rule

**Status:** MANDATORY  
**Owner:** All AI Assistants (Composer, Codex, GPT-5)

---

## Rule: DO NOT Execute Commands or Tests Without Explicit Permission

**MANDATORY:** You MUST NOT execute commands, run tests, or perform actions unless the user explicitly asks you to do so.

---

## What This Means

### FORBIDDEN Actions (Without Explicit Request):

- ❌ Running tests after fixing code
- ❌ Executing scripts after creating/fixing them
- ❌ Running validation commands after making changes
- ❌ Running build commands after modifying code
- ❌ Executing any command "to verify" or "to test" without explicit user request
- ❌ Running any tool/command "just to check" or "to see if it works"

### ALLOWED Actions (Only When Explicitly Requested):

- ✅ Fixing code/files when asked
- ✅ Creating files when asked
- ✅ Reading files to understand context
- ✅ Searching codebase to find information
- ✅ Proposing changes (but NOT executing them)
- ✅ Running commands ONLY when user explicitly says "run this" or "execute this"

---

## Examples

### ❌ BAD: Executing Without Permission

**User:** "Fix the import error in prototype_gate.py"

**WRONG Response:**
- Fixes the file ✅
- Then runs: `python3 scripts/prototype/prototype_gate.py` ❌ (NOT ASKED)

**CORRECT Response:**
- Fixes the file ✅
- Says: "Fixed the import path. The file should now work when you run it."

### ❌ BAD: Running Tests After Fixes

**User:** "Update the test to check for non-zero scores"

**WRONG Response:**
- Updates the test ✅
- Then runs: `pytest tests/prototype/test_query_superposed.py` ❌ (NOT ASKED)

**CORRECT Response:**
- Updates the test ✅
- Says: "Updated the test. You can run it with `pytest tests/prototype/test_query_superposed.py` when ready."

### ✅ GOOD: Only Fixing What's Asked

**User:** "Fix the import error in prototype_gate.py"

**CORRECT Response:**
- Fixes the file ✅
- Confirms: "Fixed the import path by adding project root to sys.path. The file should work when you run it."

---

## When User Explicitly Requests Execution

**ONLY execute when user says:**
- "Run this test"
- "Execute this command"
- "Run the prototype gate"
- "Test this"
- "Verify this works"
- "Go ahead and run it"

**DO NOT execute when user says:**
- "Fix this file" (just fix it, don't run it)
- "Update this test" (just update it, don't run it)
- "Create this script" (just create it, don't run it)
- "Make this work" (just make it work, don't test it)

---

## Enforcement

**MANDATORY:** This rule applies to ALL commands, tests, scripts, and executions.

**Before executing ANY command:**
- Ask yourself: "Did the user explicitly ask me to run/execute/test this?"
- If NO: DO NOT execute. Just fix/create/update what was asked.
- If YES: Proceed with execution.

---

## Rationale

Users want control over when tests/commands run. They may want to:
- Review changes first
- Run tests in a specific environment
- Run tests at a specific time
- Run tests with specific flags/options
- Not run tests at all (just want the fix)

**Respect user intent:** If they ask to fix something, just fix it. Don't assume they want it tested/executed.

---

**Last Updated:** 2025-12-30  
**Version:** 1.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SmartHausGroup)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SmartHausGroup)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
