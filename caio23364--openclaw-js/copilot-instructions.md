## openclaw-js

> **Welcome!** If you are an AI Coding Agent tasked to modify, review, or debug this project, read this document first. It contains essential architectural constraints, coding conventions, and project-specific knowledge.

# 🤖 OpenClaw JS — Manual for AI Agents

**Welcome!** If you are an AI Coding Agent tasked to modify, review, or debug this project, read this document first. It contains essential architectural constraints, coding conventions, and project-specific knowledge.

---

## 1. Project Overview

**OpenClaw JS** (v2026.2.14) is a security-hardened, multi-channel personal AI assistant written in TypeScript. It provides a unified interface to interact with multiple AI providers (Anthropic Claude, OpenAI GPT, Google Gemini, DeepSeek, etc.) across various messaging channels (WhatsApp, Telegram, Discord, Slack, Signal, Matrix, WebChat).

### Key Goals
- **Security-first**: Defense-in-depth with origin validation, rate limiting, SSRF protection, sandboxing
- **Multi-provider AI**: Support 15+ AI vendors with unified interface
- **Multi-channel**: Unified inbox across 7+ messaging platforms
- **Local-first**: Gateway binds to localhost by default, data stays local
- **Extensible**: Skills platform for custom tools and integrations

---

## 2. Technology Stack

| Layer | Technology |
|-------|------------|
| **Runtime** | Node.js >= 22.0.0 (ES2022, ES Modules) |
| **Language** | TypeScript 5.7+ with strict mode |
| **Build** | `tsc` (TypeScript compiler) |
| **Testing** | Vitest (48 tests, all passing) |
| **Linting** | ESLint 9.x with TypeScript plugin |
| **Formatting** | Prettier |
| **Logging** | Pino (structured logging) |
| **Gateway** | Express + WebSocket (`ws`) + Socket.IO |

### Dependencies Overview
- **23 production dependencies** (reduced from 50)
- **11 dev dependencies** (reduced from 17)
- **55% dependency reduction** achieved by using native Node.js APIs

---

## 3. Project Structure

