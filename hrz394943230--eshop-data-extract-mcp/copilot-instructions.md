## eshop-data-extract-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**eshop-data-extract-mcp** — A TypeScript MCP (Model Context Protocol) server for e-commerce data extraction.

## Tech Stack

- TypeScript, Node.js (ES2022, ESM)
- MCP SDK (`@modelcontextprotocol/sdk`)
- Zod for input validation
- Remote: https://cnb.cool/turingdistract/eshop-data-extract-mcp

## Commands

- `npm run build` — Compile TypeScript to `dist/`
- `npm run dev` — Run in dev mode with hot reload (tsx watch)
- `npm start` — Run compiled server

## Architecture

- `src/index.ts` — MCP server entry point, tool definitions
- Transport: stdio (for use with Claude Desktop, Claude Code, etc.)
- Tools are registered via `server.tool()` with Zod schemas for input validation

---
> Source: [hrz394943230/eshop-data-extract-mcp](https://github.com/hrz394943230/eshop-data-extract-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
