## hvdc-pjt

> name: strict-project-scope

# .cursorrules – Single-File Strict Mode
name: strict-project-scope
version: 1
description: >
  Limit AI edits to src/, require pandas 2.2.*, forbid network I/O
  and ask confirmation on migrations.

scope:
  include:
    - "src/**"
  exclude:
    - "tests/**"
    - "scripts/exp/**"

behavior:
  mustUseLibraries:
    - "pandas"
  mustUsePatterns:
    - "^import typing"
  forbiddenPatterns:
    - "requests\\.get("
    - "os\\.system("
  askConfirmationIf:
    - "migration"
  codingStyle:
    formatter: "black"
    linter: "pylint"
  testCommand: "pytest -q"
  coverageThreshold: 80

security:
  requireSignedCommits: true
  blockSecretsRegex:
    - "AKIA[0-9A-Z]{16}"

meta:
  maintainers:
    - id: "@mr.cha"
      email: "mrcha@example.com"
  created: "2025-07-05"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/macho715) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
