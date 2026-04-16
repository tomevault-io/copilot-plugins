## node-hexagonal-openai

> Creating rules

# Create Rule Usage

DO NOT CREATE mdc files directly, use the script.

DO NOT EDIT mdf files directly, remove and create another with the updated content.

Create the rule content in a md file and use it with the script.

Check carefully this document.

# Script Usage Guide

## Basic Usage

The create_rule.sh script simplifies the creation of Cursor rules by generating properly formatted .mdc files with appropriate metadata.

```bash
.cursor/create_rule.sh <rule_name> <rule_type> <description> [content_file] [title]
```

This script will:
1. Create a new .mdc file in the .cursor/rules directory
2. Configure the rule with the appropriate type settings
3. Add your description and content
4. Format everything according to Cursor's requirements

All rules will be saved to .cursor/rules/<rule_name>.mdc and will be immediately available for use in your Cursor environment.

## Arguments

- **rule_name**: Name of the rule file (without .mdc extension)
- **rule_type**: Type of rule: 'always', 'auto_attached', 'agent_requested', or 'manual'
- **description**: Brief description of the rule
- **content_file**: (Optional) Path to markdown file to use as rule content
- **title**: (Optional) Title of the rule (defaults to rule_name if not provided)

## Rule Types and Format

Rule types map to specific options in the .mdc files:

- **always**: Sets `alwaysApply: true`. Applied to every conversation automatically.
- **auto_attached**: Sets `alwaysApply: false` with `globs: ["*.py"]` (or relevant pattern). Applied when matching files are attached.
- **agent_requested**: Sets `alwaysApply: false` with a `description` field. AI must fetch these rules explicitly.
- **manual**: Sets `alwaysApply: false` with no additional options. User must manually attach.

## Examples

### Basic rule with default title
```bash
.cursor/create_rule.sh python-style always 'Python coding style guidelines'
```

### Rule with content from file
```bash
.cursor/create_rule.sh react-components auto_attached 'React component structure' ./react-rules.md
```

### Rule with custom title (empty content file argument)
```bash
.cursor/create_rule.sh custom-title always 'Rule with custom title' '' 'Custom Title'
```

### Complete rule with content file and custom title
```bash
.cursor/create_rule.sh database-access agent_requested 'Database access patterns' ./db-patterns.md 'Database Patterns'
```

## On finish

Remove any .md file created during the process.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atcastells) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
