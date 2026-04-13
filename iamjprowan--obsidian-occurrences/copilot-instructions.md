## obsidian-occurrences

> This document is designed to clarify on what patterns current match the intended architecture. When suggesting new code these patterns should be replicated as close as possible, but should not be considered entirely dogmatic.

# Development Guidelines for Cursor Agents

This document is designed to clarify on what patterns current match the intended architecture. When suggesting new code these patterns should be replicated as close as possible, but should not be considered entirely dogmatic.

If in the process of building a new feature an alternative pattern is worth considering we should still explore it and in the case we decide to proceed you should recommend updates to this file that better match the new approach

## Core Principles

- **Use native Obsidian Plugin API as much as possible** - Always check `node_modules/obsidian/obsidian.d.ts` for complete type definitions and prefer the use of these over building net new. When building new prefer to extend existing Obsidian classes when possible.
- **Attempt to match the look and feel of Obsidian** - Prefer to leverage existing Obsidian components, CSS classes, and variables. Limit the amount of additional CSS created. When creating new UI elements attempt to style in a manner that matches the existing UI
- **Maintain mobile app compatibility** - This plugin will be used in mobile as often as desktop. Do not recommend importing additional node modules as that will break the ability to load the plugin in mobile. Consider how component design will function on a device

## Development Context

**Single User Environment**: This plugin is currently used by only one developer. Breaking changes, experimental approaches, and direct migrations are acceptable since:

- Git rollback is always available for recovery
- No external users to impact with breaking changes
- Rapid iteration and testing is preferred over extensive safety measures
- The developer can handle debugging and fixing issues directly

When suggesting approaches, prefer:

- Direct implementation over overly cautious phased approaches
- "Move fast and fix issues" over "prevent all possible issues"
- Testing new approaches quickly rather than extensive planning
- Assuming the developer can debug and resolve problems

## Additional Requirements and Conventions

- All public methods should have JS docs
- Methods should be explicit in their purpose and avoid side effects that aren't obvious based on the name
- Event handling function should live directly in the registerEvents callback
- file.path is the only reliable unique identifier
- if file.path changes the corresponding item should be removed from memory and then added back with the new path index
- Each directory should have a clear single responsibility (data stores, UI components, form logic, etc.)

## Obsidian Plugin Guidelines Compliance

This plugin follows the [Obsidian Plugin Guidelines](https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines). Key standards:

### Security
- **Never use `innerHTML`, `outerHTML`, or `insertAdjacentHTML`** for user-generated content
- Use Obsidian's safe DOM helpers: `createEl()`, `createSpan()`, `createDiv()`, etc.
- Always construct DOM elements safely to prevent XSS vulnerabilities

### UI Text and Styling
- **Use sentence case** for all UI text (only first word and proper nouns capitalized)
- **Avoid HTML heading tags** (`<h1>`, `<h2>`, etc.) in settings - use `setHeading()` if headings are needed
- **Omit headings** for single-section settings tabs
- **Avoid "Settings" in headings** - context is already within settings
- Proper nouns (like "Occurrence", "Occurrences", "View") should be capitalized

### Console Logging
- **Minimize console logging** - remove unnecessary `console.log`, `console.info`, `console.warn` statements
- **Keep only essential error logs** in catch blocks for debugging
- Clean developer console improves user experience

### Code Organization
- Use `this.app` instead of global `app` object for future compatibility
- Use `onLayoutReady()` to defer non-essential operations during plugin load
- Use `registerEvent()` and `addCommand()` for automatic resource cleanup

### Metadata and Licensing
- Ensure LICENSE file matches package.json license field
- Include proper author information in LICENSE, package.json, and manifest.json
- Provide descriptive manifest.json description for Community Plugins directory

## Documentation Maintenance

This repository follows a documentation structure aligned with user and developer needs:

### Documentation Structure

- **README.md** - User-focused documentation (installation, usage, features, troubleshooting)
- **devlog/** - Historical development decisions, considerations, and architectural choices
- **docs/** - Detailed development documentation (setup, architecture, contributing)

### When to Update Documentation

**Always update when:**
- Adding new features or functionality → Update README.md (Features/Usage sections) and create devlog entry
- Changing user-facing behavior → Update README.md (Usage/Configuration sections)
- Making architectural decisions → Create or update devlog entry
- Changing development setup → Update `docs/development.md`
- Modifying code structure → Update `docs/architecture.md` if significant

**Consider updating when:**
- Refactoring code → Update devlog if pattern changes significantly
- Fixing bugs → Update README troubleshooting if user-visible
- Adding new dependencies → Update `docs/development.md` prerequisites

### Documentation Update Guidelines

1. **README.md Updates:**
   - Keep user-focused (avoid development details)
   - Update Features section when adding capabilities
   - Update Usage/Configuration when behavior changes
   - Add troubleshooting entries for common issues
   - Keep installation instructions current

2. **Devlog Entries:**
   - Create new entry for significant architectural decisions
   - Include attribution section with developer name and AI model
   - Document context, considerations, and rationale
   - Include Decision Log section tracking all decisions with:
     - Decision description
     - Rationale
     - Attribution (User/AI Model)
     - Date
   - Include implementation details and future considerations
   - Use format: `YYYY-MM-DD-description.md`
   - Update `devlog/README.md` index when adding entries

3. **Docs Directory Updates:**
   - `docs/development.md` - Update when setup process changes
   - `docs/architecture.md` - Update when major structural changes occur
   - `docs/contributing.md` - Update when development patterns change
   - Keep documentation index (`docs/README.md`) current

4. **This File (.cursorrules.md):**
   - Update when patterns or conventions change
   - Document new architectural decisions that affect code style
   - Keep documentation maintenance section current

### Documentation Update Checklist

When making code changes, consider:
- [ ] Does this change user-visible behavior? → Update README.md
- [ ] Is this a significant architectural decision? → Create devlog entry
- [ ] Does this change how developers work? → Update docs/development.md
- [ ] Does this change code structure? → Update docs/architecture.md
- [ ] Does this introduce new patterns? → Update .cursorrules.md

**Note**: In a single-user environment, documentation can be updated iteratively. It's acceptable to make code changes first and update documentation in a follow-up, but documentation should not be neglected entirely.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamJpRowan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