```
openclaw-js/
├── src/
│   ├── index.ts              # Main entry point, OpenClaw class
│   ├── agents/               # Agent runtime, session management, tools
│   │   └── index.ts          # AgentRuntime class, built-in tools
│   ├── browser/              # Puppeteer browser automation
│   ├── channels/             # Messaging channel integrations (lazy-loaded)
│   │   ├── index.ts          # ChannelManager
│   │   ├── whatsapp.ts       # @whiskeysockets/baileys
│   │   ├── telegram.ts       # grammy
│   │   ├── discord.ts        # discord.js
│   │   ├── slack.ts          # @slack/bolt
│   │   ├── signal.ts         # signal-cli (child_process)
│   │   ├── matrix.ts         # matrix-js-sdk
│   │   └── webchat.ts        # Socket.IO
│   ├── cli/                  # Command-line interface
│   │   └── index.ts          # Commander-based CLI with all commands
│   ├── cron/                 # Scheduled tasks (node-cron)
│   ├── gateway/              # WebSocket/HTTP gateway
│   │   ├── index.ts          # Gateway class, JSON-RPC protocol
│   │   └── commands.ts       # Chat command parsing (/status, /think, etc.)
│   ├── heartbeat/            # Periodic health checks
│   ├── identity/             # Device identity management
│   ├── memory/               # Agent memory system (sqlite/markdown)
│   ├── metrics/              # Metrics collection and dashboard
│   ├── providers/            # AI provider integrations (lazy-loaded)
│   │   ├── index.ts          # ProviderManager
│   │   ├── vendors.ts        # Vendor registry (15+ providers)
│   │   ├── anthropic.ts      # Anthropic SDK
│   │   ├── openai.ts         # OpenAI SDK (also for OpenAI-compatible)
│   │   ├── google.ts         # Google Generative AI
│   │   ├── failover.ts       # Provider failover logic
│   │   ├── streaming.ts      # Streaming response handling
│   │   └── media.ts          # Media processing for multimodal
│   ├── runtime/              # Docker runtime support
│   ├── security/             # Security controls (CRITICAL)
│   │   ├── index.ts          # RateLimiter, OriginValidator, AuditLogger, InputValidator
│   │   └── sandbox.ts        # Command sandbox, path traversal protection
│   ├── service/              # System service management (systemd/OpenRC)
│   ├── skills/               # Skills platform with audit
│   ├── tunnel/               # Tunnel providers (Cloudflare, ngrok, Tailscale)
│   ├── types/                # TypeScript type definitions
│   │   └── index.ts          # All core types (Session, Message, Agent, etc.)
│   └── utils/                # Utilities
│       ├── config.ts         # Configuration management (~/.openclaw/)
│       ├── logger.ts         # Pino logger setup
│       ├── helpers.ts        # ID generation, StateStore
│       └── index.ts          # Utility exports
├── test/                     # Vitest test suites
│   ├── security.test.ts      # Security module tests (48 tests)
│   ├── agents.test.ts        # Agent runtime tests
│   ├── providers.test.ts     # Provider tests
│   └── setup.ts              # Test setup (auto mock restore)
├── bin/                      # CLI entry point
├── Dockerfile                # Multi-stage Docker build
├── docker-compose.yml        # Docker Compose configuration
├── docker-build.sh           # Docker build script
├── DOCKER.md                 # Docker documentation
├── .dockerignore             # Docker ignore rules
├── package.json              # Dependencies and scripts
├── tsconfig.json             # TypeScript config with path aliases
├── vitest.config.ts          # Vitest configuration
├── .eslintrc.json            # ESLint rules
├── .prettierrc               # Prettier config
└── .env.example              # Environment variables template
```

---

## 4. Build and Test Commands

### Essential Commands (MUST run before committing)

```bash
# Type check (no emit) - MUST PASS
npx tsc --noEmit

# Run all tests (48 tests)
npm test

# Run only security tests (critical)
npx vitest run test/security.test.ts

# Build for production
npm run build

# Development mode (tsx)
npm run dev
```

### Other Commands

```bash
# Linting and formatting
npm run lint
npm run format

# Start gateway
npm run gateway
# or
npm start

# CLI usage
npx openclaw --help
npx openclaw gateway --port 18789
npx openclaw agent --message "Hello"
npx openclaw security audit
```

---

## 5. Code Style Guidelines

### TypeScript Configuration
- **Target**: ES2022
- **Module**: ESNext with Node16 resolution
- **Strict mode**: Enabled
- **Decorators**: Experimental (emitDecoratorMetadata enabled)
- **Source maps**: Enabled for debugging

### Path Aliases (Use these for imports)
```typescript
// Use path aliases instead of relative imports
import { something } from '@utils/config.js';     // src/utils/config.ts
import { Gateway } from '@gateway/index.js';      // src/gateway/index.ts
import { Agent } from '@types/index.js';          // src/types/index.ts
```

Available aliases: `@/*`, `@gateway/*`, `@channels/*`, `@providers/*`, `@agents/*`, `@tools/*`, `@skills/*`, `@utils/*`, `@types/*`

### Code Style (Enforced by ESLint/Prettier)
- **Indent**: 2 spaces
- **Quotes**: Single quotes
- **Semicolons**: Required
- **Line endings**: Unix (LF)
- **Print width**: 100 characters
- **Trailing commas**: ES5 style

### Naming Conventions
- **Files**: kebab-case (e.g., `security.test.ts`)
- **Classes**: PascalCase (e.g., `AgentRuntime`)
- **Interfaces**: PascalCase (e.g., `SessionSettings`)
- **Functions/Variables**: camelCase (e.g., `processMessage`)
- **Constants**: UPPER_SNAKE_CASE for true constants
- **Private members**: Prefix with underscore (e.g., `_privateMethod`)

