## mkdocs-navigation

> When updating the navigation in [mkdocs.yml](mdc:mkdocs.yml), follow these standards:

# MkDocs Navigation Standards

When updating the navigation in [mkdocs.yml](mdc:mkdocs.yml), follow these standards:

## Talk Navigation Format
For talks in the navigation, use the format: `"Topic - Speaker Name (Company)": talks/filename.md`

### Examples:
```yaml
- "Coding Agents - Nik Pash (Cline)": talks/rag-is-dead-cline-nik.md
- "RAG Anti-patterns - Skylar Payne": talks/rag-antipatterns-skylar-payne.md
- "Semantic Search - Will Bryk (Exa)": talks/semantic-search-exa-will-bryk.md
```

## Navigation Organization
Organize talks into logical chapters and subsections:

### Chapter 5 Structure:
```yaml
- "Chapter 5: User Experience":
  - "Coding Agents":
    - "Coding Agents - Nik Pash (Cline)": talks/rag-is-dead-cline-nik.md
    - "SWE-Bench Agent - Colin Flaherty (Augment)": talks/colin-rag-agents.md
  - "Document Processing":
    - "Document Parsing - Adit (Reducto)": talks/reducto-docs-adit.md
    - "Browser RAG - Michael (OpenBB)": talks/rag-without-apis-browser-michael-struwig.md
  - "Search Technologies":
    - "Semantic Search - Will Bryk (Exa)": talks/semantic-search-exa-will-bryk.md
    - "RAG Anti-patterns - Skylar Payne": talks/rag-antipatterns-skylar-payne.md
```

## Guidelines
- Keep navigation titles shorter than full article titles for better UX
- Include company names in parentheses when available
- Group related talks into logical subsections
- Maintain consistent formatting across all talk entries
- Use descriptive topic names that indicate the main focus

Reference [AGENTS.md](mdc:docs/talks/AGENTS.md) for complete talk organization guidelines.

---
> Source: [jxnl/systematically-improving-rag](https://github.com/jxnl/systematically-improving-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
