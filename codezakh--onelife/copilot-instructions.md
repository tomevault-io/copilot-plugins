## development-protocol

> **You are working in a properly configured, professional codebase.** Never write code that assumes broken tooling, missing dependencies, or improper setup. If something doesn't work, the solution is to fix the underlying issue, not to hack around it.

## DEVELOPMENT PROTOCOL - MANDATORY RULES

### FUNDAMENTAL ASSUMPTION
**You are working in a properly configured, professional codebase.** Never write code that assumes broken tooling, missing dependencies, or improper setup. If something doesn't work, the solution is to fix the underlying issue, not to hack around it.

### CORE WORKFLOW
1. **Write code in logical units** (complete feature, module, or coherent functionality)
2. **Run tests immediately after each unit**
3. **If ANY test fails or error occurs**: STOP. Use the REQUIRED FORMAT below and wait for confirmation
4. **If typing issues arise**: STOP. Ask for guidance rather than using `Any` or ignoring types

### FORBIDDEN BEHAVIORS - NEVER DO THESE
- **No defensive imports**: Never wrap imports in try/except blocks or provide fallbacks
- **No sys.path manipulation**: Never use `sys.path.append()` or similar hacks
- **No src imports**: Never import from `src.module_name`. Always use the proper module name directly
- **No command-line testing**: Always write proper test files in the test directory
- **No imports inside functions or mid-file**: ALL imports go at the very top of the file, after docstrings only
- **No test simplification**: If a test fails, fix the code to match the PRD, don't weaken the test
- **No multi-step fixes**: Make ONE change at a time, then test
- **No ignoring type errors**: If mypy/type checker fails, fix it before continuing
- **No placeholder implementations**: Don't use `pass`, `TODO`, or `raise NotImplementedError` unless explicitly requested
- **No path hacking**: Don't manipulate PYTHONPATH, sys.path, or use relative imports to work around package structure

### IMPORT RULES - CRITICAL
- **Always at the top**: Every import statement must be at the very top of the file (after module docstring only)
- **Never mid-file**: Never add imports in the middle of a file, even if you're adding functionality later
- **Use proper package names**: Import using the actual package name, never through `src`
- **Trust the environment**: Assume the package is properly installed or the development environment is correctly configured

### TEST QUALITY RULES - CRITICAL
**Write tests that verify PRD requirements, not just "does code run":**
- **Test actual functionality**: Don't test object creation - test that the object behaves correctly according to specifications
- **Test meaningful behavior**: Focus on business logic and requirements, not implementation details
- **Don't make assumptions**: Never write assertions unless the PRD explicitly states the expected behavior
- **Test edge cases**: What happens with invalid inputs? Boundary conditions? Error states?
- **Test requirements**: Every test should map back to a specific requirement or behavior described in the PRD

**Bad test pattern:**
```python
def test_object_creation():
    obj = MyClass(param=42)
    assert obj.param == 42  # This tests nothing meaningful
```

**Good test pattern:**
```python
def test_object_processes_input_according_to_spec():
    obj = MyClass()
    result = obj.process(input_data)
    assert result.meets_specific_requirement_from_prd()  # Tests actual functionality
```

### DECISION MATRIX
**When you encounter an error, ask yourself:**
- Is this a trivial fix? (typo, missing comma, obvious syntax error, simple variable rename)
  - YES: Fix it and continue
  - NO: STOP and ask for confirmation using the REQUIRED FORMAT

### REQUIRED FORMAT FOR STOPPING - USE THIS EXACTLY
**You MUST use this exact format when stopping and wait for my response:**
```
🛑 STOPPING FOR CONFIRMATION

ERROR ENCOUNTERED: [describe the specific error]
MY DIAGNOSIS: [why you think this is happening]
PROPOSED FIX: [ONE specific change you want to make]

CONFIRMATION NEEDED: Does this approach make sense before I proceed?
```

### CONFIRMATION REQUIREMENT
**Before starting any work, you must confirm understanding by:**
1. Listing the three most critical stopping conditions
2. Explaining when you would use the REQUIRED FORMAT above
3. Giving one example each of a bad test vs. good test

### BEFORE WRITING TESTS CHECKLIST
- [ ] Am I testing actual PRD requirements, not just "does code run"?
- [ ] Am I testing meaningful functionality, not just object creation?
- [ ] Am I avoiding assumptions about behavior not specified in the PRD?

### BEFORE WRITING CODE CHECKLIST
- [ ] Are ALL imports at the very top of the file?
- [ ] Am I using proper import statements that match the codebase pattern?
- [ ] Will any errors require me to STOP and use the REQUIRED FORMAT?

**Please confirm your understanding using the confirmation requirement above.**

---
> Source: [codezakh/onelife](https://github.com/codezakh/onelife) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
