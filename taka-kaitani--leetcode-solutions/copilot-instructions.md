## leetcode-solutions

> This is a LeetCode interview preparation repository where the user solves one problem per day. The primary purpose is to practice for technical interviews and maintain consistency in problem-solving.

# Claude Code Instructions for LeetCode Solutions Project

## Project Overview

This is a LeetCode interview preparation repository where the user solves one problem per day. The primary purpose is to practice for technical interviews and maintain consistency in problem-solving.

## Code Review Guidelines

When reviewing solutions in the "LeetCode Review" chat, focus on **interview readiness**:

1. **Correctness**: Verify the solution handles all cases correctly
2. **Complexity Analysis**: Ensure time/space complexity comments are filled in and accurate
3. **Interview Clarity**: Check if the solution is clear and easy to explain under pressure
   - Prefer simpler approaches over clever optimizations if equally valid
   - Flag overly complex code that would be hard to explain in 30 minutes
4. **Interview Anti-patterns**: Watch for:
   - Unnecessary premature optimization
   - Overly abstracted code for simple problems
   - Missing edge case handling
   - Poor variable naming that obscures logic

## Project Structure

```
problems/{number}_{title}/
├── solution.ts       # TypeScript solution file
└── README.md        # Problem description and notes
```

Tags are organized in `tags/` directory and track which problems use each tag/technique.

## Workflow

- **User solves, Claude reviews**: The user writes solutions independently, then asks for review
- **Don't implement solutions proactively**: Only write code when explicitly requested
- **Don't create documentation proactively**: No unsolicited README files or comments
- **Focus on interview lens**: When reviewing, think "Would this solution work in a 45-minute interview?"

## Scripts

The project includes helper scripts:
- Problem folder creation scripts (check `package.json` for available commands)
- Tag update utilities

## TypeScript Conventions

- Solutions use TypeScript
- Follow existing patterns in the codebase for consistency
- Keep solutions focused and interview-appropriate (avoid over-engineering)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taka-kaitani) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
