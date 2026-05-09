## avoid-unnecessary-comments

> Good comment style

# Avoid Unnecessary Comments

A rule to discourage adding excessive or redundant comments to the codebase.

<rule>
name: avoid_unnecessary_comments
description: Prevents redundant, obvious, or excessive comments in code

filters:
  # Apply to all code files with common extensions
  - type: file_extension
    pattern: "\\.(js|jsx|ts|tsx|py|rb|java|c|cpp|cs|go|rs|php|swift|kt)$"

actions:
  - type: reject
    conditions:
      # Look for obvious/redundant comments
      - pattern: "(?m)^\\s*(?://|/\\*|#|<!--|--) (?:This|The) \\w+ (?:function|method|class|variable)"
        message: "Avoid comments that simply state what a function/method/class/variable is"
      
      # Look for comments that just repeat the code
      - pattern: "(?m)^\\s*(?://|/\\*|#|<!--|--) (?:Increment|Decrement|Add|Set|Get|Create|Delete|Update|Remove)"
        message: "Avoid comments that just repeat what the code does - use clear naming instead"

  - type: suggest
    message: |
      # Comment Guidelines
      
      ## Avoid unnecessary comments:
      
      ❌ **Don't add comments that:**
      - State the obvious (e.g., `// increment counter`)
      - Repeat information clearly visible in the code
      - Explain WHAT the code does rather than WHY
      - Add noise that reduces readability
      
      ✅ **Do add comments when:**
      - Explaining WHY something is implemented in a certain way
      - Documenting complex business logic or algorithms
      - Noting edge cases or non-obvious behavior
      - Explaining temporary workarounds (with tickets/issues referenced)
      
      Instead of adding explanatory comments, prefer:
      - Clear, descriptive variable and function names
      - Small, focused functions with single responsibilities
      - Proper code organization and structure

examples:
  - input: |
      // This function gets the user data
      function getUserData() {
          return userData;
      }
    output: |
      function getUserData() {
          return userData;
      }
  
  - input: |
      // This implementation uses a cache to avoid expensive recalculations
      // when handling high-volume requests in production environments
      function getCachedResults(query) {
          if (cache.has(query)) {
              return cache.get(query);
          }
          const result = computeExpensiveResult(query);
          cache.set(query, result);
          return result;
      }
    output: |
      // This implementation uses a cache to avoid expensive recalculations
      // when handling high-volume requests in production environments
      function getCachedResults(query) {
          if (cache.has(query)) {
              return cache.get(query);
          }
          const result = computeExpensiveResult(query);
          cache.set(query, result);
          return result;
      }

metadata:
  priority: high
  version: 1.0
</rule>

---
> Source: [atuinsh/desktop](https://github.com/atuinsh/desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
