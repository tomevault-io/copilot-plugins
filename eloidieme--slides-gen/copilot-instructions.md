## slides-gen

> **slides-gen** is an AI-powered CLI tool for automated slide deck generation from markdown content. It uses Marp (Markdown Presentation Ecosystem) as the core rendering engine and integrates with Claude Code and MCP servers for intelligent content generation and enhancement.

# CLAUDE.md - Project Context for AI Assistants

## Project Overview

**slides-gen** is an AI-powered CLI tool for automated slide deck generation from markdown content. It uses Marp (Markdown Presentation Ecosystem) as the core rendering engine and integrates with Claude Code and MCP servers for intelligent content generation and enhancement.

## Technology Stack

- **Language**: TypeScript (Node.js runtime)
- **Framework**: Marp (via @marp-team/marp-cli and @marp-team/marp-core)
- **CLI**: Commander.js
- **Testing**: Vitest
- **Build**: TypeScript Compiler (tsc)
- **Package Manager**: npm

## Project Structure

```
slides-gen/
├── src/                    # Source code
│   ├── cli/               # CLI commands and entry point
│   ├── core/              # Core business logic
│   ├── slide-types/       # Slide type implementations
│   ├── mcp/               # MCP integrations
│   ├── types/             # TypeScript type definitions
│   └── utils/             # Utility functions
├── templates/             # Default templates and themes
├── tests/                 # Test files
├── docs/                  # Documentation
└── tickets/               # MVP implementation tickets
```

## Core Concepts

### Content Flow

1. **Analysis**: Read and parse markdown files with frontmatter
2. **Planning**: Generate slide structure based on content
3. **Generation**: Convert plan to Marp-compatible markdown
4. **Compilation**: Use Marp CLI to render HTML/PDF/PPTX

### Slide Types (MVP)

- **title**: Title slide with optional subtitle/author
- **content**: Standard bullet points and text
- **section**: Section divider slides
- **code**: Syntax-highlighted code blocks
- **diagram**: Mermaid diagrams

### Configuration

Projects use a `config.yaml` file for presentation settings (theme, aspect ratio, output formats, etc.) and per-file frontmatter for slide-specific overrides.

## Development Guidelines

### Code Style

- **TypeScript**: Strict mode enabled
- **Formatting**: Prettier with 2-space indentation
- **Linting**: ESLint with TypeScript rules
- **Naming**: camelCase for variables/functions, PascalCase for classes/types

### Testing Approach (TDD)

1. **Red**: Write failing test first
2. **Green**: Implement minimum code to pass
3. **Refactor**: Clean up while keeping tests green
4. **Repeat**: For each ticket/feature

**Test Structure**:
- Unit tests in `tests/unit/`
- Integration tests in `tests/integration/`
- Fixtures in `tests/fixtures/`
- Minimum 80% coverage target

### Git Workflow

- **Main branch**: `main` (protected, requires PR)
- **Feature branches**: `feature/ticket-X-short-description`
- **Commit format**: Conventional Commits
  - `feat: add code slide type generator`
  - `fix: handle empty markdown files`
  - `test: add unit tests for analyzer`
  - `docs: update configuration guide`

### Ticket Implementation Protocol

See `TICKET_PROTOCOL.md` for the detailed TDD implementation process.

## File Naming Conventions

- **Source files**: `kebab-case.ts` (e.g., `content-analyzer.ts`)
- **Test files**: `kebab-case.test.ts` (e.g., `content-analyzer.test.ts`)
- **Type files**: `kebab-case.ts` (e.g., `slide-config.ts`)
- **Classes**: Match filename in PascalCase (e.g., `ContentAnalyzer` in `content-analyzer.ts`)

## Key Design Decisions

### Why Marp?

- Markdown-native (perfect for AI generation)
- Multiple output formats (HTML, PDF, PPTX)
- Excellent theme system
- Active maintenance
- CLI-friendly

### Why TypeScript?

- Type safety reduces bugs
- Better IDE support for contributors
- Great async/await for MCP calls
- Good ecosystem for CLI tools

### Why Vitest?

- Fast (Vite-powered)
- Modern (ESM, TypeScript out of the box)
- Good developer experience
- Compatible with Jest ecosystem

## Common Tasks

### Setup Development Environment

```bash
npm install
npm run build
npm run test
```

### Run CLI in Development

```bash
npm run dev -- init my-slides
npm run dev -- generate ./content
```

### Run Tests

```bash
npm test                 # Run all tests
npm test -- analyzer     # Run specific test file
npm run test:watch      # Watch mode
npm run test:coverage   # Coverage report
```

### Build for Production

```bash
npm run build           # Compile TypeScript
npm run lint            # Check code style
npm run format          # Format code
```

### Create New Ticket

See `tickets/` directory for examples. Use the template in `TICKET_PROTOCOL.md`.

