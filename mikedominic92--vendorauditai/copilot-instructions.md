## vendorauditai

> - ALWAYS check and verify today's date before starting any task

# Claude Code User Preferences

## Important Rules

### ALWAYS Check Today's Date and Time First
- ALWAYS check and verify today's date before starting any task
- Use current date for all documentation, comments, and references
- Ensure technology versions are current as of today's date

### Git Commit Rules

**NO Co-Author Attribution**
- NEVER add "Co-Authored-By: Claude" to commit messages
- NEVER add the Claude Code footer to commits
- Keep commit messages clean and professional
- User takes full ownership of all commits

**NO Emojis**
- NEVER use emojis in any output
- NEVER use emojis in commit messages
- NEVER use emojis in code comments
- NEVER use emojis in documentation
- Use plain text markers instead: [+], [!], [*], [INFO], [WARN], [ERROR]

**Commit After Every Change**
- ALWAYS commit immediately after completing any change
- Push to GitHub after each commit
- Do not batch multiple changes into one commit

### Commit Message Format

```
[TYPE]: Brief description

Detailed explanation if needed.
```

Types: SECURITY, FIX, FEAT, DOCS, REFACTOR, TEST, CHORE

### Example Commit

```
SECURITY: Upgrade Next.js 16.0.6 to 16.0.7

Patches CVE-2025-66478 (CVSS 10.0) - Remote Code Execution
vulnerability in React Server Components.
```

## Workflow Rules

### Planning
- Use agents in parallel when possible to speed up work
- Create multiple agents for independent tasks
- Plan complex tasks before implementation

### ALWAYS TEST AND VERIFY
- ALWAYS test and verify after ANY action or check
- Never assume something worked - confirm it
- Run tests, check endpoints, verify deployments
- Document verification results
- Test BEFORE and AFTER every change

### Production Grade Standards
- All code must be production-ready
- No shortcuts or placeholder implementations
- Enterprise-quality security and error handling
- Complete documentation for all features

## Technology Standards (As of January 2026)

### AI Models
- Claude: claude-opus-4-5-20251101 (Claude Opus 4.5)
- Gemini: gemini-3-pro (Gemini 3 Pro)
- Embeddings: text-embedding-3-large (OpenAI)

### Backend
- Python 3.12+
- FastAPI (latest)
- SQLAlchemy 2.0+
- PostgreSQL 16+

### Frontend
- React 18+
- TypeScript 5+
- TailwindCSS 3+
- TanStack Query v5

### Infrastructure
- Docker
- Railway (Backend)
- Netlify (Frontend)
- GitHub Actions (CI/CD)

---
> Source: [MikeDominic92/VendorAuditAI](https://github.com/MikeDominic92/VendorAuditAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
