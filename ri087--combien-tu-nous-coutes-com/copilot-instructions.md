## combien-tu-nous-coutes-com

> This is an example of a project-specific cursor rule. It demonstrates how to define rules that are only applied manually in individual repositories.

# Example Project-Specific Rule

This is an example of a project-specific cursor rule. It demonstrates how to define rules that are only applied manually in individual repositories.

<rule>
name: example_project_rule
description: A demonstration project-specific rule, manually included in a project
globs: ["src/some-project-specific-folder/**/*.ts"]
filters:
  - type: file_extension
    pattern: "\.ts$"
actions:
  - type: suggest
    message: |
      Use this rule to enforce project-specific coding conventions.

      For instance, all utility functions in `src/some-project-specific-folder/` should be suffixed with `Util`.

examples:
  - input: |
      // Bad
      function handle() {}

      // Good
      function handleUtil() {}
    output: "Utility functions in this folder should use the `Util` suffix"

metadata:
  priority: normal
  version: 1.0
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ri087) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
