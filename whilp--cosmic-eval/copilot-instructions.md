## cosmic-eval

> This document outlines the development workflow and best practices for Claude when working on tasks in this repository.

# Guidelines for Claude

This document outlines the development workflow and best practices for Claude when working on tasks in this repository.

## Branch Management

**Always work on a dedicated branch for each task.**

- Before starting a new task, create a new branch (if not already on one)
- Branch names should follow the convention: `claude/<descriptive-name>-<session-id>`
- Never work directly on the main branch
- Each task should have its own isolated branch

## Commit Practices

**Commit frequently and comprehensively throughout the task.**

- Commit after each meaningful step or change
- Commit messages should explain the **rationale** for the change, not just what changed
- Include commits for:
  - Successful implementations
  - Detours and alternative approaches tried
  - Wrong turns and failed attempts
  - Debugging steps and investigations
  - Refactoring and improvements

**The commit history should be a complete record of the entire attempt to complete the task**, including the thought process, experimentation, and problem-solving journey.

### Commit Message Guidelines

- Use clear, descriptive messages that explain WHY the change was made
- Good: "Add error handling to prevent null pointer exceptions in user validation"
- Poor: "Update code" or "Fix bug"
- For failed attempts: "Try alternative approach using X instead of Y - encountered issue Z"
- For investigations: "Investigate performance bottleneck in data processing loop"

## Push Practices

**Push frequently to the remote branch.**

- Push after completing logical units of work
- Push at least after every few commits
- Don't wait until the task is complete to push
- This ensures work is backed up and visible

## Autonomous Work

**Work independently until task completion.**

- Analyze the task requirements thoroughly
- Research the codebase as needed
- Make implementation decisions based on best practices and existing patterns
- Complete the entire task without requiring intermediate guidance

## Testing and Validation

**Prove that your implementation works by running tests and providing evidence.**

- Never consider a task complete without proving it works
- Run the code/application and verify expected behavior
- Test edge cases and error conditions
- Execute automated tests if they exist
- Perform manual verification steps and document the results
- Show output, logs, or other evidence that demonstrates success
- Include test results in commit messages when relevant
- If it doesn't work, debug and fix until it does - then prove it

**Don't just write code and assume it works - prove it works.**

## Asking for Guidance

**If guidance is needed, ask the user upfront using the AskUserQuestion tool.**

- Don't make assumptions about ambiguous requirements
- Ask clarifying questions before starting work, not during
- If you encounter a blocker that requires user input, ask immediately
- Examples of when to ask:
  - Ambiguous or conflicting requirements
  - Design decisions that significantly impact architecture
  - Missing information needed to proceed
  - Choice between multiple valid approaches with different trade-offs

## Using Cosmic

**Only use the latest release from the cosmic repository.**

- Download and use the `cosmic-lua` release asset from https://github.com/whilp/cosmic
- Get the latest release using the GitHub API or releases page
- **NEVER clone the cosmic repository** - only download the release asset
- The cosmic-lua binary is the only artifact needed to complete tasks

This ensures you're working with stable, released versions of the tool rather than development code.

## Summary

The ideal workflow is:
1. **Create/verify branch** for the task
2. **Ask questions upfront** if any requirements are unclear
3. **Work autonomously** on the implementation
4. **Commit frequently** with detailed rationale in messages
5. **Push regularly** to backup work
6. **Prove the implementation works** by running tests and providing evidence
7. **Complete the task** fully before requesting review

This approach ensures a transparent, well-documented development process with a complete audit trail of the work performed, along with proof that the implementation actually works.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whilp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
