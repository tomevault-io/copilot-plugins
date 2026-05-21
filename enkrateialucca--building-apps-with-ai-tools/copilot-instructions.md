## ai-development-workflow

> AI-assisted development workflow and best practices


# AI-Assisted Development Workflow

This project emphasizes using AI tools (Claude and Cursor) effectively for web development.

## AI Tools Integration

### Claude Usage
- **Problem Definition**: Use Claude for initial problem analysis and requirements gathering
- **Research**: Leverage Claude's knowledge for API documentation and best practices
- **Flow Diagramming**: Use Claude's mermaid capabilities for visualizing app flows
- **Code Generation**: Generate initial prototypes and boilerplate code
- **Code Review**: Use Claude for code quality assessment and improvements

### Cursor IDE Features
- **Spec Documents**: Use @ symbol to reference specification files
- **File Context**: Reference multiple files with @filename for better context
- **Cursor Rules**: Leverage .cursor/rules for consistent AI behavior
- **AI Chat**: Use Cursor's AI chat for quick code assistance and debugging

## Development Process

### 1. Planning Phase
```
Problem Definition → Research → Flow Diagramming → Specification
```

**Key Activities:**
- Define the problem clearly with Claude
- Research APIs and technologies needed
- Create flow diagrams using mermaid
- Write detailed specification documents

### 2. Specification Phase
- Create detailed spec documents in markdown
- Include user stories and acceptance criteria
- Define data structures and API requirements
- Set up .cursorrules for the project

### 3. Development Phase
- Start with Claude Artifacts/Projects for prototyping
- Use Cursor for implementation and refinement
- Follow the single-file approach for simple apps
- Implement proper error handling and accessibility

### 4. Testing and Refinement
- Test functionality across different browsers
- Validate accessibility compliance
- Optimize performance and user experience
- Update documentation as needed

## Best Practices

### AI Prompting
- Be specific about requirements and constraints
- Provide context about the project structure
- Ask for explanations of generated code
- Request multiple implementation approaches when appropriate

### Code Quality
- Always review AI-generated code
- Ensure proper error handling
- Follow accessibility guidelines
- Test thoroughly before deployment

### Documentation
- Document AI-assisted decisions
- Include setup instructions for complex apps
- Provide examples and use cases
- Keep README files updated

## Key Resources

- [Claude Documentation](https://docs.anthropic.com/claude/)
- [Cursor Documentation](https://docs.cursor.com/)
- [Claude Tokenizer Tool](https://claude-tokenizer.vercel.app/)
- [Cursor Rules Examples](https://github.com/PatrickJS/awesome-cursorrules)

## Example Workflow

1. **Problem**: "I need a quiz application"
2. **Research**: Use Claude to research quiz app patterns and best practices
3. **Specification**: Create [apps/quiz-app/docs/spec-doc.md](mdc:apps/quiz-app/docs/spec-doc.md)
4. **Prototype**: Use Claude to generate initial HTML structure
5. **Implementation**: Use Cursor to refine and add interactivity
6. **Testing**: Validate functionality and accessibility
7. **Documentation**: Update README with setup instructions

---
> Source: [EnkrateiaLucca/building-apps-with-ai-tools](https://github.com/EnkrateiaLucca/building-apps-with-ai-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