### Import Conventions
- Always use `.js` extension in imports (ESM requirement)
- Prefer path aliases over relative imports
- Group imports: external libs first, then internal modules, then types

---

## 6. Architecture Deep Dive

### 6.1 Gateway (WebSocket Control Plane)

**File**: `src/gateway/index.ts`

The Gateway is the central WebSocket/HTTP server that:
- Accepts connections from apps, CLI, and web clients
- Implements JSON-RPC protocol (Mission Control compatible)
- Handles authentication with challenge-response
- Routes messages between channels and agents
- Provides REST API endpoints

**Security features**:
- Origin validation (CVE-2026-25253)
- Rate limiting (per IP, per client)
- Input validation and size limits
- Security headers on all responses
- Audit logging for security events

**Key RPC Methods**:
- `connect` - Authentication handshake
- `sessions.list`, `sessions.create`, `sessions.send`
- `agents.list`
- `node.list`, `node.describe`, `node.invoke`

### 6.2 Agent Runtime

**File**: `src/agents/index.ts`

The AgentRuntime manages:
- Agent lifecycle (create, get, configure)
- Session management with TTL cleanup
- Message processing pipeline
- Tool execution with approval system
- Context window management (auto-compact)

**Session TTL**: 30 minutes of inactivity (configurable)
**Max Context**: 50 messages (configurable)

**Built-in Tools**:
- `system.info`, `system.time`
- `sessions.list`, `sessions.send`, `sessions.history`, `sessions.spawn`

### 6.3 Providers (Lazy-Loaded)

**File**: `src/providers/index.ts`, `src/providers/vendors.ts`

ProviderManager handles AI provider integrations:
- **Lazy loading**: SDKs only imported when API key is configured
- **Vendor registry**: 15+ providers in `vendors.ts`
- **Protocol support**: Anthropic, OpenAI, Google
- **OpenAI-compatible**: Most vendors reuse OpenAIProvider with custom baseUrl

**Supported Providers** (prefix for model strings):
- `anthropic/` - Claude 3.x models
- `openai/` - GPT-4o, o1, etc.
- `google/` - Gemini models
- `deepseek/`, `groq/`, `openrouter/`, `ollama/`, etc.

### 6.4 Channels (Lazy-Loaded)

**File**: `src/channels/index.ts`

ChannelManager handles messaging integrations:
- Lazy-loads SDKs only when channel is enabled
- Supports 7 channel types
- Unified message format across all channels

### 6.5 Security Module (CRITICAL)

**File**: `src/security/index.ts`

Centralized security controls addressing specific CVEs:

| Class/Function | CVE | Purpose |
|---------------|-----|---------|
| `OriginValidator` | CVE-2026-25253 | WebSocket origin validation |
| `RateLimiter` | - | Token-bucket rate limiting |
| `AuditLogger` | - | Ring-buffer security event logging |
| `InputValidator` | - | Message size/structure validation |
| `isUrlSafe()` | CVE-2026-25255 | SSRF protection |
| `isSafeHostname()` | CVE-2026-25157 | SSH hostname injection prevention |
| `sanitizeEnvPath()` | CVE-2026-24763 | PATH manipulation defense |
| `isCommandSafe()` | CVE-2026-24763 | Dangerous command blocking |
| `isPathAllowed()` | CVE-2026-25256 | Path traversal protection |

**NEVER disable these security controls without explicit human permission.**

---

## 7. Testing Strategy

### Test Framework: Vitest

**Configuration**: `vitest.config.ts`
- **Pool**: `forks` (isolated processes)
- **Workers**: 4-16 local, 2-3 in CI
- **Timeout**: 30s (60s hooks on Windows)
- **Setup**: `test/setup.ts` (auto mock restore)

