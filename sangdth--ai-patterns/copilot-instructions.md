## deepwiki

> Before starting any implementation, always perform a DeepWiki research phase.


# DeepWiki Research Rule

Before starting any implementation, always perform a DeepWiki research phase.

## Process

1. Identify all key topics/concepts in the request.
2. Use the DeepWiki MCP tools in this order as needed:
   - `read_wiki_structure`: Get the list of relevant documentation topics.
   - `read_wiki_contents`: Retrieve detailed docs for those topics.
   - `ask_question`: Clarify gaps or ambiguities with AI-grounded answers.
3. Summarize findings clearly before writing any code or content.
4. Base all implementation decisions on the research results.
5. If DeepWiki is unavailable, notify the user before proceeding.

This rule applies to all code, documentation, and content work.

## Coordination with Context7

### When to Use DeepWiki vs Context7

**Use DeepWiki when:**

- You need general research on a topic/concept
- You're exploring new technologies or approaches
- You need AI-grounded answers and clarifications
- You're doing broad research before narrowing down

**Use Context7 when:**

- You need specific code examples from libraries
- You're implementing with a known library/framework
- You need official documentation snippets
- You're looking for implementation patterns

### Integrated Research Workflow

1. **DeepWiki Phase**: Use `@deepwiki.mdc` for general research and topic
   exploration
2. **Context7 Phase**: Use `@context7.mdc` for specific library documentation
   and code examples
3. **Implementation**: Combine insights from both research phases

### Avoiding Redundancy

- Use DeepWiki first for broad understanding
- Use Context7 for specific implementation details
- Don't duplicate research phases - coordinate them

---
> Source: [sangdth/ai-patterns](https://github.com/sangdth/ai-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
