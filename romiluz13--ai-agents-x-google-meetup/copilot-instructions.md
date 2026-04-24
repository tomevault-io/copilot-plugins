## ai-agents-x-google-meetup

> This repository contains resources from our community meetup. These guidelines help maintain consistency when contributing or using these resources.

# AI Agents Community Repository

This repository contains resources from our community meetup. These guidelines help maintain consistency when contributing or using these resources.

## Design Log Methodology

### Before Making Changes
1. **Check design logs** in `design-log/` for existing designs and implementation notes
2. **For new features**: Create design log first, get approval, then implement
3. **Read related design logs** to understand context and constraints

### When Creating Design Logs
1. **Structure**: Background → Problem → Questions and Answers → Design → Implementation Plan → Examples → Trade-offs
2. **Be specific**: Include file paths, type signatures, validation rules
3. **Show examples**: Use ✅/❌ for good/bad patterns, include realistic code
4. **Explain why**: Don't just describe what, explain rationale and trade-offs
5. **Ask Questions (in the file)**: For anything that is not clear, or missing information
6. **When answering question**: keep the questions, just add answers
7. **Be brief**: write short explanations and only what most relevant
8. **Draw Diagrams**: Use mermaid inline diagrams when it makes sense
9. **Define verification criteria**: how do we know the implementation solves the original problem

### When Implementing
1. **Follow the implementation plan** phases from the design log
2. **Write tests first** or update existing tests to match new behavior
3. **Do not Update design log** initial section once implementation started
4. **Append design log** with "Implementation Results" section as you go
5. **Document deviations**: Explain why implementation differs from design
6. **Run tests**: Include test results (X/Y passing) in implementation notes
7. **After Implementation** add a summary of deviations from original design

### When Answering Questions
1. **Reference design logs** by number when relevant (e.g., "See Design Log #50")
2. **Use repository terminology**: Adapt to this repository's conventions
3. **Show examples**: Include code examples when helpful
4. **Consider consistency**: Maintain consistency with existing patterns

## Code Research (octocode Pattern)

When research is needed:
1. **Research before implementing** - Use external and local search
2. **Document findings** - Include research results in design logs
3. **Reference sources** - Cite where information came from
4. **Explain choices** - Why approaches were selected

## Repository Structure

- `content/` - Original presentations, transcripts, summaries
- `guides/` - Implementation guides for each methodology/tool
- `templates/` - Reusable templates
- `examples/` - Real-world examples
- `resources/` - Links, comparisons, further reading
- `docs/` - Documentation for AI agents

## File Organization

- Keep files focused and under 500 lines
- Use clear, descriptive names
- Follow existing structure and patterns
- Maintain consistency across similar files

## Documentation Standards

- Use Markdown for all documentation
- Include code examples where helpful
- Use Mermaid diagrams for workflows
- Keep documentation up-to-date
- Reference related documents

## Best Practices

1. **Read before writing** - Check existing content first
2. **Design before developing** - Create design logs for significant changes
3. **Preserve history** - Don't modify past design logs, append instead
4. **Be specific** - Include details, examples, and verification criteria
5. **Ask questions** - When information is unclear

## See Also

- [Design Log Methodology Guide](guides/design-log-methodology/)
- [LLM Instructions](docs/llm-instructions.md)
- [Reference Implementation](content/reference-implementations/design-log.mdc)

---
> Source: [romiluz13/ai-agents-x-google-meetup](https://github.com/romiluz13/ai-agents-x-google-meetup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