### Test Structure
```
test/
├── security.test.ts    # 400+ lines, comprehensive security tests
├── agents.test.ts      # Agent runtime tests
├── providers.test.ts   # Provider integration tests
├── skills.test.ts      # Skills platform tests
└── ...
```

### Running Tests
```bash
# All tests
npm test

# Specific test file
npx vitest run test/security.test.ts

# Watch mode
npx vitest

# With coverage
npx vitest run --coverage
```

### Test Requirements
- All 48 tests MUST pass before committing
- Security tests are especially critical
- Use `vi.restoreAllMocks()` in `afterEach` (handled by setup.ts)

---

## 8. Configuration System

**File**: `src/utils/config.ts`

Configuration is stored in `~/.openclaw/config.json`:

```typescript
interface OpenClawConfig {
  gateway: GatewayConfig;      // WebSocket server settings
  channels: Record<string, ChannelConfig>;
  agents: Record<string, AgentConfig>;
  providers: Record<string, ProviderConfig>;
  sandbox: SandboxConfig;      // Security sandbox
  memory: MemoryConfig;        // Memory backend
  // ... more
}
```

**Key directories**:
- `~/.openclaw/` - Config root
- `~/.openclaw/state/` - Runtime state
- `~/.openclaw/logs/` - Log files
- `~/.openclaw/skills/` - Custom skills
- `~/.openclaw/workspace/` - Sandbox workspace

**Environment variables** (see `.env.example`):
- `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_API_KEY`, etc.
- Channel tokens: `TELEGRAM_BOT_TOKEN`, `DISCORD_BOT_TOKEN`, etc.

### OpenClaw Configurator (index.html)

Para facilitar a configuração, use o **OpenClaw Configurator** — uma interface web visual:

https://caio23364.github.io/openclaw-js/

```bash
# Abra diretamente no navegador
open index.html

# Ou sirva com um servidor local
npx serve index.html
```

**Funcionalidades do Configurator:**
- 🎨 Interface moderna com toggle dark/light mode
- 🔐 Suporte a 15+ provedores de IA (incluindo novos: GLM, Moonshot, Qwen, NVIDIA, Cerebras, Volcengine)
- 💬 Configuração completa de 7 canais (Telegram, Discord, Slack, Matrix, Signal)
- ⚙️ Configuração do Gateway (porta, host, auth mode, JWT secret)
- 📦 Múltiplos formatos de exportação:
  - `.env` — arquivo de ambiente padrão
  - `docker-compose.yml` — para deploy Docker
  - `openclaw.service` — para systemd
  - `start.sh` — script de inicialização
- 📥 Importação de arquivos `.env` existentes
- 🔍 Filtro de configurações por busca
- 🎨 Syntax highlighting no preview

**Variáveis suportadas no configurator:**
- **AI Providers**: Anthropic, OpenAI, Google, DeepSeek, Groq, OpenRouter, GLM, Moonshot, Qwen, NVIDIA, Cerebras, Volcengine
- **Local**: Ollama, VLLM, llama.cpp, Osaurus (MLX)
- **Channels**: Telegram, Discord, Slack, Matrix, Signal
- **Gateway**: Port, Host, Auth Mode, JWT Secret
- **Features**: Log Level, Memory Backend, Browser, Cron
- **Custom Providers**: Provedores OpenAI-compatible dinâmicos

---

## 9. Security Considerations

### Mandatory Security Checks

1. **Never hardcode secrets** - Always use environment variables or config
2. **Always validate inputs** - Use `InputValidator` for user input
3. **Check origins** - Gateway validates WebSocket origins
4. **Rate limit** - Both connection and message rate limits active
5. **Sandbox tools** - Tools requiring approval are BLOCKED by default
6. **Log security events** - Use `AuditLogger` for security-relevant events

