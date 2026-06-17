## hello-world-skill

> A greeting skill that ONLY generates simple hello/greeting messages. NOT for calculations, data processing, or code execution. Use ONLY for greeting requests like "say hello", "greet someone", or similar.


# Hello World Skill

This skill ONLY generates simple greeting messages. It is NOT suitable for calculations, data processing, or any code execution tasks.

## When to Use This Skill

**ONLY** use this skill when:
- The user explicitly asks to "say hello" or "greet" someone
- The user wants a simple greeting message
- Testing basic greeting functionality

**DO NOT** use this skill for:
- Mathematical calculations (use calculator-skill instead)
- Data processing or analysis
- Code execution
- Any task requiring computation

## How It Works

This skill generates a simple greeting message in the format "Hello, [name]!" where [name] can be:
- A name provided by the user
- "World" as the default if no name is specified

## Instructions

1. Check if the user has provided a name to greet
2. If no name is provided, use "World" as the default
3. Generate the greeting message: "Hello, [name]!"
4. Optionally include a timestamp

**IMPORTANT:** This skill should NOT invoke any tools for calculations or code execution. It only returns a simple text greeting.

## Examples

### Example 1: Default Greeting

**User request:** "Say hello"

**Output:**
```
Hello, World!
Generated at: 2024-12-25 10:30:00 UTC
```

### Example 2: Personalized Greeting

**User request:** "Say hello to Alice"

**Output:**
```
Hello, Alice!
Generated at: 2024-12-25 10:30:00 UTC
```

### Example 3: Multiple Names

**User request:** "Greet Bob and Charlie"

**Output:**
```
Hello, Bob!
Hello, Charlie!
Generated at: 2024-12-25 10:30:00 UTC
```

## What This Skill Does NOT Do

- ❌ Mathematical calculations
- ❌ Data processing
- ❌ Code execution
- ❌ File operations
- ❌ Web requests

If a user request involves any of the above, use a different skill (e.g., calculator-skill for calculations).

## Edge Cases

- **Empty name:** Treat as if no name was provided, use "World"
- **Special characters in name:** Include them in the greeting as-is
- **Very long name:** Accept and use the full name provided

## Output Format

The greeting should be clear and friendly. Include:
1. The greeting message
2. A timestamp (optional but recommended)

Keep the output simple and human-readable.

---
> Source: [smallnest/hello-world-skill](https://github.com/smallnest/hello-world-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
