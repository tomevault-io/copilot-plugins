## my-mastra-agent

> bun dev               # Start development server with hot reload

# CLAUDE.md - my-mastra-agent AI Implementation Rules

## 🚀 Quick Start Commands

```bash
# Essential Development Workflow
bun dev               # Start development server with hot reload
bun build            # Build for production
bun start            # Start production server
bun test             # Run tests
bun run typecheck    # Type check
bun run lint         # Lint code
bun run format       # Format code

# Deployment
./scripts/deploy.sh  # Deploy to Google Cloud Run
```

## 📋 Essential Rules Summary

| Context          | Rule                 | Pattern                        | Forbidden                     |
| ---------------- | -------------------- | ------------------------------ | ----------------------------- |
| **Architecture** | DMMF + ROP           | `chain().andThen().toResult()` | `class`, `throw`, `interface` |
| **Backend**      | Result<T,E> patterns | `ok(data)` / `err(error)`      | Exception throwing            |
| **Types**        | `type` declarations  | `type User = {...}`            | `interface` declarations      |
| **Naming**       | kebab-case files     | `weather-tool.ts`              | camelCase files               |
| **Runtime**      | Bun-first approach   | `Bun.serve()`, `bunx`          | Node.js specific APIs         |

## 📚 Detailed Documentation

### Core Implementation Guides

- **[Backend Implementation](./claude-docs/backend.md)** - Workflows, chains, dependencies, error handling
- **[Error Handling](./claude-docs/error-handling.md)** - Result types, validation patterns

## 🔧 Technology Stack

```yaml
runtime: Bun@1.1.51
language: TypeScript@5.8
api: Express@4.21 + LINE Bot SDK@9.4
ai-framework: Mastra@0.2.33 + Google Gemini AI
validation: Zod@3.23
deployment: Google Cloud Run + Docker
build: Bun bundler + TypeScript compiler
```

## ❌ Critical Anti-Patterns

```typescript
// ❌ Forbidden - Use functions instead
class WeatherService {}

// ❌ Forbidden - Use type instead
interface WeatherData {}

// ❌ Forbidden - Use unknown instead
function process(data: any) {}

// ❌ Forbidden - Use Result<T,E> instead
throw new Error();

// ❌ Forbidden - Use Bun APIs when available
import { readFile } from 'fs/promises'; // Use Bun.file() instead
```

## 🎯 Development Patterns

### Backend (DMMF + ROP with Mastra)

```typescript
// ✅ Workflow pattern with Mastra
export function createWeatherWorkflow(): Workflow {
  return createWorkflow({
    name: "weather-workflow",
    execute: async ({ input }) => {
      return chain(input)
        .andThen(validateInput)
        .asyncAndThen(fetchWeatherData)
        .asyncAndThen(processWithAI)
        .toResult();
    },
  });
}

// ✅ Tool pattern with Result
export const weatherTool = createTool({
  id: "weather-api",
  execute: async (input: WeatherInput): Promise<Result<WeatherData, Error>> => {
    // Implementation returning Result<T,E>
  },
});

// ✅ Agent pattern
export const weatherAgent = createAgent({
  name: "weather-assistant",
  model: gemini15Flash,
  tools: { weatherTool },
  // Agent configuration
});
```

### API Patterns

```typescript
// ✅ Express route with Result pattern
app.get("/api/weather", async (req, res) => {
  const result = await getWeatherWorkflow(req.query.city);
  
  if (result.isOk()) {
    res.json({ success: true, data: result.value });
  } else {
    res.status(400).json({ success: false, error: result.error });
  }
});

// ✅ LINE Bot webhook with error handling
app.post("/api/line/webhook", async (req, res) => {
  const result = await handleLineEvents(req.body.events);
  res.json(result.isOk() ? { status: "ok" } : { status: "error" });
});
```

## 🏗️ Project Structure

```
src/
├── server.ts          # Express server with API routes
├── mastra/
│   ├── index.ts      # Mastra instance configuration
│   ├── agents/       # AI agent definitions
│   ├── tools/        # Tool implementations (weather API)
│   └── workflows/    # Business logic workflows
├── types/            # TypeScript type definitions
├── utils/            # Utility functions
└── config/           # Configuration files

dist/                 # Bundled output
claude-docs/          # AI implementation rules
scripts/              # Deployment and utility scripts
```

## 🐳 Docker & Deployment

```dockerfile
# Multi-stage build pattern
FROM oven/bun:1-alpine AS base
# ... build stages ...

# Production stage
FROM base AS runtime
USER bun
EXPOSE 8080
CMD ["bun", "run", "dist/server.js"]
```

## 🔄 Development Workflow

### Daily Development

1. **Start Development**: `bun dev`
2. **Type Check**: `bun run typecheck`
3. **Test Changes**: `bun test`
4. **Build**: `bun build`
5. **Deploy**: `./scripts/deploy.sh`

### Code Quality Checks

```bash
# Before committing
bun run typecheck
bun run lint
bun run format

# Test workflows
bun test src/mastra/workflows
```

### Environment Variables

```bash
# Required for local development
MASTRA_APP_URL=http://localhost:3000
GOOGLE_GENERATIVE_AI_API_KEY=your-key
WEATHERAPI_API_KEY=your-key
LINE_CHANNEL_SECRET=your-secret
LINE_CHANNEL_ACCESS_TOKEN=your-token
```

## 📖 Glossary

**DMMF**: Domain Modeling Made Functional | **ROP**: Railway Oriented Programming | **Mastra**: AI agent framework | **Bun**: High-performance JavaScript runtime

---

## 🔄 Rule Update Process

### For Developers

1. **Identify Pattern**: When adding new Mastra agents or tools
2. **Document Pattern**: Add examples to `claude-docs/backend.md`
3. **Test Implementation**: Ensure patterns work with Bun runtime
4. **Update Examples**: Keep code examples current with Mastra API

### Maintenance Commands

```bash
# Validate code quality
bun run typecheck
bun run lint

# Test Mastra components
bun test src/mastra

# Build and verify
bun build
bun run dist/server.js
```

---

_Last Updated: 2025-06-28_
_Rule Version: 1.0 (Mastra Agent Project)_

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mitochon9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
