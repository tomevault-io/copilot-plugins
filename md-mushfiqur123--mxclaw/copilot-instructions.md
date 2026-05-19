## mxclaw

> Complete MxClaw: 22 channels, 19 providers, 3 native apps, CI/CD, tests, docs, bundled publishable CLI.

# MxClaw — AGENTS.md

## Goal
Complete MxClaw: 22 channels, 19 providers, 3 native apps, CI/CD, tests, docs, bundled publishable CLI.

## Install Flow
```bash
npm install -g mxclaw@latest
mxclaw onboard            # interactive wizard (providers, channels, agents, daemon)
mxclaw bootstrap          # view/manage API keys in bootstrap.env + bootstrap.json
mxclaw doctor             # validate config
mxclaw gateway            # start server
mxclaw onboard --install-daemon  # auto-start on boot
mxclaw init --clone ./my-mxclaw # clone from GitHub + install + build + onboard
```

## Current State (May 2026)

### Stats
- **681 tests** across 57 files — all passing
- **8 integration/E2E test** files in `test/`
- **239 KB** bundled CLI at `dist/cli.mjs` (all 41 workspace packages inlined, external deps: commander, zod, ws, uuid)
- **pnpm@9.15.4** monorepo, Node >=20, ESBuild bundling
- **54/54 packages build clean** — zero TS errors across all packages
- **54/54 packages have `package.json`** — all publishable as npm modules
- **53/54 packages have test coverage** — only `control-ui` remains untested (React/Vite UI)
- Only remote to publish: `npm publish` (prepublishOnly auto-bundles)

### Full Inventory

#### Core Infrastructure (7 packages, all with package.json, all built)
| Package | Tests | Description |
|---------|-------|-------------|
| `core` | ✅ 99 tests | Types (Zod schemas), config (JSON load/save/watch/snapshot) |
| `cli` | ✅ 6 tests | Commander CLI — 11 subcommands + onboard wizard + daemon mgmt |
| `gateway` | ✅ 54 tests | HTTP/WS server, session manager, agent runner, tool executor, rate limiter, token counter, context engine, webhook verify, model catalog |
| `plugin-system` | ✅ 12 tests | Dynamic plugin loading from packages/ |
| `logging` | ✅ 7 tests | Structured logging with level/subsystem filtering |
| `security` | ✅ 21 tests | AES-256-GCM vault, SecretsManager |
| `memory` | ✅ 19 tests | MemoryAdapter interface + InMemoryMemoryAdapter with JSONL persistence |

#### Channel Packages (22, 6 with package.json)
| Package | pkg.json | Tests | Description |
|---------|----------|-------|-------------|
| `channel-discord` | ✅ | ✅ 16 | Discord bot via WebSocket intents |
| `channel-telegram` | ✅ | ✅ 4 | Telegram bot via Bot API |
| `channel-slack` | ✅ | ✅ 4 | Slack app via Socket Mode |
| `channel-whatsapp` | ✅ | ✅ 2 | WhatsApp via Baileys (QR login) |
| `channel-signal` | ✅ | ✅ 2 | Signal via signal-cli REST API |
| `channel-matrix` | ✅ | ✅ 4 | Matrix homeserver via sync API |
| `channel-imessage` | ✅ | ✅ 4 | iMessage via BlueBubbles (macOS only) |
| `channel-irc` | ✅ | ✅ 2 | IRC via raw TCP socket |
| `channel-googlechat` | ✅ | ✅ 4 | Google Chat via REST API |
| `channel-teams` | ✅ | ✅ 4 | Teams via Bot Framework |
| `channel-feishu` | ✅ | ✅ 4 | Feishu/Lark via Open API |
| `channel-line` | ✅ | ✅ 4 | LINE via Messaging API |
| `channel-mattermost` | ✅ | ✅ 4 | Mattermost via WebSocket API |
| `channel-nextcloud` | ✅ | ✅ 4 | Nextcloud Talk via OCS API |
| `channel-nostr` | ✅ | ✅ 4 | Nostr relay via WebSocket |
| `channel-webchat` | ✅ | ✅ 4 | Embedded WS server |
| `channel-synology` | ✅ | ✅ 4 | Synology Chat REST polling |
| `channel-tlon` | ✅ | ✅ 4 | Tlon/Urbit HTTP SSE+poke |
| `channel-twitch` | ✅ | ✅ 4 | Twitch via IRC |
| `channel-zalo` | ✅ | ✅ 4 | Zalo OA REST API |
| `channel-wechat` | ✅ | ✅ 4 | WeChat Official Account REST |
| `channel-qq` | ✅ | ✅ 2 | QQ OneBot v11 WebSocket |

