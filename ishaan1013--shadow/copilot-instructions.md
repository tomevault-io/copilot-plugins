## shadow

> This guide provides comprehensive information for Claude Code development on the Shadow platform - a remote, autonomous coding agent with hardware-isolated execution environments.

# Shadow Development Guide for Claude Code

This guide provides comprehensive information for Claude Code development on the Shadow platform - a remote, autonomous coding agent with hardware-isolated execution environments.

## Platform Overview

Shadow is an AI coding platform that enables autonomous agents to work on GitHub repositories through real-time collaboration, semantic code search, and long-horizon task management. The platform provides both local and remote execution modes with enterprise-grade security.

### Core Architecture

**Monorepo Structure** (Turborepo managed):
- `apps/frontend/` - Next.js 15 + React 19 application with real-time chat interface
- `apps/server/` - Node.js orchestrator for LLM integration and WebSocket communication  
- `apps/sidecar/` - Express.js service for isolated file operations and command execution
- `packages/db/` - Prisma schema and PostgreSQL client
- `packages/types/` - Shared TypeScript type definitions
- `packages/command-security/` - Security utilities for command validation

**Dual Execution Modes**:
- **Local Mode**: Direct filesystem execution on host machine for development
- **Remote Mode**: Hardware-isolated execution in Kata QEMU containers for production

## Key Technologies

- **Frontend**: Next.js 15, React 19, Tailwind CSS, Radix UI, Socket.IO client
- **Backend**: Node.js, Express, Prisma ORM, PostgreSQL, Socket.IO server
- **AI Integration**: AI SDK with support for Anthropic, OpenAI, OpenRouter
- **Authentication**: BetterAuth for session management
- **Infrastructure**: AWS EKS, Kata Containers, Docker

## Core Development Workflows

### Chat Message Streaming
The platform uses Socket.IO for real-time streaming with structured message parts:

```typescript
// Message part types in streaming
- TextPart: Regular text content
- ReasoningPart: AI reasoning with signatures
- ToolCallPart: Tool execution requests
- ToolResultPart: Tool execution results
- ErrorPart: Error information
```

### Task Lifecycle
1. **Initialization**: Workspace setup, VM creation (remote mode), dependency installation
2. **Active**: Chat streaming, tool execution, file operations
3. **Completion**: Git commits, PR generation, cleanup scheduling

### Tool System Architecture
Built-in tools available to AI agents:

**File Operations**:
- `read_file` - Read file contents with line range support
- `edit_file` - Write and modify files with validation
- `search_replace` - Precise string replacement
- `delete_file` - Safe file deletion
- `list_dir` - Directory exploration

**Code Search**:
- `grep_search` - Pattern matching with regex
- `file_search` - Fuzzy filename search  
- `semantic_search` - AI-powered semantic code search (when indexing complete)

**Terminal & Execution**:
- `run_terminal_cmd` - Command execution with real-time output streaming
- Command validation and security checks via `packages/command-security/`

**Task Management**:
- `todo_write` - Structured task management with WebSocket updates
- `add_memory` - Repository-specific knowledge storage
- `list_memories` - Retrieve stored knowledge by category
- `remove_memory` - Delete specific memories

**MCP Integration**:
- External tools via Model Context Protocol
- Dynamic tool discovery and execution
- Tool name transformation for AI SDK compatibility

## Database Schema Understanding

### Core Models
- **User**: Authentication, settings, GitHub app connections
- **Task**: Central entity for AI agent work sessions
- **ChatMessage**: Streaming messages with metadata and tool usage
- **Memory**: Repository-specific knowledge with categorization
- **CodebaseUnderstanding**: Shadow Wiki content storage
- **Todo**: Structured task tracking with real-time updates

### Key Relationships
```typescript
User -> Task (one-to-many)
Task -> ChatMessage (one-to-many) 
Task -> Memory (one-to-many)
Task -> Todo (one-to-many)
Task -> CodebaseUnderstanding (many-to-one)
```

## Security & Safety

### Command Security
All terminal commands go through validation in `packages/command-security/`:
- Command parsing and analysis
- Security level assessment
- Path traversal protection
- Workspace boundary enforcement

### Execution Isolation
- **Local Mode**: Process isolation with workspace boundaries
- **Remote Mode**: Hardware-level VM isolation via Kata QEMU containers
- Container runtime class: `kata-qemu` for true isolation

### API Key Management
- Secure storage in browser cookies
- Per-provider validation (Anthropic, OpenAI, OpenRouter)
- Model context service for API key management

## Development Guidelines

### TypeScript Patterns
- Strict type checking throughout
- Shared type definitions in `packages/types/`
- Discriminated unions for message parts and tool results
- Zod schemas for runtime validation

### Testing Approach
```bash
# Frontend testing
npm run check-types --filter=frontend
npm run lint --filter=frontend

# Server testing  
npm run check-types --filter=server
npm run lint --filter=server

# Full project
npm run check-types
npm run lint
```

### WebSocket Events
Maintain compatibility across frontend/backend changes:
- `stream-chunk` - Real-time content streaming
- `task-status-updated` - Task state changes
- `terminal-output` - Command execution output
- `todo-update` - Task management updates

### Code Organization
- Components in feature-based directories
- Hooks for state management and API calls
- Shared utilities in `lib/` directories
- API routes following REST conventions

## Common Development Tasks

### Adding New Tools
1. Create tool schema in `packages/types/src/tools/schemas.ts`
2. Add tool implementation in `apps/server/src/agent/tools/index.ts`
3. Create prompt files in `apps/server/src/agent/tools/prompts/[tool_name]/`
4. Add frontend components in `apps/frontend/components/chat/tools/`

### Database Schema Changes
```bash
# Development
npm run db:push

# Production migrations
npm run db:migrate:dev
npm run db:migrate:deploy
```

### Environment Modes
- **Development**: `NODE_ENV=development`, `AGENT_MODE=local`
- **Production**: `NODE_ENV=production`, `AGENT_MODE=remote`

## Troubleshooting

### Build Errors
```bash
# Module resolution issues
npm run build

# Type checking issues
npm run check-types
```

### Database Issues
```bash
# Reset development database
npm run db:push:reset

# Regenerate Prisma client
npm run generate
```

### WebSocket Connection Issues
- Check CORS configuration in server config
- Verify Socket.IO server/client version compatibility
- Ensure proper cookie handling for API keys

### Remote Mode Issues
- Verify AWS credentials and EKS cluster access
- Check Kata Containers installation on Amazon Linux 2023 nodes
- Ensure proper RBAC configuration for Kubernetes

## Advanced Topics

### Codebase Intelligence
- Tree-sitter parsing for multiple programming languages
- Graph-based code representation with symbol tracking
- Shadow Wiki generation for comprehensive documentation
- Pinecone vector database integration for semantic search

### Infrastructure Management
```bash
# Deploy EKS cluster with Kata Containers
./scripts/deploy-remote-infrastructure.sh

# Deploy ECS backend service  
./scripts/deploy-backend-ecs.sh

# Infrastructure cleanup
./scripts/cleanup-infrastructure.sh
```

### MCP Server Development
- Implement MCP protocol for external tool integration
- Tool name transformation for AI SDK compatibility
- Context7 integration with token limiting

This guide covers the essential knowledge needed for effective Claude Code development on Shadow. For specific implementation details, refer to the source code and inline documentation.

---
> Source: [ishaan1013/shadow](https://github.com/ishaan1013/shadow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
