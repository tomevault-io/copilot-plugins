## ai

> Documentation standards for README files and project documentation


#### Documentation Standards

##### README File Length Limit

**Maximum Length:**
- README.md files should **never exceed 400 lines** (not characters)
- Keep content concise, scannable, and focused on essential information
- Remove verbose explanations, redundant examples, and excessive prose

**When Additional Context is Needed:**

If a README would exceed 400 lines with necessary information, use these strategies:

1. **Split into separate markdown files** - Create focused docs:
   - `DEPLOYMENT.md` - Deployment procedures, CI/CD, release processes
   - `DEVELOPMENT.md` - Development setup, local workflows, debugging
   - `TROUBLESHOOTING.md` - Common issues, error messages, solutions
   - `ARCHITECTURE.md` - System design, patterns, technical decisions
   - `API.md` - API documentation, endpoints, examples
   - `CONTRIBUTING.md` - Contribution guidelines, code standards

2. **Use visual diagrams** - Replace lengthy text explanations with:
   - **Mermaid sequence diagrams** for workflows and processes
   - **Mermaid flowcharts** for decision trees and logic flows
   - **Mermaid architecture diagrams** for system overviews
   - **Mermaid state diagrams** for state machines and lifecycle flows

3. **Link to detailed docs** - Keep README as high-level overview

##### README Best Practices

**Structure:**
- **Quick-start first** - Get users running in < 5 minutes
- **Table of contents** - For READMEs > 100 lines
- **Sections in priority order:**
  1. Brief description (1-2 sentences)
  2. Quick start / Installation
  3. Common use cases
  4. Configuration (essential only)
  5. Links to detailed docs
  6. Troubleshooting (top 3-5 issues only)

**Content Guidelines:**
- ✅ **Prioritize quick-start information** and common use cases
- ✅ **Use tables, bullet points, inline code** for scannability
- ✅ **Include only essential examples** (one per concept)
- ✅ **Link to external docs** for deep dives
- ✅ **Use diagrams** to replace multi-paragraph explanations
- ❌ **Avoid verbose prose** - prefer concise bullet points
- ❌ **Avoid redundant examples** - one good example beats three similar ones
- ❌ **Avoid copy-pasting** - link to canonical docs instead

**Formatting for Scannability:**

**❌ BAD: Verbose prose**
```markdown
Before you begin, you will need to make sure that you have
Node.js version 18 or higher installed on your system. You
will also need Docker and kubectl.
```

**✅ GOOD: Concise with inline code**
```markdown
**Prerequisites:** Node.js 18+, Docker, `kubectl` installed
```

**❌ BAD: Paragraphs of text**
```markdown
In development mode, we use mock authentication for easier
testing, while in production we use OAuth 2.0. Similarly,
the database is SQLite in development but PostgreSQL in
production.
```

**✅ GOOD: Table for comparison**
```markdown
| Feature | Development | Production |
|---------|-------------|------------|
| Auth    | Mock        | OAuth 2.0  |
| DB      | SQLite      | PostgreSQL |
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chimon2000) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
