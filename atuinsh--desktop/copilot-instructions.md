## conventional-commits

> Writing a good commit message

# Conventional Commits

A rule to enforce proper conventional commit formatting with clear, descriptive messages.

<rule>
name: conventional_commits
description: Enforces conventional commit format with concise, explanatory first lines

filters:
  - type: event
    pattern: "git_commit"

actions:
  - type: reject
    conditions:
      # Reject non-conventional commit types
      - pattern: "^(?!feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(?:\\(.*\\))?:"
        message: "Commit message must start with a conventional type (feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert)"
      
      # Reject first lines that are too long (>72 chars)
      - pattern: "^.{73,}"
        message: "First line should be 72 characters or less to avoid truncation in GitHub and other tools"
      
      # Reject commits without a space after the colon
      - pattern: "^\\w+(?:\\(.*\\))?:(?!\\s)"
        message: "Include a space after the colon in your commit message"
      
      # Reject first lines that start with lowercase (after the type)
      - pattern: "^\\w+(?:\\(.*\\))?:\\s[a-z]"
        message: "First word after type should be capitalized"

  - type: suggest
    message: |
      # Conventional Commit Format
      
      Follow this format for all commits:
      
      ```
      type(scope): Short summary
      
      Detailed explanation if needed
      ```
      
      ## Requirements:
      
      - **Type**: One of: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
      - **Scope**: Optional component name in parentheses
      - **Summary**: 
        - Clear and concise (50-72 chars)
        - Written in imperative ("Add feature" not "Added feature")
        - First letter capitalized
        - No period at the end
      - **Body**: Optional, separated by blank line, explains WHY not HOW
      
      ## Examples:
      
      ✅ `feat(auth): Add password reset functionality`
      
      ✅ `fix: Prevent crash when API returns empty response`
      
      ✅ `docs(readme): Update installation instructions`

examples:
  - input: |
      added new button to the ui
    output: |
      feat(ui): Add new button
  
  - input: |
      fixed the bug in authentication
    output: |
      fix(auth): Resolve authentication failure on expired sessions
  
  - input: |
      feat(users): implemented email verification system with token expiration and email templates for better onboarding experience
    output: |
      feat(users): Implement email verification system
      
      Add token-based verification with:
      - 24h expiration
      - Customized email templates
      - Improved onboarding flow

metadata:
  priority: high
  version: 1.0
</rule>

---
> Source: [atuinsh/desktop](https://github.com/atuinsh/desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
