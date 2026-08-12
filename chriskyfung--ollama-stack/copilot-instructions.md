## ollama-stack

> This project provides a Docker-based setup to run an [Ollama](https://ollama.ai/) instance and expose it to the internet securely using a [Cloudflare Tunnel](https://www.cloudflare.com/products/tunnel/).

# Ollama Stack Project

## Project Overview

This project provides a Docker-based setup to run an [Ollama](https://ollama.ai/) instance and expose it to the internet securely using a [Cloudflare Tunnel](https://www.cloudflare.com/products/tunnel/).

The stack is defined in `docker-compose.yaml` and consists of two main services:

*   **`ollama`**: The core service running the Ollama large language model server.
*   **`cloudflared`**: The service that creates a secure tunnel to the `ollama` service, allowing it to be accessed from a public URL.

A `docker-compose.override.yaml` is also present, which suggests a mechanism for local development overrides, such as allowing specific origins (`moz-extension://*`) and connecting to an external Docker network.

## Building and Running

### Prerequisites

1.  **Docker and Docker Compose:** You must have Docker and Docker Compose installed.
2.  **Cloudflare Tunnel Token:** You need a Cloudflare Tunnel token. Create a file at `.secrets/tunnel_token.txt` and place your token in it. This file is git-ignored for security.

### Commands

*   **To start the services:**
    ```bash
    docker-compose up -d
    ```

*   **To stop the services:**
    ```bash
    docker-compose down
    ```

*   **To view logs:**
    ```bash
    docker-compose logs -f
    ```

## Development Conventions

*   **Configuration:** The primary configuration is in `docker-compose.yaml`.
*   **Secrets:** Secrets, such as the Cloudflare Tunnel token, are managed in the `.secrets/` directory, which is excluded from Git via `.gitignore`.
*   **Local Overrides:** The `docker-compose.override.yaml` file is used to provide local modifications to the base configuration without altering the main `docker-compose.yaml`. This is useful for development or specific deployment environments. For example, it is used here to set `OLLAMA_ORIGINS` and connect to a different network.
*   **Environment Variables:** The `.gitignore` file also indicates the potential use of a `.env` file for environment-specific variables, which is a common practice with Docker Compose.

---
> Source: [chriskyfung/ollama-stack](https://github.com/chriskyfung/ollama-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
