## readme-conventions

> The project's README.md follows specific conventions to maintain consistency and clarity. The documentation is split between README.md (project overview) and CURSOR_TIPS.md (Cursor-specific documentation).

# README Conventions and Guidelines 📚

The project's README.md follows specific conventions to maintain consistency and clarity. The documentation is split between README.md (project overview) and CURSOR_TIPS.md (Cursor-specific documentation).

## Language Requirements

All documentation MUST be written in English. This includes:
- Main content
- Comments
- Examples
- Section headers
- File names and paths

## README.md Structure

1. **Title and Introduction**
   ```markdown
   # Project Title 🚀

   Brief description of what the project does and its main benefits.
   ```

2. **Content Overview**
   ```markdown
   ## 📚 What You'll Find Here

   ### [CURSOR_TIPS.md](mdc:CURSOR_TIPS.md)
   Brief description of what's in CURSOR_TIPS.md
   ```

3. **Rules Section**
   ```markdown
   ## 📋 Rules

   Brief explanation of Cursor rules and their purpose.

   | Rule Name | Purpose | Description |
   |-----------|---------|-------------|
   | [rule-name](mdc:.cursor/rules/rule-name.mdc) | Category | Description |
   ```

4. **Contributing Section**
   ```markdown
   ## 🤝 Contributing

   How others can contribute to the project
   ```

5. **License Section**
   ```markdown
   ## 📝 License

   License information and terms
   ```

## Content Guidelines

1. **General Principles**
   - Keep README.md focused on project overview
   - Move Cursor-specific content to CURSOR_TIPS.md
   - Use clear, professional English
   - Be concise and direct

2. **Formatting**
   - Use **bold** for emphasis
   - Use `code` for technical terms
   - Use emojis frgaUse horizontal rules (---) between main sections

3. **Links and References**
   - Link to CURSOR_TIPS.md for Cursor-specific details
   - Link to rule files in .cursor/rules/
   - Use descriptive link text
   - Example:
   ```markdown
   [Cursor Usage Guide](mdc:CURSOR_TIPS.md) | [Git Conventions](mdc:.cursor/rules/git-conventions.mdc)
   ```

## Examples

✅ Good README Structure:
```markdown
# Project Name 🚀

This project helps developers maximize productivity with Cursor IDE.

## 📚 What You'll Find Here

### [CURSOR_TIPS.md](mdc:CURSOR_TIPS.md)
Comprehensive guide for professional Cursor usage, including:
- Initial setup
- Project customization
- Advanced features

## 📋 Rules

Rules help maintain consistency across the project...

| Rule Name | Purpose | Description |
|-----------|---------|-------------|
| [git-conventions](mdc:.cursor/rules/git-conventions.mdc) | Git Standards | Enforces commit format... |
```

❌ Bad README Structure:
```markdown
# Project

Setup:
1. Install cursor
2. Configure stuff
3. Do things

Rules:
- git stuff
- readme stuff
```

## Maintenance

1. **Regular Updates**
   - Keep both README.md and CURSOR_TIPS.md synchronized
   - Ensure all links between documents work
   - Update rule references when rules change

2. **Review Process**
   - Verify English language usage
   - Check links to rule files
   - Validate table formatting
   - Ensure proper separation of concerns between files

---
> Source: [gifflet/cursor-like-pro](https://github.com/gifflet/cursor-like-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
