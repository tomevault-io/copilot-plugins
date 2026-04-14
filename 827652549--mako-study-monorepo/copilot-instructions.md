## mako-study-monorepo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **polyrepo-style monorepo** containing multiple independent study projects focused on various JavaScript/TypeScript technologies. Each project is completely independent with its own dependencies, configurations, and build processes. There is no centralized workspace configuration or shared dependencies.

## Project Structure

Each subdirectory is a self-contained project for studying specific technologies:

- **react-playground/** - Modern Next.js 15 + React 19 + Turbopack + Tailwind CSS experiments
- **virtual-list/** - Virtual list/virtualization implementation demos using Next.js and various virtualization libraries
- **langchain-demo/** - AI/ML integration experiments with LangChain, DeepSeek, and OpenAI APIs
- **webpack-demo/** - Webpack bundler configuration and build optimization studies
- **cra-demo/** - Create React App with custom Webpack 4 configuration (legacy React 16)
- **write-ck/** - Kafka and ClickHouse integration for streaming data pipelines
- **lock-study/** - Package manager lock file testing and analysis
- **middleware-use-async-cb/** - Express middleware async callback handling patterns
- **node-module-events/** - Node.js EventEmitter API study

## Common Commands by Project

### Next.js Projects (react-playground, virtual-list)

All Next.js projects use the same command structure:

```bash
cd <project-directory>
pnpm install          # Install dependencies
pnpm dev              # Start development server with Turbopack
pnpm build            # Build for production
pnpm start            # Start production server
pnpm lint             # Run ESLint
```

**Key technologies:**
- Next.js 15.x with App Router (not Pages Router)
- React 19.x
- TypeScript 5.x
- Turbopack for fast builds (`--turbopack` flag in dev mode)
- Tailwind CSS for styling
- Path alias: `@/*` maps to `./src/*`

### Webpack Study Projects (cra-demo, webpack-demo)

```bash
cd cra-demo
npm install
NODE_OPTIONS=--openssl-legacy-provider npx webpack  # Build with legacy OpenSSL
```

**Note:** These projects use legacy React 16 and Webpack 4, requiring the `--openssl-legacy-provider` flag due to OpenSSL 3.x compatibility issues.

### LangChain AI Projects (langchain-demo)

```bash
cd langchain-demo/<subdirectory>  # e.g., deepseek-quick-start
npm install
node <script-name>.js             # Run specific experiments
```

**Key integrations:**
- LangChain with DeepSeek AI provider
- OpenAI SDK
- Multiple AI model providers via custom API endpoints

### Kafka + ClickHouse Project (write-ck)

```bash
cd write-ck
npm install
node <script-name>.ts  # Run data pipeline scripts
```

**Key technologies:**
- KafkaJS and kafka-node for event streaming
- ClickHouse client for analytics database
- Express.js for web server

## Development Workflow

1. **Navigate to project directory** - Each project must be worked on independently
2. **Install dependencies** - Run appropriate package manager command (pnpm for Next.js projects, npm for others)
3. **Check package.json scripts** - Each project may have unique scripts
4. **No shared tooling** - Each project has its own TypeScript config, ESLint config, etc.

## Package Management

- **pnpm** - Preferred for Next.js projects (react-playground, virtual-list)
- **npm** - Used in legacy and experimental projects
- **bun** - Used in some experiments
- `.pnpm-store/` directory is excluded from git

## TypeScript Configuration

Each project has its own `tsconfig.json`. The Next.js projects use:
- `moduleResolution: "bundler"`
- `jsx: "preserve"` (processed by Next.js)
- Path alias `@/*` pointing to `./src/*`
- Next.js plugin for enhanced type checking

## Architecture Notes

- **No workspace dependencies** - Projects do not depend on each other
- **Independent versioning** - Each project can use different versions of the same library
- **Mixed module systems** - Some projects use CommonJS, others use ES modules
- **Experimental nature** - Code quality and patterns vary as these are learning projects

## AI API Configuration

Some projects use custom AI API endpoints. When working with AI integrations, check for:
- Custom base URLs for API providers
- Environment variables for API keys
- Proxy configurations in code

## Build Outputs

Standard exclusions in `.gitignore`:
- `node_modules/` directories
- `dist/` build outputs
- `.next/` Next.js build cache
- `playground.ts` temporary files
- `.pnpm-store/` central package cache

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/827652549)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/827652549)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
