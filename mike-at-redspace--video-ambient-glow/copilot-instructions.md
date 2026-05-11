## wiki

> Documentation wiki files in Markdown format - git submodule for GitHub wiki


- Use Markdown format for all documentation
- Include code examples with syntax highlighting
- Reference framework integrations: React, Vue, Svelte, Angular
- API documentation should match types in `src/types.ts`
- Include installation instructions, usage examples, configuration options
- Use clear headings and structure
- Link between pages where relevant
- Examples should be copy-paste ready and tested
- Include performance tips and best practices
- Document all public API methods and options
- **Note**: `wiki/` is a git submodule pointing to the GitHub wiki repository
- After editing wiki files, publish with `npm run wiki` (runs `scripts/publish-wiki.sh`)
- Remember to commit the submodule update in main repo: `git add wiki && git commit -m "Update wiki submodule"`
- @file wiki/API-Reference.md
- @file wiki/Examples.md
- @file scripts/publish-wiki.sh

---
> Source: [mike-at-redspace/video-ambient-glow](https://github.com/mike-at-redspace/video-ambient-glow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
