## my-agent

> This project is a Go-based AI agent built using the **Google ADK (Agent Development Kit)** and integrated with **Gemini**. It serves as a prototype for a multi-tool agent capable of performing Google Searches to answer user queries.

# Gemini Instruction Context

This project is a Go-based AI agent built using the **Google ADK (Agent Development Kit)** and integrated with **Gemini**. It serves as a prototype for a multi-tool agent capable of performing Google Searches to answer user queries.

## Project Overview

- **Main Technologies:** Go (v1.25+), [Google ADK](https://google.golang.org/adk), Google GenAI SDK.
- **Model:** `gemini-2.5-flash`.
- **Architecture:** A single-entrypoint Go application (`agent.go`) that defines the agent's behavior, instructions, and tools.
- **Tools:** `geminitool.GoogleSearch{}` is currently integrated.

## Building and Running

### Prerequisites
- Go 1.25.4 or higher.
- A valid Google API Key for Gemini.

### Setup
1. Create a `.env` file in the project root:
   ```env
   GOOGLE_API_KEY=your_api_key_here
   GOOGLE_GENAI_USE_VERTEXAI=FALSE
   ```
2. Install dependencies and vendor them (optional but recommended):
   ```bash
   go mod tidy
   go mod vendor
   ```

### Running the Agent
Start the agent with the ADK launcher (provides Web UI and API):
```bash
go run ./cmd/agent web webui api
```

## Development Conventions

- **Environment Variables:** Secrets are managed via a `.env` file, which is automatically loaded at startup by `agent.go`.
- **Tooling:** Always run `go mod tidy` after adding new dependencies.
- **Project Structure:**
  - `agent.go`: Contains the main logic, agent definition, and model configuration.
  - `vendor/`: Contains third-party dependencies.
  - `go.mod` / `go.sum`: Dependency management (Note: `go.mod` is currently ignored by `.gitignore` in this environment).
- **Authentication:** The project uses the `GOOGLE_API_KEY` environment variable for authenticating with the Gemini API.

## Key Files
- `agent.go`: The core logic for the agent.
- `README.md`: High-level project documentation and setup instructions.
- `.gitignore`: Configured to ignore secrets (`.env`) and build artifacts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/herdiagusthio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/herdiagusthio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
