## badge

> Standards for placing and organizing Cursor rule files


# Cursor Rules Location

Rules for placing and organizing Cursor rule files in the repository. Rules are automatically included in AI responses when files match the specified glob patterns.

<rule>
name: cursor_rules_location
description: Standards for placing and organizing Cursor rule files. Auto-included for .mdc files in .cursor/rules.
globs: [".cursor/rules/*.mdc"]

filters:
  # Match any .mdc files
  - type: file_extension
    pattern: "\\.mdc$"
  # Match files that look like Cursor rules
  - type: content
    pattern: "(?s)<rule>.*?</rule>"
  # Match file creation events
  - type: event
    pattern: "file_create"

actions:
  - type: suggest
    message: |
      When creating Cursor rules:

      1. File Header (Frontmatter):
         ```markdown
         ---
         description: Brief description of what the rule does
         globs: pattern/to/match/**/*.{ts,tsx}
         alwaysApply: true/false
         ---
         ```
         The `globs` field is crucial - it determines when this rule is automatically included in AI responses.
         When a file matches the glob pattern, the AI will automatically consider this rule in its responses.
         Note: In the header, use plain glob patterns without quotes or arrays.

      2. Rule Structure:
         ```markdown
         # Rule Title

         Brief description of the rule's purpose and when it's auto-included.

         <rule>
         name: rule_name
         description: Detailed description. Include "Auto-included for..." if applicable
         globs: ["pattern/to/match/**/*.{ts,tsx}"]  # Inside rule tag, use array format
         filters:
           - type: file_extension
             pattern: "pattern"
           # Add other relevant filters
         actions:
           - type: suggest
             message: |
               Your rule content here
         examples:
           - input: |
               // Bad example
               // Good example
             output: "Description of the example"
         metadata:
           priority: high
           version: 1.0
         </rule>
         ```

      3. File Location:
         Always place rule files in PROJECT_ROOT/.cursor/rules/:
         ```
         PROJECT_ROOT/
         ├── .cursor/
         │   └── rules/
         │       ├── your-rule-name.mdc
         │       └── ...
         └── ...
         ```

      4. Naming Convention:
         - Use kebab-case for filenames (e.g., backend-standards.mdc)
         - Always use .mdc extension
         - Make names descriptive of the rule's purpose

      5. Never place rule files:
         - In the project root
         - In subdirectories outside .cursor/rules
         - In any other location

      6. UPDATE THE AVAILABLE RULES FILE:
         Whenever creating a new rule, always update the .cursor/rules/available-rules.mdc file:
         - Add your new rule to the <available_instructions> section
         - Use the format: rule-name: Brief description of what the rule does
         - Keep the list alphabetically sorted

examples:
  - input: |
      # Bad: Header with array and quotes
      ---
      description: My rule
      globs: ["app/**/*.ts"]
      ---

      # Good: Header with plain glob pattern
      ---
      description: My rule
      globs: app/**/*.ts
      ---
    output: "Use plain glob patterns in the header frontmatter"

  - input: |
      # Bad: Missing auto-inclusion info and incorrect structure
      ---
      description: My rule
      globs: app/**/*.ts
      ---
      # My Rule
      <rule>...</rule>

      # Good: Complete structure with auto-inclusion info
      ---
      description: Standards for backend functions
      globs: app/controllers/**/*.ts
      ---
      # Backend Standards

      Standards for backend functions. Auto-included for controller files.

      <rule>
      name: backend_standards
      description: Standards for backend functions. Auto-included for controller files.
      globs: ["app/controllers/**/*.ts"]
      ...
      </rule>
    output: "Correctly structured rule file with auto-inclusion information"

  - input: |
      # Bad: Not updating the available-rules.mdc file
      [Create a new rule without updating available-rules.mdc]

      # Good: Updating available-rules.mdc
      [Create a new rule]
      [Open .cursor/rules/available-rules.mdc]
      [Add the new rule to the <available_instructions> section]
      ```
      <available_instructions>
      ...
      my-new-rule: Description of my new rule
      ...
      </available_instructions>
      ```
    output: "Always update available-rules.mdc when creating a new rule"

metadata:
  priority: high
  version: 1.0
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/La-404-Devinci) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
