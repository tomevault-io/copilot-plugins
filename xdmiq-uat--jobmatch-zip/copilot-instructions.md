## jobmatch-zip

> - AI-Enabled LLC Matching Platform for job matching

# Cursor Rules for AI-Enabled LLC Matching Platform

## Project Context
- AI-Enabled LLC Matching Platform for job matching
- Anonymous identity system (zero-knowledge)
- XDMIQ credentialing integration
- Human-in-the-loop architecture
- Built with FastAPI (backend) and Next.js 14+ (frontend)

## Development Tools
- **Warp**: Terminal operations
- **Claude Code**: AI-assisted coding with hooks
- **Cursor**: Interactive IDE for business logic

## Key Principles
1. Human-in-the-loop always more beneficial
2. Anonymous identity preserved throughout
3. Capability over credentials
4. State recovery and testing support

## Code Style
- Backend: FastAPI with Pydantic models, SQLAlchemy ORM
- Frontend: Next.js 14+ with TypeScript, React Server Components
- Always include proper error handling
- Maintain anonymous identity in all operations
- Create state checkpoints for AI decisions

## When Creating Code
- Reference existing patterns in the codebase
- Use hooks from `.claude-code/hooks.json` for context
- Maintain consistency with existing code
- Focus on business requirements
- Consider scalability (1B+ users target)

## AI Integration Guidelines
- Always include human review queue
- Create checkpoints for state recovery
- Use OpenAI API with fallbacks
- Log AI decisions for human review
- Support rollback capabilities

## Authentication
- Support: Facebook, LinkedIn, Google, Microsoft, Apple, Email, SMS/VoIP
- All providers link to anonymous identity
- Never reveal real identity

## XDMIQ Integration
- Use preference-based questions ("Which do you prefer?" + "Why?")
- Calculate XDMIQ scores (0-100)
- Integrate scores into matching algorithm

## Cursor Agents System
- **Agent Registry**: `.cursor/agents/agents-registry.json` contains all available agents
- **Agent Configurations**: Each agent has a JSON file in `.cursor/agents/` directory
- **Agent Usage**: Reference agents using `@agent-name [command]` syntax
- **Available Agents**: 15 specialized agents for C-Suite, Development, Project Management, Operations, and Specialized tasks
- **Agent Integration**: Agents coordinate with each other automatically based on their integration definitions
- **Agent Context**: Each agent knows its workspace paths, relevant files, and script locations from `E:\agents\` directory

### Agent Categories Available:
- **C-Suite**: fractional-ceo-agent, fractional-cfo-agent, fractional-coo-agent
- **Development**: principal-dev-agent, dev-sync-agent, review-agent
- **Project Management**: git-project-manager-agent, project-manager-agent, program-manager-agent, product-agent
- **Operations**: change-management-agent, infrastructure-agent
- **Specialized**: historian-agent, data-governance-agent, link-validation-agent

### When to Use Agents:
- For strategic decisions: @fractional-ceo-agent
- For financial analysis: @fractional-cfo-agent
- For operational questions: @fractional-coo-agent
- For code review/architecture: @principal-dev-agent
- For Git/GitHub management: @git-project-manager-agent
- For sprint/project management: @project-manager-agent
- For program coordination: @program-manager-agent
- For product features: @product-agent
- For change management: @change-management-agent
- For documentation: @historian-agent
- For compliance: @data-governance-agent
- For link validation: @link-validation-agent

### Agent Verification:
At the start of each chat, Cursor should acknowledge available agents. You can verify by asking:
- "What agents are available?"
- "Show me the agent registry"
- "List all agents"

All agent configurations reference scripts and tools from `E:\agents\` directory structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XDMIQ-UAT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
