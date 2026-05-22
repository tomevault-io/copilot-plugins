## raycast-openai-server

> This project is a Raycast extension that provides an OpenAI-compatible API server. It allows you to use Raycast's AI capabilities through a local server that mimics the OpenAI API. This is useful for tools that are built to work with the OpenAI API, as you can point them to your local server instead.

# GEMINI.md

## Project Overview

This project is a Raycast extension that provides an OpenAI-compatible API server. It allows you to use Raycast's AI capabilities through a local server that mimics the OpenAI API. This is useful for tools that are built to work with the OpenAI API, as you can point them to your local server instead.

The extension is built using TypeScript and the `@raycast/api`. It provides two commands:
- `start-openai-server`: Starts the OpenAI-compatible server on a configurable port.
- `kill-openai-server`: Stops the server.

The server supports both streaming and non-streaming responses, and it can list the available AI models.

## Building and Running

### Prerequisites
- Node.js and npm

### Installation

```bash
npm install
```

### Development

To run the extension in development mode:

```bash
npm run dev
```

### Building

To build the extension for production:

```bash
npm run build
```

### Publishing

To publish the extension to the Raycast Store:

```bash
npm run publish
```

## Usage

### Starting the server

Run the `start-openai-server` command in Raycast. The server will start on the port specified in the extension's preferences (default: `1235`).

### Stopping the server

Run the `kill-openai-server` command in Raycast.

### Tool Calling Modes

The server supports two modes of tool calling:

#### 1. OpenAI Native Tools
When you provide a `tools` parameter in your request, the server uses OpenAI's native tool calling format. The AI will respond with structured tool calls that you can execute and return results for.

**Example request with tools:**
```bash
curl -X POST http://localhost:1235/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "google-gemini-2.0-flash",
    "tools": [
      {
        "type": "function",
        "function": {
          "name": "search",
          "description": "Search for information",
          "parameters": {
            "type": "object",
            "properties": {
              "query": {"type": "string"}
            }
          }
        }
      }
    ],
    "messages": [
      {"role": "user", "content": "Search for weather in Paris"}
    ]
  }'
```

#### 2. MCP-Style Tool Calling
When no `tools` parameter is provided, the server assumes MCP-style (Model Context Protocol) tool calling. In this mode, the server:
- Passes system messages through unchanged (allowing clients to define tools in natural language)
- Transforms tool results from MCP format into readable text for the AI
- Maintains full conversation history including all tool interactions

This mode works with clients like Continue.dev that implement tool calling through descriptive system prompts rather than OpenAI's structured tools format.

### Making requests to the server

You can make requests to the server using `curl` or any other HTTP client.

#### List models

```bash
curl http://localhost:1235/v1/models | jq .
```

#### Chat completions (streaming)

```bash
curl -N -X POST http://localhost:1235/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "google-gemini-2.0-flash", "stream": true, "messages": [{"role": "user", "content": "Who is the president?"}]}'
```

#### Chat completions (non-streaming)

```bash
curl -X POST http://localhost:1235/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "google-gemini-2.0-flash", "stream": false, "messages": [{"role": "user", "content": "Who is the president?"}]}'
```

## Development Conventions

### Linting

The project uses ESLint for linting. To check for linting errors:

```bash
npm run lint
```

To fix linting errors automatically:

```bash
npm run fix-lint
```

### Code Formatting

The project uses Prettier for code formatting. It is recommended to set up your editor to format on save.

### Commits and Pull Requests

TODO: Add information about commit message conventions and the pull request process.

---
> Source: [kirel/raycast-openai-server](https://github.com/kirel/raycast-openai-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
