## knowledge

> Always-apply markdown structure rules (migrated from .cursorrules)


# Cursor Rules for Knowledge Base

## Markdown File Structure

Every Markdown page should follow this structure:

1. **YAML Frontmatter** (required)
   - Must include `tags` array with relevant topics
   ```yaml
   ---
   tags:
     - Topic1
     - Topic2
   ---
   ```

2. **Main Heading** (required)
   - Single `# Title` at the top after frontmatter

3. **Table of Contents** (required)
   - Must be included after the main heading
   - Use markdown links to section anchors
   - Format:
   ```markdown
   ---
   - [Section 1](#section-1)
   - [Section 2](#section-2)
     - [Subsection](#subsection)
   - [Links](#links)
   ---
   ```

4. **Summary Section** (recommended)
   - Brief overview of the document content
   - Placed after the table of contents

5. **Content Sections**
   - Use `##` for main sections
   - Use `###` for subsections
   - Keep hierarchy consistent

## Formatting Guidelines

- Use tables for comparisons and structured data
- Use code blocks with language specification for code snippets
- Use `<details>` tags for collapsible long code examples
- Include links to external resources where relevant
- Use horizontal rules `---` to separate major sections

## Example Template

```markdown
---
tags:
  - Tag1
  - Tag2
---

# Page Title

---
- [Section 1](#section-1)
- [Section 2](#section-2)
- [Links](#links)
---

## Summary

Brief description of what this document covers.

## Section 1

Content here...

## Section 2

Content here...

## Links

- [Resource Name](url)
```

## When Creating or Editing Pages

- Always check if table of contents exists; add if missing
- Keep table of contents synchronized with actual sections
- Use kebab-case for anchor links (e.g., `#section-name`)
- Maintain consistent indentation in nested lists

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrzejsydor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andrzejsydor)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
