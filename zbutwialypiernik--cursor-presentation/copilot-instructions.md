## cursor-presentation

> 1. **General/Language Rules First** - Place broad programming principles and language-specific rules at the top

# Meta Rules for Writing Cursor Rules

## Rule Organization Priority
1. **General/Language Rules First** - Place broad programming principles and language-specific rules at the top
2. **Project-Specific Rules** - Follow with rules for specific libraries, frameworks, or project patterns
3. **File-Specific Rules** - End with retiic file types or paths

## Requirecture
```markdown
---
description:ne description of what the rule enforces
globs: path/to/files/*.ext, other/path/**/*
alwaysApply: boolean
---
- **Main Points in Bold**
  - Sub-points with details
  - Examples and explanations
```

## Rule Content Guidelines
- **Keep Rules Concise** - Write shortest possible rules that still convey requirements
- **Make Rules Generic** - Create broadly applicable rules, not ones based on single examples
- **Minimize Code Examples** - Use code only when text cannot adequately describe the rule
- **Avoid Visual Clutter** - No emoticons, excessive formatting, or decorative elements
- **Start with High-Level Overview** - Begin with the core principle
- **Include Actionable Requirements** - Make rules implementable across similar scenarios

## File References
- Use `@filename` to reference files
- Example: @prisma.mdc for rule references  
- Example: @schema.prisma for code references

## Code Examples (Use Sparingly)
- Only include when text description is insufficient
- Avoid DO/DON'T examples unless absolutely critical
- Keep examples minimal and focused

## Rule Maintenance
- **Update When Patterns Emerge** - Add rules for recurring issues
- **Remove Outdated Patterns** - Clean up obsolete requirements  
- **Cross-Reference Related Rules** - Keep rules DRY by linking to others
- **Add Examples from Actual Codebase** - Use real code over hypothetical

## Best Practices
- Use bullet points for clarity
- Keep descriptions concise
- Focus on broad applicability over specific cases
- Use consistent formatting across rules
- Prioritize text descriptions over code examples

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZbutwialyPiernik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ZbutwialyPiernik)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
