## sample-agentic-ai-strands

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Installation

```bash
# Install Python dependencies
uv sync

# Set up environment variables
cp env.example .env
# Edit .env file with appropriate values
```

### Create Required Infrastructure

```bash
# Create DynamoDB table for user configuration
aws dynamodb create-table \
    --table-name mcp_user_config_table \
    --attribute-definitions AttributeName=userId,AttributeType=S \
    --key-schema AttributeName=userId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

### Running the Application

```bash
# Start the backend service
bash start_all.sh

# Stop all services
bash stop_all.sh
```

### Frontend Setup

#### Running in Development Mode

```bash
# Navigate to the React UI directory
cd react_ui

# Create environment variables for frontend
cp .env.example .env.local

# Install dependencies
npm install

# Start development server
npm run dev

# For HTTPS development mode
npm run dev:https
```

#### Docker Setup (Frontend)

```bash
# Navigate to the React UI directory
cd react_ui

# Create environment variables for frontend
cp .env.example .env.local

# Build and start frontend containers
docker-compose up -d --build
```

### Docker Commands

```bash
# View container logs
docker logs -f mcp-bedrock-ui

# Restart containers
docker-compose restart

# Stop containers
docker-compose down

# Rebuild and start (after code updates)
docker-compose up -d --build
```

## Architecture Overview

This repository contains an Agentic AI application built using Strands Agents SDK that provides integration between large language models and external tool systems through the Model Context Protocol (MCP). The architecture follows a decoupled frontend and backend design, with a React UI for the frontend and a Python FastAPI server for the backend.

### Key Components of the Application

1. **Backend Service (`src/main.py`)**
   - FastAPI server that manages sessions, model interactions, and MCP server connections
   - Handles streaming responses from LLMs
   - Manages user sessions and MCP server configurations

2. **Strands Agent Client (`src/strands_agent_client.py`, `src/strands_agent_client_stream.py`)**
   - Provides integration with various LLM providers (Bedrock, OpenAI)
   - Handles streaming responses and agent interactions
   - Manages tool invocations and results

3. **MCP Client (`src/mcp_client_strands.py`)**
   - Implements the Model Context Protocol for tool integration
   - Manages connections to MCP servers
   - Handles tool registration and invocation

4. **Frontend (React UI)**
   - Next.js 15 based web interface for interacting with the agent
   - Displays streaming responses and tool results with markdown rendering
   - Manages MCP server configurations
   - Features a resizable tool usage panel for viewing tool calls and results
   - Supports dark/light mode themes with Tailwind CSS and Shadcn UI

### React UI Components

1. **ChatInterface (`components/chat/ChatInterface.tsx`)**
   - Main chat interface with messaging and tool panel
   - Handles resizable sidebar for tool usage display
   - Manages user sessions and message history

2. **Server Management (`components/sidebar/`)**
   - Model selector for choosing LLM models
   - Server list for managing MCP servers
   - Add server dialog for configuring new MCP servers

3. **Tool Integration (`components/chat/ToolUsagePanel.tsx`)**
   - Displays tool calls and their results
   - Provides detailed view of tool usage in modal dialogs
   - Handles displaying images from tool results

### Data Flow

1. User queries are sent to the FastAPI backend
2. Backend creates/retrieves user sessions and forwards requests to the Strands agent
3. Agent processes queries and may invoke tools via MCP servers
4. Results are streamed back to the user through SSE (Server-Sent Events)

### Deployment Options

1. **Development Mode**: Local deployment using `start_all.sh`
2. **Production Mode**: AWS ECS Fargate deployment using CDK

## Security

**IMPORTANT**: This application has undergone security hardening to fix critical vulnerabilities.

### Key Security Features

1. **Server-Side Authentication**: API keys are never exposed to clients
2. **Input Validation**: Strict validation prevents command injection attacks
3. **CORS Protection**: Configurable allowed origins prevent cross-site attacks

### Quick Security Setup

```bash
# 1. Set a secure API key
echo "API_KEY=$(openssl rand -hex 32)" >> .env

# 2. Configure allowed origins for production
echo "ALLOWED_ORIGINS=https://your-domain.com" >> .env

# 3. Review security documentation
cat SECURITY.md
```

### Security Documentation

- **SECURITY.md** - Complete security guide and best practices
- **SECURITY_MIGRATION.md** - Migration guide for existing deployments
- **src/security.py** - Input validation implementation

### Security Checklist

Before deploying to production:
- [ ] Change default API_KEY
- [ ] Configure ALLOWED_ORIGINS with your frontend domain
- [ ] Use HTTPS (set USE_HTTPS=1)
- [ ] Store API keys in AWS Secrets Manager (recommended)
- [ ] Review and test security controls

## Configuration

### Backend Configuration

The backend application can be configured through the `.env` file:

- Model provider selection (Bedrock, OpenAI)
- AWS credentials and regions
- Server host and port settings
- Observation capabilities (Langfuse)
- Memory features (mem0)

See `env.example` for all available configuration options.

### Frontend Configuration

The React UI can be configured through the `.env.local` file in the `react_ui` directory:

- API endpoint configuration
- Server MCP base URL
- API Key for authentication

The UI components are built with:
- Next.js 15 and React 18
- Tailwind CSS for styling
- Shadcn UI component library
- Radix UI primitives

---
> Source: [aws-samples/sample_agentic_ai_strands](https://github.com/aws-samples/sample_agentic_ai_strands) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