### Security Headers
All HTTP responses include:
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Content-Security-Policy`
- `Referrer-Policy`

### Tool Approval System
Tools in `requireApproval` list are **blocked** until explicitly approved:
```typescript
// This will BLOCK execution and return error
if (tool.requireApproval) {
  return {
    success: false,
    error: `Tool requires explicit user approval`
  };
}
```

---

## 10. Development Workflow

### Before Making Changes
1. Read relevant code in `src/` directory
2. Check existing types in `src/types/index.ts`
3. Review tests in `test/` for expected behavior

### Making Changes
1. Follow code style guidelines
2. Use path aliases for imports
3. Add/update tests as needed
4. Handle errors with `log.error()` or `log.warn()`

### Before Committing
1. **Type check**: `npx tsc --noEmit` (MUST pass)
2. **Run tests**: `npx vitest run test/security.test.ts` (MUST pass)
3. **Run all tests**: `npm test` (MUST pass)
4. **Check security**: Review any changes to `src/security/`

### Adding New Features

**New Provider**:
1. Add vendor to `VENDOR_REGISTRY` in `src/providers/vendors.ts`
2. If not OpenAI-compatible, create provider class
3. Update `ProviderManager` if needed

**New Channel**:
1. Create channel class in `src/channels/`
2. Export from `src/channels/index.ts`
3. Add to `createChannelInstance()` switch

**New Command**:
1. Add to `src/gateway/commands.ts`
2. Update `parseChatCommand()` function

**New Tool**:
1. Add to `builtInTools` array in `src/agents/index.ts`
2. Define parameters, handler, approval requirement

---

## 11. Common Patterns

### Singleton Pattern
Most modules use singleton pattern with getter/creator:
```typescript
let instance: Class | null = null;

export function getInstance(): Class {
  if (!instance) throw new Error('Not initialized');
  return instance;
}

export function createInstance(): Class {
  instance = new Class();
  return instance;
}
```

### Lazy Loading
Providers and channels use dynamic imports:
```typescript
const { SomeProvider } = await import('./some-provider.js');
```

### Error Handling
Always log errors, don't swallow silently:
```typescript
try {
  await someOperation();
} catch (error) {
  log.error('Operation failed:', error);
  throw error; // or return error result
}
```

### Logging
Use the structured logger:
```typescript
import { log } from '@utils/logger.js';

log.info('Starting service');
log.warn('Deprecated feature used');
log.error('Operation failed', error);
```

---

## 12. Troubleshooting

### Common Issues

**Type errors on build**:
```bash
# Check TypeScript version (must be 5.7+)
npx tsc --version

# Clean build
rm -rf dist && npm run build
```

**Tests failing**:
```bash
# Reset vitest cache
npx vitest --clearCache

# Run with more verbose output
npx vitest run --reporter=verbose
```

**Module resolution errors**:
- Ensure imports use `.js` extension
- Check path aliases match `tsconfig.json`

---

## 13. Custom Providers (OpenAI-Compatible)

OpenClaw JS supports adding custom OpenAI-compatible providers dynamically via environment variables. This allows you to use any provider that implements the OpenAI API protocol (v1) without modifying code.

### Configuration Format

```bash
# List your custom provider prefixes (comma-separated)
CUSTOM_PROVIDERS=together,fireworks,perplexity

# For each prefix, define:
<prefix>_NAME="Display Name"           # Human-readable name
<prefix>_BASE_URL="https://api..."     # API base URL (required)
<prefix>_API_KEY="your-key"            # API key (optional)
<prefix>_MODELS="model1,model2"        # Supported models (optional)
```

### Examples

**Together AI:**
```bash
CUSTOM_PROVIDERS=together
TOGETHER_NAME="Together AI"
TOGETHER_BASE_URL=https://api.together.xyz/v1
TOGETHER_API_KEY=your-together-key
```

**Fireworks AI:**
```bash
CUSTOM_PROVIDERS=fireworks
FIREWORKS_NAME="Fireworks AI"
FIREWORKS_BASE_URL=https://api.fireworks.ai/inference/v1
FIREWORKS_API_KEY=your-fireworks-key
```

**Perplexity:**
```bash
CUSTOM_PROVIDERS=perplexity
PERPLEXITY_NAME="Perplexity"
PERPLEXITY_BASE_URL=https://api.perplexity.ai
PERPLEXITY_API_KEY=your-perplexity-key
```

**Multiple providers:**
```bash
CUSTOM_PROVIDERS=together,fireworks,perplexity

