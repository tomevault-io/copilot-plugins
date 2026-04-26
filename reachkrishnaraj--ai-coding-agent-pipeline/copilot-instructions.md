## ai-coding-agent-pipeline

> A cloud-hosted service that manages the full lifecycle of AI coding tasks: intake via Web UI or Slack, LLM analysis with OpenAI gpt-4o, clarification Q&A, GitHub Issue creation with label-based agent routing, and PR tracking. Deployed on Railway.

# AI Pipeline — Agent Onboarding

## What This Is
A cloud-hosted service that manages the full lifecycle of AI coding tasks: intake via Web UI or Slack, LLM analysis with OpenAI gpt-4o, clarification Q&A, GitHub Issue creation with label-based agent routing, and PR tracking. Deployed on Railway.

## Specification
Read `SPEC.md` for the complete V2.1 specification. Read `MASTER-AGENT-PLAN.md` for the build plan with per-session prompts.

## Tech Stack
- **Backend:** NestJS 11, TypeScript 5, Mongoose 8, MongoDB 7
- **Frontend:** React + Vite + Tailwind CSS (in `/web`)
- **Package Manager:** pnpm
- **Hosting:** Railway (container + MongoDB Atlas or Railway MongoDB plugin)
- **LLM:** OpenAI gpt-4o (chat completions API)
- **Auth:** GitHub OAuth (mothership org members only)

## Conventions

### Project Structure
```
src/
  tasks/        # Core task CRUD, state machine, event logging
  llm/          # OpenAI integration, prompt templates
  github/       # Issue creation, webhook handling
  slack/        # Slash commands, DM notifications
  auth/         # GitHub OAuth, session, guards
  common/       # Shared interfaces, config, filters, schemas
  database/     # MongoDB connection module
web/            # React frontend (Vite + Tailwind)
```

### Code Style
- Follow NestJS module structure: module → controller → service → dto
- Use Mongoose for ALL database access (never raw MongoDB driver calls)
- Use `class-validator` decorators for DTO validation
- Use `@nestjs/config` for environment variables — NEVER hardcode secrets
- All API routes under `/api/*`
- All webhook routes under `/api/webhooks/*`
- Use proper HTTP status codes (201 created, 400 bad input, 502 upstream failure)
- Export one thing per file when possible
- Use TypeScript strict mode

### Testing
- Unit tests with Jest (NestJS default)
- Frontend tests with Vitest
- Mock external services (OpenAI, GitHub, Slack) in tests
- Test state machine transitions (valid AND invalid)
- Test webhook signature verification

### Git Workflow
- Each feature gets its own branch: `feat/core-api`, `feat/integrations`, `feat/web-ui`, `feat/slack`, `feat/deploy`
- Commit messages: `type: description` (e.g., `feat: add task state machine`, `fix: handle LLM parse failure`)
- One logical change per commit

### Environment Variables
All secrets must come from environment variables. See `.env.example` for the full list.
Required: MONGODB_URI, OPENAI_API_KEY, GITHUB_TOKEN, GITHUB_WEBHOOK_SECRET, SESSION_SECRET
Optional: SLACK_BOT_TOKEN, SLACK_SIGNING_SECRET

### Security Rules
- Validate repo starts with `mothership/` before any GitHub API call
- Verify webhook signatures (GitHub: X-Hub-Signature-256, Slack: X-Slack-Signature)
- GitHub OAuth: reject users not in the `mothership` org
- Never expose API keys in logs, responses, or issue bodies

## Agent Team Structure
This project uses Claude Code sub-agents organized as a team:

| Agent | Domain | Branch | Dependencies |
|-------|--------|--------|--------------|
| `core-api` | DB + API + State Machine | `feat/core-api` | None |
| `integrations` | LLM + GitHub services | `feat/integrations` | None |
| `web-ui` | Auth + React frontend | `feat/web-ui` | None |
| `slack` | Slack integration | `feat/slack` | core-api merged |
| `deploy` | Dockerfile + Railway | `feat/deploy` | All features merged |
| `coordinator` | Orchestrates all agents | main | N/A |

## Merge Order
1. `feat/core-api` (has DB schema)
2. `feat/integrations` (replace mocks with real services)
3. `feat/web-ui` (adds auth + frontend)
4. `feat/slack` (after core-api is available)
5. `feat/deploy` (after all features merged)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reachkrishnaraj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