#### Provider Packages (19, all with package.json, all tested)
| Package | Tests |
|---------|-------|
| `provider-openai-compatible` | ✅ 28 |
| `provider-openai` | ✅ 15 |
| `provider-anthropic` | ✅ 16 |
| `provider-azure` | ✅ 14 |
| `provider-bedrock` | ✅ 28 |
| `provider-cloudflare` | ✅ 13 |
| `provider-cohere` | ✅ 13 |
| `provider-deepseek` | ✅ 14 |
| `provider-fireworks` | ✅ 13 |
| `provider-gemini` | ✅ 19 |
| `provider-groq` | ✅ 14 |
| `provider-huggingface` | ✅ 13 |
| `provider-lmstudio` | ✅ 13 |
| `provider-mistral` | ✅ 14 |
| `provider-ollama` | ✅ 19 |
| `provider-perplexity` | ✅ 13 |
| `provider-replicate` | ✅ 11 |
| `provider-requesty` | ✅ 14 |
| `provider-together` | ✅ 13 |
| `provider-xai` | ✅ 13 |

#### Other Service Packages (6)
| Package | pkg.json | Tests | Description |
|---------|----------|-------|-------------|
| `storage` | ✅ | ✅ 12 | JSONL + SQLite + LanceDB adapters |
| `tools` | ✅ | ✅ 5 | bash, browser, canvas, cron, session spawn, image gen, file r/w |
| `skills` | ✅ | ✅ 6 | Skills system |
| `voice` | ✅ | ✅ 38 | OpenAI Realtime, Gemini Live, ElevenLabs, System TTS |
| `logging` | ✅ | ❌ | Structured logging |
| `control-ui` | ✅ | ❌ | React/Vite web UI |

#### Native Apps (3)
| App | Platform | Language | Features |
|-----|----------|----------|----------|
| `macos` | macOS 14+ | SwiftUI | MenuBarExtra, popover chat, push-to-talk hotkey, WebSocket |
| `ios` | iOS 17+ | SwiftUI | TabView (Chat/Pair/Settings), AVCapture QR, WebSocket |
| `android` | Android API 26+ | Kotlin/Jetpack Compose | OkHttp WebSocket, CameraX QR scanner, Chat/Pair/Settings |

#### CI/CD
| File | Trigger | What it does |
|------|---------|--------------|
| `.github/workflows/ci.yml` | push/PR to main | Matrix build/test (Node 20/22 × Ubuntu/Windows), Docker build |
| `.github/workflows/release.yml` | tag v* | Build, changelog, GitHub Release, npm publish (dry-run) |

#### Docs
| File | Topic |
|------|-------|
| `docs/index.html` | Main docs |
| `docs/getting-started.html` | Getting started |
| `docs/architecture.html` | Architecture |
| `docs/channels.html` | Channel config |
| `docs/providers.html` | Provider config |
| `docs/voice.html` | Voice |
| `docs/security.html` | Security |
| `docs/api.html` | API reference |
| `docs/development.html` | Dev guide |

#### Docker
| File | Details |
|------|---------|
| `Dockerfile` | 3-stage Alpine, Node 22, non-root user |
| `docker-compose.yml` | gateway + redis + control-ui |

### Build Status
- `pnpm -r build` — **54/54 packages build clean** (zero TS errors)
- `node scripts/bundle-cli.mjs` — 239 KB single-file CLI
- `npx vitest run` — **681 tests, 57 files, all passing**

## Known Gaps
1. ~~**35 packages missing package.json**~~ **DONE** — all 54 packages now have package.json
2. ~~**44 packages with no tests**~~ **DONE** — only `control-ui` (React/Vite UI) remains without unit tests
3. **No npm publish yet** — `npm publish` from root (prepublishOnly auto-bundles). Already on npm at v0.1.1 (by mushfiqurh2b, 3 days ago) — update needed.

## Key Decisions
- Bundled CLI approach: esbuild bundles all `@mxclaw/*` packages inline, externalizes npm deps (commander, zod, ws, uuid)
- Root `package.json` is the publishable unit
- Native apps are real Swift/Kotlin implementations (not stubs)

---
> Source: [MD-Mushfiqur123/mxclaw](https://github.com/MD-Mushfiqur123/mxclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
