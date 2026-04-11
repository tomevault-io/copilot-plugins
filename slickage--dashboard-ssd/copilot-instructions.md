## dashboard-ssd

> This project is a modern dashboard for managing software development projects, clients, deployments, and integrations. It is built with Elixir and Phoenix, using LiveView for the frontend and Tailwind CSS for styling. The dashboard provides real-time updates and integrates with various tools like Linear, Slack, Notion, and Google Drive. User authentication is handled through Google OAuth, with role-based access control.

# Gemini Project: dashboard-ssd

## Project Overview

This project is a modern dashboard for managing software development projects, clients, deployments, and integrations. It is built with Elixir and Phoenix, using LiveView for the frontend and Tailwind CSS for styling. The dashboard provides real-time updates and integrates with various tools like Linear, Slack, Notion, and Google Drive. User authentication is handled through Google OAuth, with role-based access control.

## Building and Running

### Prerequisites

*   Elixir 1.18+
*   Phoenix 1.8+
*   PostgreSQL
*   Node.js 20+

### Quick Start

1.  **Clone and setup**:
    ```bash
    git clone https://github.com/slickage/dashboard-ssd.git
    cd dashboard-ssd
    mix setup
    ```

2.  **Configure environment**:
    ```bash
    cp .env.example .env
    # Edit .env with your API keys and secrets
    ```

3.  **Run the application**:
    ```bash
    mix phx.server
    ```

4.  **Visit**: [`localhost:4000`](http://localhost:4000)

### Development Commands

*   **Run tests**: `mix test`
*   **Check code quality**: `mix check` (includes formatting, linting, dialyzer, tests, and docs)
*   **Live reload**: Assets are compiled automatically with esbuild.

## Development Conventions

*   **Code Style**: The project follows the standard Elixir and Phoenix conventions. Use `mix format` to format the code and `mix credo --strict` to check for style issues.
*   **Testing**: All new features should be accompanied by tests. The `mix check` command includes a test run.
*   **Contributions**: Before pushing changes, ensure that `mix check` passes. Update documentation as needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slickage)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/slickage)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
