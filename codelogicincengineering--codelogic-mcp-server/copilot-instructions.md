## about-codebase

> General information about the CodeLogic MCP Server codebase and its purpose

- This repository contains a Model Context Protocol (MCP) server that integrates with CodeLogic's knowledge graph APIs
- It enables AI programming assistants to access dependency data from CodeLogic to analyze code and database impacts
- **NEW**: Provides DevOps CI/CD integration capabilities for CodeLogic scanning in Jenkins, GitHub Actions, Azure DevOps, and GitLab CI
- **NEW**: Generates structured data for AI models to directly modify CI/CD files and implement CodeLogic scanning
- The core package is in src/codelogic_mcp_server/ with server.py, handlers.py, and utils.py
- **DevOps Tools**: codelogic-docker-agent, codelogic-build-info, codelogic-pipeline-helper for CI/CD integration

---
> Source: [CodeLogicIncEngineering/codelogic-mcp-server](https://github.com/CodeLogicIncEngineering/codelogic-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
