## raimy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code Generation Principles

### Clarification Over Assumption
- **Always ask for clarification** when requirements are ambiguous or incomplete
- Don't assume implementation details, library choices, or architectural patterns
- When multiple valid approaches exist, present options and ask the user to choose
- If user intent is unclear, ask specific questions before writing code

### Minimal and Focused Changes
- Only make changes that are directly requested or clearly necessary
- Don't add features, refactoring, or "improvements" beyond what was asked
- A bug fix doesn't need surrounding code cleaned up
- A simple feature doesn't need extra configurability
- Don't add comments, docstrings, or type annotations to code you didn't change

### Avoid Over-Engineering
- Don't create abstractions for one-time operations
- Don't design for hypothetical future requirements
- Three similar lines of code is better than a premature abstraction
- Don't add error handling for scenarios that can't happen
- Trust internal code and framework guarantees
- Only validate at system boundaries (user input, external APIs)

### Read Before Writing
- **Never propose changes to code you haven't read**
- Always read existing files before modifying them
- Understand the existing patterns and conventions in the codebase
- Match the existing code style and structure

### Security First
- Be vigilant about security vulnerabilities (command injection, XSS, SQL injection, OWASP Top 10)
- If you write insecure code, immediately fix it
- Validate and sanitize user input at system boundaries

### Complete Deletions
- If something is unused, delete it completely
- Don't use backwards-compatibility hacks (renaming unused variables with `_` prefix, re-exporting types, `// removed` comments)
- Clean removals are better than commented-out code

## Tech Stack

### Frontend
- Next.js with React
- TypeScript
- Tailwind CSS

### Backend
- Python with FastAPI
- LangGraph and LangChain for AI agents
- PostgreSQL database
- SQLAlchemy with Alembic for migrations
- MCP (Model Context Protocol) for tool integration

### Infrastructure
- Docker and Docker Compose
- WebSocket for real-time communication with frontend
- Redis for pub/sub messaging between services
- Google OAuth for authentication

## Architecture Patterns

### Microservices Structure
The project uses a microservices architecture with:
- Multiple backend services communicating via HTTP
- Shared PostgreSQL database
- WebSocket for real-time communication with frontend
- Redis pub/sub for inter-service messaging
- Service-to-service authentication

### Agent System
The conversational AI uses:
- LangGraph for agent orchestration
- MCP (Model Context Protocol) for tool integration
- Session-based context management
- Structured outputs for UI integration

### Key Concepts
- **Sessions**: Conversations have types that determine agent behavior
- **Tools**: Agent capabilities exposed through MCP protocol
- **Service Auth**: Inter-service calls use API key authentication

## Development Workflow

### Setup and Installation
See [README.md](README.md) for environment setup and installation instructions.

### Common Commands
Check `package.json` scripts for frontend commands and `docker-compose.yml` for backend services.

### Testing
Run tests before committing. Test configuration in `pytest.ini`.

### Database Changes
Use Alembic for migrations. Migrations run automatically on backend startup in development.

### Git Commits
- **DO NOT** include Claude Code signatures, co-author tags, or "Generated with Claude Code" footers
- Add commit task only when I asked you to. We always need to test changes first
- Write concise commit messages with a summary line followed by 3-5 bullet points
- Focus on high-level "what" and "why" rather than listing every file or implementation detail
- Avoid verbose explanations, architectural details, or step-by-step descriptions
- **Describe the final result, not the implementation process**
  - ❌ Bad: "Create button component, make it clickable, add hover effects"
  - ✅ Good: "Add navigation button with hover effects"
  - Commit message should reflect what changed from the previous version, not the iterative steps taken during implementation

## Important Guidelines

### When Working with Python Backend
- **Always use proper logging, never use `print()` statements**
- Use Python's `logging` module with appropriate log levels:
  - `logger.debug()` - Detailed diagnostic information
  - `logger.info()` - General informational messages
  - `logger.warning()` - Warning messages for potentially problematic situations
  - `logger.error()` - Error messages for serious problems
- Initialize logger at module level: `logger = logging.getLogger(__name__)`
- Include contextual information in log messages (session_id, etc.)
- Use structured logging with clear prefixes (e.g., `logger.info(f"⏱️  TTFT: {ttft:.0f}ms")`)

### When Working with Agent/LLM Code
- Read existing prompts before modifying agent behavior
- Understand tool workflows before adding new MCP tools
- Session context is critical - don't break message history

### When Working with WebSocket Code
- WebSockets handle real-time communication between frontend and backend
- Authentication happens at connection time
- Session IDs route messages to correct connections
- Structured message format must be preserved for frontend

### When Working with Inter-service Messaging
- Redis pub/sub is used for real-time communication between backend services
- Session IDs are used to route messages to correct channels

### When Working with Services
- Services communicate via HTTP - check environment variables for URLs
- Database operations should use the centralized service layer
- Service authentication required for cross-service calls

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vzotov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vzotov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
