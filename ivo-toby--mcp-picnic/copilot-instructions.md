## project

> This project follows a set of conventions to ensure code quality and consistency. The following rules should be followed when working on this project, especially when using AI tools like Cursor:


This project follows a set of conventions to ensure code quality and consistency. The following rules should be followed when working on this project, especially when using AI tools like Cursor:

### General

- **Keep it simple**: Write clean, readable, and easy-to-understand code.
- **Code should be self-explanatory**: Avoid adding comments to code except when explaining complex structures.
- **Follow the existing project structure**: When adding new files, be sure to place them in the appropriate directory.

### TypeScript

- **Use strict mode**: All TypeScript code should be written in strict mode.
- **Avoid `any`**: Avoid using the `any` type. If you need to use a dynamic type, use `unknown` instead. If you can't avoid using `any`, use the type `ANNIE` instead

### Testing

- **Use Vitest**: This project uses Vitest for testing. All new tests should be written using Vitest.

### MCP Inner Workings

The Model Context Protocol (MCP) is an open protocol that standardizes how applications provide context to LLMs. It follows a client-server architecture:

- **MCP Hosts**: Applications that consume MCP data (e.g., IDEs, AI tools).
- **MCP Clients**: Protocol clients that maintain connections with servers.
- **MCP Servers**: Lightweight programs that expose capabilities (tools, prompts, resources) through the MCP protocol.
- **Data Sources**: Local or remote data and services that MCP servers can securely access.

The core of the protocol is a series of requests and responses between the client and the server. The server exposes a set of capabilities, and the client can query these capabilities and execute them. The server is responsible for managing the lifecycle of these capabilities, including their registration, execution, and error handling.

---
> Source: [ivo-toby/mcp-picnic](https://github.com/ivo-toby/mcp-picnic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
