## google-ads-mcp-dev-project

> The **primary and non-negotiable purpose** of this project is to build a Model Context Protocol (MCP) server that integrates with the Google Ads API. This MCP server is the cornerstone of the entire project architecture and **cannot be replaced or bypassed** with alternative solutions.


# MCP Server Purpose and Implementation Clarification
**Date: 2025-03-18**

## CRITICAL PROJECT DIRECTIVE

### Core Project Purpose

The **primary and non-negotiable purpose** of this project is to build a Model Context Protocol (MCP) server that integrates with the Google Ads API. This MCP server is the cornerstone of the entire project architecture and **cannot be replaced or bypassed** with alternative solutions.

### Why MCP Server is Essential

1. **Core Requirement**: As explicitly stated in all project documentation, the MCP server is the foundation of our solution architecture and the primary deliverable.

2. **Claude Desktop Integration**: The entire workflow depends on Claude Desktop being able to access Google Ads data through the MCP protocol. This integration pattern is fundamental and immutable.

3. **Visualization Through Claude Artifacts**: We are specifically building this system to leverage Claude Artifacts for visualization capabilities, which requires proper MCP implementation.

4. **Technical Specifications**: All technical documents (Backend Schema, Tech Stack, Implementation Plan) are designed around MCP server implementation.

### Prohibited Alternative Approaches

The following approaches **must not be pursued** as they fundamentally contradict our project objectives:

1. ❌ **Direct API Integration**: Do not attempt to have Claude directly access the Google Ads API without an MCP server.

2. ❌ **Custom API Endpoints**: Do not create alternative API endpoints outside the MCP protocol.

3. ❌ **Client-Side Processing**: Do not move server functionality to client-side processing.

4. ❌ **Static Data Solutions**: Do not substitute static data or canned responses in place of a proper MCP server.

5. ❌ **Proxy/Middleware Layers**: Do not introduce proxy layers between Claude and the MCP server that change the fundamental architecture.

### Correct Implementation Approach

The correct implementation must adhere to:

1. ✅ **MCP Protocol Standards**: Follow the Model Context Protocol specification completely.

2. ✅ **Server-Based Architecture**: Implement a proper server that processes requests from Claude Desktop.

3. ✅ **Google Ads API Integration**: Connect to Google Ads API from the server side.

4. ✅ **Server-Side Processing**: Handle all data processing, caching, and formatting on the server.

5. ✅ **Proper Resource Structuring**: Define MCP resources that map to Google Ads data structures.

### Implementation Verification Checklist

All implementation must satisfy these requirements:

- [ ] Uses Python MCP SDK (version 0.4.0) for server implementation
- [ ] Follows MCP protocol for all client-server communication
- [ ] Provides proper resource and tool definitions per MCP protocol
- [ ] Handles authentication with Google Ads API on the server side
- [ ] Returns properly structured data for Claude Artifacts visualization
- [ ] Implements caching mechanism as described in Backend Schema Document

## Conclusion

The MCP server implementation is not optional or substitutable. It is the core project requirement and the primary focus of our development efforts. All work must directly contribute to building, enhancing, and testing the MCP server that integrates with the Google Ads API.

The project's success is defined by delivering a functional MCP server that Claude Desktop can communicate with to retrieve Google Ads data.

---
> Source: [bjorndavidhansen/google-ads-mcp-server](https://github.com/bjorndavidhansen/google-ads-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
