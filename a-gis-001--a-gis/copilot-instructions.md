## a-gis

> You have two modes of operation:

## Core Rules

You have two modes of operation:

1. Plan mode - You will work with the user to define a plan, you will gather all the information you need to make the changes but will not make any changes
2. Act mode - You will make changes to the codebase based on the plan

- You start in plan mode and will not move to act mode until the plan is approved by the user.
- You will print `# Mode: PLAN` when in plan mode and `# Mode: ACT` when in act mode at the beginning of each response.
- Unless the user explicity asks you to move to act mode, by typing `ACT` you will stay in plan mode.
- You will move back to plan mode after every response and when the user types `PLAN`.
- If the user asks you to take an action while in plan mode you will remind them that you are in plan mode and that they need to approve the plan first.
- When in plan mode always output the full updated plan in every response.

## Error Handling Pattern

For functions that return structs (using A_GIS.Code.make_struct):
1. Always include an `error` field in the return struct
2. Return validation errors as strings in the `error` field instead of raising exceptions
3. Set `error` to empty string ("") when no errors occur
4. Set other fields to appropriate default values when an error occurs

Example:
```python
def my_function(*, arg1: str, arg2: int) -> "type['A_GIS.Code.make_struct']":
    error = ""
    if not isinstance(arg1, str):
        error = "arg1 must be a string"
    if not isinstance(arg2, int):
        error = "arg2 must be an integer"
    
    return A_GIS.Code.make_struct(
        result=some_value if not error else None,
        error=error,
        _arg1=arg1,
        _arg2=arg2
    )
```

This pattern ensures consistent error handling across all struct-returning functions and makes it easier to handle errors at the call site.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/a-gis-001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/a-gis-001)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
