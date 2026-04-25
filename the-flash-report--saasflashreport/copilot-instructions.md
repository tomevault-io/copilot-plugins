## saasflashreport

> The AI's memory helps maintain consistency and adapt to specific project needs or user preferences discovered during interactions. It prevents the AI from repeatedly asking for the same information or making suggestions contrary to established patterns.

# AI Memory Management

## Purpose
The AI's memory helps maintain consistency and adapt to specific project needs or user preferences discovered during interactions. It prevents the AI from repeatedly asking for the same information or making suggestions contrary to established patterns.

## Memory Storage Location
All learned project-specific knowledge and preferences should be stored and referenced in the `learned-memories.mdc` file located in `.cursor/rules`.

## When to Update Memory
The AI should update memory when learning:
- **User Preferences**: Coding style preferences, tool choices, workflow preferences
- **Technical Decisions**: Framework versions, architectural choices, deployment methods
- **Project Conventions**: Naming patterns, file organization, custom processes
- **Domain Knowledge**: Business rules, industry standards, regulatory requirements

## Memory Update Process
When new information is learned:

1. **Identify Key Information**: Determine the core piece of knowledge to be stored
2. **Check Existing Memory**: Review `learned-memories.mdc` to see if this information contradicts or updates existing entries
3. **Propose Update**: Suggest an edit to `learned-memories.mdc` to add or modify the relevant information. Keep entries concise and clear
4. **Categorize Properly**: Place information in the appropriate section (User Preferences, Technical Decisions, etc.)

## Using Memory
Before proposing solutions, code changes, or answering questions, the AI should:
- Consult `learned-memories.mdc` to ensure response alignment
- Reference established patterns and preferences
- Avoid contradicting previously learned information
- Build upon established project knowledge

## Memory File Structure
The `learned-memories.mdc` file should follow this structure:

```markdown
# Project Memory

This file stores project-specific knowledge, conventions, and user preferences learned by the AI assistant.

## User Preferences
- [Specific coding style preferences]
- [Tool and framework preferences]
- [Workflow and process preferences]

## Technical Decisions
- [Framework versions and configurations]
- [Architecture and design patterns]
- [Integration and deployment choices]

## Project Conventions
- [Naming conventions and patterns]
- [File organization standards]
- [Code structure guidelines]

## Domain Knowledge
- [Business rules and requirements]
- [Industry standards and regulations]
- [Domain-specific best practices]
```

## Example Memory Update

**Scenario**: User mentions "We use Tailwind v4 for this project, not v3."

**AI Action**:
1. Recognize this as a project-specific technical decision
2. Check `learned-memories.mdc` for existing CSS framework information
3. Propose adding/updating entry in the Technical Decisions section:
   ```markdown
   ## Technical Decisions
   - **CSS Framework**: Tailwind v4 is used. Ensure usage aligns with v4 documentation and practices, noting differences from v3.
   ```

## Memory Maintenance
- Keep entries concise but comprehensive
- Update conflicting information rather than creating duplicates
- Remove outdated information when project requirements change
- Organize entries logically within their respective sections
- Date-stamp major changes when helpful for context

## Integration with Development Workflow
- Reference memory before suggesting code changes
- Use established patterns when creating new features
- Maintain consistency with documented preferences
- Evolve memory as project requirements mature

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/The-Flash-Report) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
