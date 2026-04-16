## svelte-kit-template

> **Activation Mode:** Always On

# RAG Priority Rule

**Activation Mode:** Always On

**Priority:** High

## Primary Directive: Local Documentation First

For all development tasks, you MUST follow this exact order:

### 1. Search Local Documentation First
- **ALWAYS** search and retrieve relevant instructions from `./.rag-docs` directory (all Markdown files)
- Use semantic search to find the most relevant documentation chunks
- Look for official documentation, implementation guides, and best practices

### 2. Implement Based on Local Docs
- If relevant local documentation is found, implement the solution **EXACTLY** as described in the local docs
- Follow the specific patterns, code examples, and configurations provided in the local documentation
- Cite the specific file path from `./.rag-docs` that guided your implementation

### 3. Web Fallback Only When Necessary
- **ONLY** if no matching local content exists for the specific task, continue with web search or general knowledge
- Clearly state when you're falling back to web search: "No local documentation found, searching web..."
- Provide citations for web sources used

## Implementation Guidelines

### When Local Docs Are Found
- Follow the documented patterns precisely
- Reference the specific documentation file that guided your decisions
- Do not deviate from documented approaches unless explicitly asked

### When No Local Docs Exist
- Search for current, official documentation
- Prioritize official sources over tutorials or blog posts
- Document your search process and sources
- Consider adding the discovered information to `./.rag-docs` for future reference

## Quality Assurance

### Before Implementing
- Verify the local documentation is relevant to the current task
- Confirm the approach matches the project's existing patterns

### After Implementing
- Verify the implementation follows the local documentation exactly
- Note any deviations and explain why they were necessary

## Rationale

This approach ensures:
- **Consistency:** All implementations follow documented patterns
- **Reliability:** Reduces bugs from outdated or incorrect implementations  
- **Efficiency:** Minimizes iterations by using proven solutions
- **Knowledge Building:** Creates a comprehensive local knowledge base

## Examples

**Good:** "Found relevant documentation in `./.rag-docs/sveltekit/routing.md` - implementing route guards as documented in section 3.2"

**Good:** "No local documentation found for this specific use case. Searching web for current SvelteKit file upload patterns..."

**Bad:** Implementing without checking local docs first
**Bad:** Ignoring local doc patterns in favor of web alternatives

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/malinjr07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
