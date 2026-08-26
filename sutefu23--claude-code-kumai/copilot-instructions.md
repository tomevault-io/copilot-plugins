## claude-code-kumai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Marp presentation project for "「Claude CodeによるAI駆動開発入門」に書けなかったコト" (Deep Dive into Claude Code × AI-Driven Development with the Author). The repository contains Japanese presentation materials about Claude Code, AI-driven development practices, and best practices for using Claude Code effectively.

## Key Technologies

- **Marp**: Markdown Presentation Ecosystem for creating slides from Markdown
- **Custom Theme**: "kouen" theme defined in `.marp/themes/kouen.css`
- **Tailwind CSS**: Integrated via CDN for utility styling

## File Structure

- `index.md`: Main presentation slides (primary content)
- `primary.md`: Introduction/beginner guide to Claude Code
- `SKILL例.md`: Example Skill definition for lint-checker
- `style.css`: Additional custom styles for presentations
- `.marp/themes/kouen.css`: Custom Marp theme definition
- `image/`: Presentation assets (PNG, SVG images)

## Building and Previewing Presentations

### Using Marp CLI

To preview presentations:
```bash
npx @marp-team/marp-cli@latest index.md --preview
npx @marp-team/marp-cli@latest primary.md --preview
```

To build HTML output:
```bash
npx @marp-team/marp-cli@latest index.md -o output/index.html
npx @marp-team/marp-cli@latest primary.md -o output/primary.html
```

To build PDF output:
```bash
npx @marp-team/marp-cli@latest index.md --pdf -o output/index.pdf
npx @marp-team/marp-cli@latest primary.md --pdf -o output/primary.pdf
```

### Using Marp for VS Code

If the Marp for VS Code extension is installed, you can:
1. Open any `.md` file
2. Use "Marp: Open Preview" command
3. Export via "Marp: Export Slide Deck"

## Marp Frontmatter Configuration

All presentation files use this frontmatter structure:
```yaml
---
marp: true
theme: kouen
class: lead
paginate: true
backgroundColor: white
header: "Optional header text"
---
```

## Custom CSS Classes

The kouen theme and style.css provide these utility classes:

- `.columns`: Two-column grid layout (1fr 1fr)
- `.rows`: Two-row grid layout
- `.orange`: Orange text color (#FF5722)
- `.lead`: Larger font size (1.2rem)
- Custom image sizing within columns (max-height: 450px)

## Content Guidelines

### Language
- All content is in Japanese
- Technical terms often use English (Claude Code, MCP, etc.)

### Slide Structure
- Use `---` to separate slides
- Use `<!-- _class: lead -->` for special lead slides
- Headers are colored #0078D7 (blue)

### Images
- Store all images in `image/` directory
- Reference with relative paths: `./image/filename.png`
- Use `![height:XXXpx](./image/...)` for height control
- Use `![height:XXXpx center](./image/...)` for centered images

### Code Blocks
- Use language-specific code fences: ` ```bash`, ` ```javascript`, etc.
- Code has light gray background (#f0f0f0) with rounded corners

## Claude Code Specific Configuration

### Session End Hook
The project has a Stop hook configured to regenerate CLAUDE.md:
```json
"Stop": [{
  "hooks": [{
    "type": "prompt",
    "prompt": "CLAUDE.mdを確認して再生成してください。"
  }]
}]
```

### Permissions
Auto-allowed commands include git operations and GitHub CLI commands.

## Development Workflow

When editing presentations:

1. Modify the relevant `.md` file (index.md or primary.md)
2. Preview changes using Marp CLI or VS Code extension
3. Check that custom CSS classes render correctly
4. Verify Japanese text displays properly
5. Test image paths and sizing
6. Stage and commit changes when satisfied

## Important Notes

- The presentation discusses Claude Code features including:
  - Context window management and optimization
  - Custom slash commands, subagents, hooks, and skills
  - MCP servers (Context7, Playwright, Serena)
  - Spec-driven development approaches
  - Architecture patterns (onion architecture) for AI-driven development

- When adding new content, maintain consistency with existing presentation structure and the "kouen" brand aesthetic (blue headers, orange accents)

- Images should be optimized for presentation display (not too large, appropriate resolution)

- Keep slide content concise - aim for key points with brief explanations

---
> Source: [sutefu23/claude-code-kumai](https://github.com/sutefu23/claude-code-kumai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
