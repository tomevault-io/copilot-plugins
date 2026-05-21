## project-structure

> Project structure and organization guidelines for AI-assisted web development


# Project Structure and Organization

This project follows a structured approach to building simple web apps with AI tools.

## Repository Structure

- **apps/**: Complete application examples
  - Each app should be self-contained with its own directory
  - Use descriptive names (e.g., `quiz-app`, `habit-tracker`, `geo-spatial-app-air-quality`)
  - Include README.md for complex apps with setup instructions

- **tutorials/**: Step-by-step learning materials
  - Markdown files with clear instructions
  - Include code examples and best practices
  - Reference specific apps in the `apps/` directory

- **demos/**: Demonstration materials and walkthroughs
  - Organized by topic or tool (e.g., `1-claude-walkthrough`, `2-cursor-walkthrough`)
  - Include migration guides and examples

- **presentation/**: Course materials and visual assets
  - PDFs, images, and HTML presentations
  - Keep assets organized by date or topic

## App Development Standards

### File Organization
- **Single-file apps**: Use `index.html` with embedded CSS and JavaScript
- **Multi-file apps**: Organize as `index.html`, `styles.css`, `script.js`
- **Complex apps**: Use subdirectories for `css/`, `js/`, `assets/`

### Naming Conventions
- Use kebab-case for directories and files
- Use descriptive names that indicate functionality
- Include version numbers for iterations (e.g., `quiz-app-version2`)

### Documentation Requirements
- Every app should have a clear purpose
- Include setup instructions for apps requiring dependencies
- Document any external API requirements
- Provide example data files when applicable

## Key Files to Reference

- [README.md](mdc:README.md) - Main project documentation
- [apps/simple-webpage/index.html](mdc:apps/simple-webpage/index.html) - Basic landing page example
- [apps/quiz-app/index.html](mdc:apps/quiz-app/index.html) - Interactive quiz application
- [tutorials/first-web-app.md](mdc:tutorials/first-web-app.md) - Getting started guide

## Development Workflow

1. **Planning**: Define the problem and requirements
2. **Specification**: Create detailed spec documents
3. **Prototyping**: Build initial version with AI assistance
4. **Implementation**: Refine and polish the application
5. **Documentation**: Update README and add examples

---
> Source: [EnkrateiaLucca/building-apps-with-ai-tools](https://github.com/EnkrateiaLucca/building-apps-with-ai-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
