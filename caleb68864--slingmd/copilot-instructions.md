## dotnet-framework-namespace-naming

> description: Enforces .NET Framework namespace naming conventions

---
description: Enforces .NET Framework namespace naming conventions
globs: "*.cs"
---
# .NET Framework Namespace Naming

<rule>
name: dotnet_namespace_naming
description: Ensures namespaces follow .NET Framework naming conventions
filters:
  - type: file_extension
    pattern: "\\.cs$"
  - type: content
    pattern: "namespace\\s+[\\w.]+"

actions:
  - type: reject
    conditions:
      - pattern: "namespace\\s+[a-z]"
        message: "Namespace names must start with an uppercase letter"
      - pattern: "namespace\\s+[^\\w.]"
        message: "Namespace names can only contain letters, numbers, and dots"

  - type: suggest
    message: |
      Namespace naming guidelines:
      1. Start with an uppercase letter
      2. Use PascalCase
      3. Use dots for hierarchy
      4. Avoid underscores

examples:
  - input: |
      namespace myProject.core
      namespace 1InvalidNamespace
      namespace My_Project
    output: |
      namespace MyProject.Core

metadata:
  priority: high
  version: 1.0
</rule> 

---
> Source: [Caleb68864/SlingMD](https://github.com/Caleb68864/SlingMD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
