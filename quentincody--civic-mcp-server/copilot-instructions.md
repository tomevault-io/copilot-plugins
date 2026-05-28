## civic-mcp-server

> This project is a Cloudflare Workers-based Model Context Protocol (MCP) server that provides tools for querying the CIViC (Clinical Interpretation of Variants in Cancer) API. The server converts GraphQL responses into queryable SQLite tables using Durable Objects for efficient data processing. The CIViC database is a crowd-sourced repository of clinical interpretations of cancer variants. This MCP server enables structured queries and data analysis of cancer genomics information through natural language interactions with AI assistants.

# Project: CIViC MCP Server

## Project Overview

This project is a Cloudflare Workers-based Model Context Protocol (MCP) server that provides tools for querying the CIViC (Clinical Interpretation of Variants in Cancer) API. The server converts GraphQL responses into queryable SQLite tables using Durable Objects for efficient data processing. The CIViC database is a crowd-sourced repository of clinical interpretations of cancer variants. This MCP server enables structured queries and data analysis of cancer genomics information through natural language interactions with AI assistants.

The server is built with TypeScript and utilizes the Cloudflare Workers platform for deployment. It uses the `@modelcontextprotocol/sdk` to implement the MCP specification.

## Building and Running

The following commands are available to build, run, and test the project:

*   **Install Dependencies:**
    ```bash
    npm install
    ```

*   **Run in Development:**
    ```bash
    npm run dev
    ```
    This command starts a local development server using `wrangler dev`.

*   **Deploy to Cloudflare:**
    ```bash
    npm run deploy
    ```
    This command deploys the application to Cloudflare Workers using `wrangler deploy`.

*   **Lint and Format:**
    ```bash
    npm run format
    npm run lint:fix
    ```
    These commands use Biome to format and lint the codebase.

*   **Generate Cloudflare Types:**
    ```bash
    npm run cf-typegen
    ```
    This command generates types for Cloudflare Workers.

## Development Conventions

*   **Language:** The project is written in TypeScript.
*   **Formatting and Linting:** The project uses Biome for code formatting and linting. Configuration can be found in `biome.json`.
*   **Deployment:** The project is deployed to Cloudflare Workers using the Wrangler CLI. Configuration can be found in `wrangler.jsonc`.
*   **Dependencies:** Project dependencies are managed with npm.
*   **MCP Implementation:** The server implements the MCP 2025-06-18 specification.
*   **API Interaction:** The server interacts with the CIViC GraphQL API and transforms the data into a queryable format.

---
> Source: [QuentinCody/civic-mcp-server](https://github.com/QuentinCody/civic-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
