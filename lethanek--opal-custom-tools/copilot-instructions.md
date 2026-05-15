## opal-custom-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Build and Development:**
- `yarn build` - Compile TypeScript to JavaScript (outputs to `build/` directory)
- `yarn start` - Run the compiled application from `build/main.js`
- `yarn dev` - Run in development mode with hot reload using tsc-watch

**Package Management:**
- Uses Yarn 4.3.1 as package manager
- Run `yarn install` to install dependencies

## Architecture

This is an Optimizely Opal custom tools service that exposes tools via HTTP endpoints using the `@optimizely-opal/opal-tools-sdk`.

**Core Structure:**
- `src/main.ts` - Main application entry point that sets up Express server and imports tools
- `src/tools/` - Directory containing individual tool implementations
- Express server with CORS enabled serves tools at port 3000 (or PORT env var)
- Tools are registered using the `@tool` decorator pattern from opal-tools-sdk
- Discovery endpoint available at `/discovery`

**Tool Development Pattern:**
1. Create new tool file in `src/tools/` directory
2. Define TypeScript interfaces for tool parameters
3. Implement async function with typed parameters
4. Register tool using `tool()` decorator with explicit parameter definitions
5. Use `ParameterType` enum from SDK for parameter type specification (String, Number, Boolean)
6. Import the tool file in `src/main.ts`

**Current Tools:**
- `greeting` - Multi-language greeting tool (English, Spanish, French)
- `todays-date` - Date formatting tool with multiple format options
- `api_call` - HTTP client wrapper supporting GET, POST, PUT, PATCH, DELETE with custom headers

**TypeScript Configuration:**
- Target: ES2022 with NodeNext modules
- Experimental decorators enabled for tool registration
- Strict type checking enabled
- Hot reload enabled via tsc-watch in development

---
> Source: [lethanek/opal-custom-tools](https://github.com/lethanek/opal-custom-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
