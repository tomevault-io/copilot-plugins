## ai-study-companion

> Never create Markdown (.md) files unless explicitly requested by the user


# Never Create Markdown Files

## **Critical Rule**
- **NEVER create .md files** unless the user explicitly requests it
- This includes README.md, documentation.md, CHANGELOG.md, or any other markdown files
- The user has explicitly stated they do not want MD files created automatically

## **When to Create MD Files**
- ✅ **ONLY** when the user explicitly asks for a markdown file
- ✅ **ONLY** when the user says "create a README" or "write documentation in markdown"
- ❌ **NEVER** create MD files proactively
- ❌ **NEVER** create MD files "for documentation purposes"
- ❌ **NEVER** create MD files as part of feature implementation

## **What to Do Instead**
- Use code comments for inline documentation
- Use TypeScript JSDoc comments for function documentation
- Update existing memory bank files (memory-bank/*.md) when requested
- Use .cursor/rules/*.mdc files for project rules (these are allowed)

## **Exceptions**
- Memory bank files in `memory-bank/` directory (these are maintained as part of the project)
- Rule files in `.cursor/rules/` directory (these use .mdc extension and are part of the rules system)

## **Examples**

❌ **DON'T:**
- "I'll create a README.md to document this feature"
- "Let me add a CHANGELOG.md for tracking changes"
- "I'll create documentation.md to explain the API"

✅ **DO:**
- Add code comments explaining the feature
- Use JSDoc for function documentation
- Update existing memory bank files when user requests it
- Only create MD files when user explicitly asks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MuradAles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
