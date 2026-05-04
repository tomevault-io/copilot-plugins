## claude-code-training

> This project contains comprehensive training materials for teaching Claude Code to professional software developers. The course emphasizes practical, hands-on learning with real-world scenarios.

# Claude Code Training Course Development

This project contains comprehensive training materials for teaching Claude Code to professional software developers. The course emphasizes practical, hands-on learning with real-world scenarios.

## Project Structure & Standards

### Core Course Materials
- **`slides.md`**: Main Slidev presentation with progressive learning path
- **`lab_handout.md`**: Six progressive hands-on labs (Lab 0-6) for students
- **`instructor-guide.md`**: Detailed instructor guidance with specific demo scenarios
- **`exercises/`**: Collection of real-world sample projects across multiple languages

### Sample Projects Architecture
- **Python Projects**:
  - `weather-app`: Clean Flask application for feature addition exercises
  - `flask-api`: Intentionally poorly-written API for refactoring demonstrations
- **JavaScript/TypeScript**:
  - `lyrics-trainer`: Interactive web app with TypeScript and comprehensive test suite
- **Java Projects**:
  - `certificate-service`: Modern Spring Boot application with PDF generation
  - `shopping-service`: Legacy Grails/Groovy application for modernization exercises

### Custom Commands Integration
- **`custom-commands.md`**: Comprehensive showcase of workflow automation commands
- **`commands/`**: Ready-to-use command files for immediate copying to `~/.claude/commands/`
- Commands cover Java modernization, Spring scaffolding, security reviews, documentation generation, and onboarding automation

## Development Workflow

### Presentation Development
1. **Run development server:** `pnpm dev`
2. **Build for production:** `pnpm build` 
3. **Export to PDF:** `pnpm export`
4. **Export to PowerPoint:** `pnpm export --format pptx --output claude-code-training.pptx`

**Note**: For exporting presentations, you may need to install Playwright first:
```bash
pnpm install -D playwright-chromium
pnpm exec playwright install
```

### Content Development Standards
- **Branch-First Approach**: All exercises emphasize creating branches before changes
- **Progressive Complexity**: Labs build from simple to advanced workflows
- **Real-World Focus**: Use actual project scenarios, not contrived examples
- **Multi-Language Support**: Ensure examples work across Python, JavaScript, and Java

### Lab Design Principles
- **Lab 0**: Project creation from scratch (lyrics display app - origins of lyrics-trainer)
- **Labs 1-3**: Foundation skills (exploration, testing, documentation)
- **Labs 4-6**: Advanced workflows (refactoring, Plan Mode, custom commands)

### Custom Commands Teaching Integration
- **Demonstration**: Showcase commands during Lab 6 advanced workflows
- **Hands-on Practice**: Students copy and customize commands for their workflows  
- **Real-world Application**: Use Java commands with certificate-service, security commands with vulnerable examples
- **Team Sharing**: Demonstrate how teams can standardize workflows through shared commands

## Teaching Philosophy

### Core Principles
- **Practical First**: Get students productive immediately
- **Safety Through Branching**: Always demonstrate branch-first workflow
- **Real Scenarios**: Use diverse, realistic codebases (modern to legacy)
- **Professional Workflows**: Emphasize enterprise development practices

### Key Messaging
- Claude Code creates complete projects from scratch
- AI assistance accelerates understanding of unfamiliar codebases
- Branch-first workflow enables fearless experimentation
- Tool adapts to any language/framework combination

## Content Guidelines

### Exercise Design
- Start each significant change with: "Create a new branch for this exercise"
- Use specific project names and concrete examples
- Include realistic time estimates (Lab 0: 20min, others: 15-60min)
- Provide both instructor demos and student hands-on activities

### Prompt Engineering Examples
- Always include complete, copy-pasteable prompts in code blocks
- Demonstrate iterative refinement techniques
- Show context-aware instructions that reference specific files/frameworks
- Include error recovery and troubleshooting scenarios

## Technical Requirements

### Development Environment
- Node.js and pnpm for presentation
- Python 3.x for Flask applications
- Java 17+ for Spring Boot/Grails projects
- TypeScript for modern web development

### Student Prerequisites
- Claude Code installed and configured
- Git workflow familiarity
- Multi-language development environment setup
- Basic understanding of web development concepts

---
> Source: [kousen/claude-code-training](https://github.com/kousen/claude-code-training) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
