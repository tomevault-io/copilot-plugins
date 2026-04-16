## gitlab-mcp

> A Model Context Protocol (MCP) server that provides a comprehensive interface for interacting with GitLab. This server enables LLMs to search projects, manage issues and merge requests, perform advanced code searches, and navigate repository structures.

# GitLab MCP Server

A Model Context Protocol (MCP) server that provides a comprehensive interface for interacting with GitLab. This server enables LLMs to search projects, manage issues and merge requests, perform advanced code searches, and navigate repository structures.

## Project Overview

- **Purpose:** Bridges the gap between LLMs and GitLab's rich feature set using the Model Context Protocol.
- **Main Technologies:** TypeScript, Node.js, `@modelcontextprotocol/sdk`, Axios, Zod, Dotenv.
- **Architecture:**
  - `src/index.ts`: The entry point that initializes the MCP server and registers all tools.
  - `src/tools/`: Domain-specific implementation logic (Issues, MRs, Projects, Notes, Repository, Search).
  - `src/schemas/`: Zod input schemas for strict type safety and LLM documentation.
  - `src/services/`:
    - `gitlab.ts`: Class-based `GitLabService` with connection pooling, 30s TTL caching, and gzip compression.
    - `project-resolver.ts`: Logic for resolving project IDs from names, paths, or custom aliases.
  - `src/types.ts`: Shared TypeScript interfaces for GitLab API responses.

## Key Capabilities

- **Projects:** Search, get metadata (with recent Issues/MRs), and set persistent shorthand shortcuts (e.g., `nds`).
- **Issues:** List, filter, get full details, create, and update (assign/state/labels) issues.
- **Merge Requests:** List, create, update, view diffs, and add inline code review comments.
- **CI/CD Pipelines:** Monitor pipelines, inspect specific jobs, and fetch job trace logs with auto-analysis prompts.
- **Repository:** Fetch raw file contents (with line-range support), fetch multiple files concurrently, follow relative imports, view recursive directory trees, profile the tech stack, and create branches.
- **Advanced Search:** Search for code snippets or specific symbol definitions globally or scoped to groups/projects.

## Development Conventions

- **Tool Naming:** Follow the `gitlab_<action>_<entity>` pattern.
- **Response Formatting:** All tools MUST return concise, human-readable Markdown to minimize token usage and improve clarity for the LLM.
- **Project Resolution:** All project-related tools MUST use `projectResolver.resolve()` to allow the user to provide names, paths, or shortcuts instead of numeric IDs.
- **Caching:** The `GitLabService` transparently caches GET requests for 30 seconds. This can be bypassed with `disableCache: true` in the request config if necessary.
- **Persistence:** Custom shortcuts are saved to `project-cache.json` in the project root.

## Building and Running

### Environment Setup
Create a `.env` file with:
- `GITLAB_PERSONAL_ACCESS_TOKEN`: Your GitLab PAT (requires `api` and `read_api` scopes).
- `GITLAB_API_URL`: (Optional) Defaults to `https://gitlab.com/api/v4`.

### Commands
- **Install Dependencies:** `npm install`
- **Build Project:** `npm run build`
- **Development Mode:** `npm run dev` (Uses `tsx watch`)
- **Run Server (Stdio):** `npm start`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisburlacu1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
