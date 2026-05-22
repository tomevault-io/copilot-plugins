## exception-handling

> Enforces using System.Exception instead of just Exception in catch blocks


# Exception Handling Standards

Enforces using fully qualified `System.Exception` instead of just `Exception` in catch blocks to avoid ambiguity.

<rule>
name: exception_handling_standards
description: Enforces using System.Exception instead of just Exception in catch blocks

filters:
  - type: file_extension
    pattern: "\\.cs$"
  - type: content
    pattern: "catch\\s*\\(\\s*Exception\\s*\\)"

actions:
  - type: suggest
    conditions:
      - pattern: "catch\\s*\\(\\s*Exception\\s*\\)"
        message: |
          Use fully qualified System.Exception instead of just Exception to avoid ambiguity:
          ```csharp
          // Instead of:
          catch (Exception)
          
          // Use:
          catch (System.Exception)
          ```

examples:
  - input: |
      try
      {
          // Some code
      }
      catch (Exception)
      {
          // Handle error
      }
    output: |
      try
      {
          // Some code
      }
      catch (System.Exception)
      {
          // Handle error
      }

metadata:
  priority: high
  version: 1.0
</rule> 

---
> Source: [Caleb68864/SlingMD](https://github.com/Caleb68864/SlingMD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
