## mcp-vercel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Model Context Protocol (MCP) integration for Vercel's REST API. It implements tools for interacting with Vercel's APIs, enabling LLMs and other applications to programmatically manage Vercel deployments, projects, teams and environment variables.

## Commands

### Development

```bash
# Install dependencies
npm install

# Run the MCP server in development mode
npm run dev

# Build the project
npm run build

# Start the production server
npm start
```

### Docker Commands

```bash
# Build the Docker image
docker build -t vercel-mcp .

# Run in development mode with live reload
docker build --target builder -t vercel-mcp-dev .
docker run -it --rm \
  -e VERCEL_API_TOKEN=your_token_here \
  -p 3399:3399 \
  -v $(pwd)/src:/app/src \
  vercel-mcp-dev

# Run in production mode
docker run -it --rm \
  -e VERCEL_API_TOKEN=your_token_here \
  -p 3399:3399 \
  vercel-mcp
```

## Environment Variables

- `VERCEL_API_TOKEN`: Required Vercel API token for authentication with Vercel API.

## Architecture

This project is built as a Model Context Protocol (MCP) server that implements a set of tools for interacting with the Vercel API. Key components include:

1. **Server Initialization (`src/index.ts`)**:

   - Entry point that creates an MCP server instance
   - Registers tool handlers and configures error handling

2. **Tools Definition (`src/constants/tools.ts`)**:

   - Defines the schemas and descriptions for all available tools
   - Each tool has a defined name, description, and input schema

3. **Tool Handlers (src/tools/<domain>/handlers.ts)**:

   - Implements the actual tool functionality
   - Four main domains: deployments, environments, projects, and teams
   - Each handler validates inputs, makes API calls, and formats responses

4. **API Utilities (`src/utils/api.ts` & `src/utils/config.ts`)**:

   - Handles authentication and API requests to Vercel
   - Configures API endpoints and tokens

5. **Type Definitions (src/tools/<domain>/types.ts)**:
   - TypeScript interfaces for Vercel API responses and parameters
   - Ensures type safety throughout the application

The application follows a modular architecture where:

- Each Vercel API domain has its own directory with handlers, schemas, and types
- The server routes tool calls to the appropriate handler based on the tool name
- All handlers use a common API utility for making authenticated requests to Vercel

## Development Workflow

1. Define a new tool in `src/constants/tools.ts`
2. Create handler, schema, and types files in the appropriate domain directory
3. Register the tool handler in `src/index.ts`
4. Test the tool functionality with an MCP client

When adding a new tool, follow the pattern established in existing tools:

1. Create a schema definition using Zod
2. Implement a handler function that validates inputs and calls the Vercel API
3. Define types for request parameters and response data
4. Register the tool in the `VERCEL_TOOLS` array and add the handler to the switch statement in index.ts

---
> Source: [nganiet/mcp-vercel](https://github.com/nganiet/mcp-vercel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