## Dependencies

### Production

- `@marp-team/marp-cli`: Marp command-line interface
- `@marp-team/marp-core`: Core Marp engine
- `commander`: CLI framework
- `yaml`: YAML parsing
- `gray-matter`: Frontmatter parsing
- `chalk`: Terminal colors
- `ora`: Loading spinners

### Development

- `typescript`: TypeScript compiler
- `@types/node`: Node.js types
- `vitest`: Test framework
- `tsx`: TypeScript execution
- `eslint`: Linting
- `prettier`: Code formatting

## Environment Requirements

- **Node.js**: >= 18.0.0 (for native fetch, ESM support)
- **npm**: >= 9.0.0
- **OS**: macOS, Linux, Windows (via WSL recommended)

## MCP Integration Notes

### Image Provider (Future)

The tool will support MCP servers for image search:

```typescript
// Example MCP call
const images = await mcpClient.callTool('search_images', {
  query: 'professional office',
  count: 1
});
```

### Graceful Degradation

If MCP server is unavailable:
- Use placeholder images
- Log warning (not error)
- Continue generation

## Performance Targets

- **Analysis**: < 100ms per markdown file
- **Generation**: < 500ms for typical presentation (10-20 slides)
- **Compilation**: Dependent on Marp CLI (typically 1-3s for PDF)
- **Total**: < 5s for complete workflow (content → PDF)

## Error Handling Philosophy

- **User errors**: Clear, actionable messages with suggestions
- **System errors**: Graceful degradation where possible
- **Validation**: Fail fast before expensive operations
- **Logging**: Structured logs for debugging (use debug levels)

## Security Considerations

- **Input validation**: All markdown input sanitized
- **Path traversal**: Validate all file paths stay within project
- **Code execution**: No eval() or arbitrary code execution
- **MCP**: Only allow configured, trusted MCP servers

## Documentation Standards

### Code Comments

- Use JSDoc for public APIs
- Explain "why" not "what" in complex logic
- Keep comments updated with code

### README Updates

When adding features:
1. Update main README.md
2. Add/update relevant docs/ file
3. Update examples if needed

## MVP Scope Reminder

**In Scope (MVP)**:
- CLI with init, generate, preview, validate commands
- 5 slide types: title, content, section, code, diagram
- Mermaid diagrams
- Syntax highlighting
- HTML and PDF output
- 3 built-in themes
- YAML config support

**Out of Scope (MVP)**:
- PPTX export (post-MVP)
- MCP image integration (post-MVP)
- Watch mode (post-MVP)
- Interactive quiz slides (future)
- Speaker notes (future)

## Helpful Context for AI Assistants

### When Implementing Features

1. Check `spec.md` for technical requirements
2. Read relevant ticket in `tickets/` directory
3. Follow TDD protocol in `TICKET_PROTOCOL.md`
4. Write tests first (Red-Green-Refactor)
5. Update documentation if API changes

### When Fixing Bugs

1. Write failing test that reproduces bug
2. Fix minimum code to pass test
3. Check for similar issues elsewhere
4. Update tests if edge case was missed

### When Reviewing Code

- Check test coverage (should be high for core logic)
- Verify TypeScript types are specific (no `any`)
- Ensure error messages are user-friendly
- Look for security issues (path traversal, injection)
- Verify follows project conventions

## Troubleshooting Common Issues

### Marp CLI Not Found

Ensure `@marp-team/marp-cli` is installed as dependency, not devDependency.

### TypeScript Errors in Tests

Check `tsconfig.json` includes test files and has correct `types` array.

### MCP Connection Failures

These should be non-fatal. Check graceful degradation is working.

### Mermaid Rendering Issues

Marp has built-in Mermaid support. Ensure using correct fence:

```markdown
\`\`\`mermaid
graph TD
  A --> B
\`\`\`
```

## References

- **Spec**: See `spec.md` for complete technical specification
- **Protocol**: See `TICKET_PROTOCOL.md` for implementation workflow
- **Tickets**: See `tickets/` for MVP implementation tasks
- **Marp Docs**: https://marpit.marp.app/
- **Mermaid Docs**: https://mermaid.js.org/

## Project Goals

1. **Simple**: Minimal setup, intuitive CLI
2. **Fast**: Generate slides in seconds
3. **Flexible**: Support various content types and styles
4. **AI-Native**: Designed for AI-assisted content generation
5. **Terminal-First**: Full functionality without GUI

## Contact & Contribution

- **Issues**: GitHub Issues for bugs and features
- **PRs**: Welcome! Follow TDD protocol and coding standards
- **Discussions**: GitHub Discussions for questions

---

**Last Updated**: 2026-01-04
**Version**: 0.1.0-alpha
**Status**: Initial development

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eloidieme)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/eloidieme)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