# Together AI
TOGETHER_NAME="Together AI"
TOGETHER_BASE_URL=https://api.together.xyz/v1
TOGETHER_API_KEY=tgp_v1_...

# Fireworks AI
FIREWORKS_NAME="Fireworks AI"
FIREWORKS_BASE_URL=https://api.fireworks.ai/inference/v1
FIREWORKS_API_KEY=f1_...

# Perplexity
PERPLEXITY_NAME="Perplexity"
PERPLEXITY_BASE_URL=https://api.perplexity.ai
PERPLEXITY_API_KEY=pplx-...
```

### Usage

Once configured, use the custom provider with the prefix notation:

```bash
# CLI
openclaw agent -m "Hello" --model together/llama-3.1-70b

# Or in chat
/model together/llama-3.1-70b
```

### Supported Providers

Any provider implementing the OpenAI API v1 protocol:

- **Together AI** (together.xyz)
- **Fireworks AI** (fireworks.ai)
- **Perplexity** (perplexity.ai)
- **Anyscale** (anyscale.com)
- **Baseten** (baseten.co)
- **Replicate** (replicate.com)
- **OctoAI** (octo.ai)
- **Your own self-hosted models** (vLLM, llama.cpp, etc.)

### Implementation Details

- Custom providers use the same `OpenAIProvider` class as built-in OpenAI-compatible vendors
- They are lazy-loaded on first use or during ProviderManager initialization
- Custom providers appear in status endpoints with `isCustom: true`
- API keys can be provided via environment variables or config file

---

## 14. Docker Deployment

The project includes full Docker support with multi-stage builds and multi-platform support.

### Platform Support

| Platform | Architecture | Notes |
|----------|-------------|-------|
| Linux | AMD64 (x86_64) | Standard servers, Intel/AMD CPUs |
| Linux | ARM64 (aarch64) | Apple Silicon (M1/M2/M3/M4), Raspberry Pi 4/5, AWS Graviton |
| macOS | AMD64 | Intel Macs via Docker Desktop |
| macOS | ARM64 | Apple Silicon Macs via Docker Desktop (native) |

### Quick Start with Docker

```bash
# 1. Configure environment
cp .env.example .env
# Edit .env with your API keys

# 2. Run with Docker Compose (recommended)
docker-compose up -d

# 3. Or build and run manually
./docker-build.sh
docker run -d --name openclaw -p 18789:18789 --env-file .env openclaw-js:latest
```

### Docker Files

| File | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage build (Node.js 22 + Chromium) |
| `docker-compose.yml` | Complete stack configuration |
| `.dockerignore` | Files excluded from build context |
| `docker-build.sh` | Build script with tagging |
| `DOCKER.md` | Complete Docker documentation |

### Key Docker Features

- **Multi-stage build**: Compiles TypeScript in builder stage, copies only `dist/` to production image
- **Security**: Runs as non-root user (`nodejs`), includes security headers
- **Puppeteer support**: Includes Chromium from Alpine packages
- **Health check**: Configured HTTP health endpoint check
- **Volume persistence**: `~/.openclaw/` data persisted in Docker volume

### Environment Variables in Docker

All variables from `.env.example` are supported. Key ones:

```bash
# Required: at least one AI provider
ANTHROPIC_API_KEY=...
OPENAI_API_KEY=...

