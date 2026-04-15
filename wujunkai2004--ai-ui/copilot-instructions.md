## ai-ui

> We are building an AI Agent backend that bridges the gap between vague user intents and precise execution.

# Project Specification: GenUI-Agent-Backend

## 1. Project Overview
We are building an AI Agent backend that bridges the gap between vague user intents and precise execution.
**Core Philosophy:** Users cannot always prompt perfectly. The system interprets vague requests, generates a dynamic, structured UI (Generative UI) to collect missing data, and then executes tasks using the Model Context Protocol (MCP).
The project is done in Python using FastAPI and OpenAI's LLMs, with MCP for tool integration.

## 2. Tech Stack & Environment
- **Language:** Python 3.10+
- **Package Manager:** `uv`
- **Web Framework:** FastAPI
- **Database:** SQLite (via `peewee` ORM)
- **LLM Provider:** OpenAI API (gpt-4o or compatible / DeepSeek)
- **Core Protocol:** MCP (Model Context Protocol) - Python SDK
- **Architecture pattern:** Clean Architecture / Service-Repository pattern.

## 3. Core Workflow (The "Three-Step Loop")

### Step 1: Clarification (Intent Analysis)
- Input: User's query + Chat History + Authentication.
- Process: LLM analyzes the query to identify missing information or answer directly.
- Output: A structured `UIResponse` containing UI components OR a direct text answer (`message` field).

### Step 2: Information Collection
- The frontend renders the components defined in Step 1.
- User interacts and submits structured data.

### Step 3: Execution (Orchestration)
- Input: The structured data from Step 2 combined with original intent and Chat History.
- Process: The Backend acts as an **MCP Host**. It connects to available MCP Servers.
- Action: The LLM plans the execution path and calls MCP tools.
- Output: Final result to the user (automatically saved to history).

## 4. System Components

### 4.1 Database Models
- `User`: Handles authentication (Token-based) and user profile.
- `Chat`: Represents a conversation session.
- `Message`: Stores user queries, assistant responses (UI JSON or Text), and execution results.

### 4.2 The UI Protocol
Defined in `app/models/ui_protocol.py`.
Supported Component Types: `Input`, `Select`, `DatePicker`, `MultiSelect`, `Button`, `MapPin`, `RangeSlider`, `VisualPicker`, `Stepper`, `Switch`.
Response Format: `UIResponse(components=[...], message="...")`

### 4.3 API Endpoints
Base URL: `/api/v1`
**Authentication:** Required for all endpoints (except `/login`). Header: `Authorization: <token>`

*   `POST /login`: Authenticate or register (auto-create account).
*   `POST /chat`: Create a new chat session.
*   `GET /chatList`: Retrieve user's chat sessions.
*   `GET /history`: Retrieve message history for a chat (supports paging).
*   `POST /analyze`: Analyze intent. Requires `chat_id` in body.
*   `POST /execute`: Execute task. Requires `chat_id` in body.

## 5. MCP Integration
- The application acts as an **MCP Client/Host**.
- **Configuration:** `mcp_config.json` defines MCP servers (command + args).
- **Service:** `MCPClientService` manages connections via Stdio.

## 6. Critical Operational Rules
1.  **INSPECT BEFORE EDIT:** Before modifying ANY code or configuration file, you MUST first read the file to confirm its current state. Do not assume the content based on memory or previous turns. This prevents overwriting user-made changes.
2.  **Minimal Changes:** When updating code, apply the principle of least disturbance. Use `replace` or targeted edits rather than full file overwrites whenever possible.
3.  **Security:** Never commit secrets (`config.toml`, `.env`). Ensure `.gitignore` is active.

## 7. Development Guidelines
- Use `pydantic` for all data validation.
- Use `fastapi.APIRouter` to structure endpoints.
- Ensure the OpenAI API key is loaded from environment variables (`config.toml`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WuJunkai2004)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WuJunkai2004)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