# Gateway binds to 0.0.0.0 inside container
GATEWAY_HOST=0.0.0.0
GATEWAY_PORT=18789

# Puppeteer uses system Chromium
PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
```

### Connecting to External Services

**Ollama on host machine**:
```bash
OLLAMA_BASE_URL=http://host.docker.internal:11434/v1
```

**Ollama in Docker Compose**:
Uncomment the `ollama` service in `docker-compose.yml`.

### Multi-Platform Builds

Build for multiple architectures using Docker Buildx:

```bash
# Build for current architecture only
docker build -t openclaw-js .

# Build for specific platform
docker buildx build --platform linux/arm64 -t openclaw-js .

# Build for multiple platforms (requires registry push)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t username/openclaw-js:latest \
  --push .

# Using docker-bake.hcl
docker buildx bake
```

### Apple Silicon (M1/M2/M3/M4) Notes

On Apple Silicon Macs, Docker Desktop automatically runs ARM64 containers natively:

```bash
# No special flags needed - Docker auto-detects
docker-compose up -d

# Verify you're running ARM64 version
docker exec openclaw uname -m  # Should print: aarch64
```

**Performance benefits:**
- Native ARM64 execution (no Rosetta translation)
- Lower memory usage
- Better battery life on laptops

### Docker Commands Reference

```bash
# Build
docker-compose build
./docker-build.sh [tag]

# Run
docker-compose up -d
docker run -d --name openclaw -p 18789:18789 openclaw-js

# Logs
docker-compose logs -f
docker logs -f openclaw

# Stop
docker-compose down
docker stop openclaw

# Shell access
docker exec -it openclaw sh
```

---

## 15. Termux (Android) Deployment

OpenClaw JS can run on Android devices via [Termux](https://termux.dev/), a terminal emulator and Linux environment for Android.

### Quick Install (One Command)

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-js/main/install-termux.sh | bash
```

This script will:
1. Update Termux packages
2. Install Node.js LTS, git, python, build tools
3. Clone the repository to `~/openclaw-js`
4. Install npm dependencies
5. Create initial `.env` configuration
6. Apply Termux-specific settings:
   - Disable browser automation (no Chrome on Android)
   - Bind gateway to `0.0.0.0` for network access
   - Skip Puppeteer Chrome download
7. Create convenient shortcuts (`openclaw` command)

### Post-Install

```bash
# Configure API keys
nano ~/openclaw-js/.env

# Start OpenClaw
openclaw
# or
cd ~/openclaw-js && npm start
```

### Accessing the Gateway

On Termux, the gateway binds to `0.0.0.0:18789`, allowing access from:
- Local device: `http://localhost:18789`
- Network devices: `http://<phone-ip>:18789`

Find your IP:
```bash
ifconfig
# or
ip addr
```

### Termux Tips

- **Keep running in background**: Run `termux-wake-lock` before starting OpenClaw
- **Home screen shortcut**: Install Termux:Widget, then use the `openclaw` shortcut
- **Update**: Run `~/openclaw-js/update-termux.sh`
- **Battery optimization**: Disable battery optimization for Termux in Android settings

### Limitations on Termux

- Browser automation (Puppeteer) is disabled
- Some native modules may compile slower on ARM devices
- WhatsApp QR scanning works best with external camera apps

---

## Appendix: Dependency Audit Summary

**Last Audit**: 2026-02-18

**Production**: 50 → 23 dependencies (**54% reduction**)
**Dev**: 17 → 11 dependencies (**35% reduction**)

**Key removed packages**: axios (use native fetch), dotenv (use `--env-file`), uuid (use `crypto.randomUUID()`), winston (use pino), zod (not used), bcryptjs, jsonwebtoken, cheerio, jsdom, and many more.

**See original `AGENTS.md` Appendix A for full details.**

---

**Document Version**: 2026.2.14  
**Last Updated**: 2026-02-25 (Configurator v2 adicionado)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Caio23364) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
