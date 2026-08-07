## zerokey

> OpenAI-compatible AI proxy for DeepSeek, Claude & ChatGPT — no API keys, real browser sessions

# ZeroKey

## PROJECT
 OpenAI-compatible AI proxy for DeepSeek, Claude & ChatGPT — no API keys, real browser sessions
 Node.js >= 18, Express 5, pnpm, SSE streaming

## DIRECTORY
 config/
  constants.js # CONFIG, MODEL_HASH, MODELS — single source of truth for models/ports
 core/
  chat-router.js # buildRouter → per-provider route builder dispatch
  session-selector.js # SessionSelector — TUI wizard for provider/user/session, live-credential validation, rate-limit awareness
  claude/
   api.js # ClaudeAPI — browser-session client, org-id extraction, stream completion, file upload
   stream-handler.js # claudeStreamHandler — SSE parsing, limit detection, summary fallback
   set-instructions.js # setClaudeInstructions — project+system-instructions upsert
  deepseek/
   api.js # DeepSeekAPI — PoW challenge solver, session CRUD, file upload with polling
   stream-handler.js # streamHandler — SSE parsing, auto-retry on stream close
   pow.js # DeepSeekPOW — WASM-based proof-of-work solver
  chatgpt/
   api.js # ChatGPTAPI — sentinel refresh, conduit token, prepare flow, file upload (Azure blob)
   stream-handler.js # chatgptStreamHandler — SSE parsing, session-id tracking
   pow.js # ChatGPTProofOfWork — sentinel proof token decode/generate/solve
  session-selector.js # session-selector.js
 engine/
  compiler.js # ToolCompiler — singleton per IDE×provider: uploadAndGetMessages, uploadAndFormatPrompt, uploadAndFormatPromptForRaw, buildPrompt, compile/parse/emit, matchSkill
  pipeline.js # StreamPipeline — SSE stream head: scanning, emitting, MCP injection, skill handling, error formatting
  instructions.js # Instructions — lazy-loads instructions.md + skills-extra.md, hash for change detection
  instructions.md # Base system prompt (agent rules, BPI syntax, execution model, output contract)
  skills-extra.md # Extra prompt appends (tool grammar, dynamic-tools listing)
  triggers.js # Skills: $cwd, $save, $test, $browser, $mcp, $mcp-dump; MCP auto-registration, passthrough, restore
  tool-defs.js # TOOLS — generic tool grammar + per-IDE mappings (vscode, terax, opencode), output shorteners
  mcp/
   browser.js # BROWSER_MCP — built-in browser MCP alias map
   inject.js # injectMcpAliases — registers MCP tools into compiler.tools
   auto.js # buildAutoAliasMaps, hashTools — auto-registration from mcp_<server>_<tool> naming
   playwright.js # playwrightMCP — Playwright MCP alias map
 routes/
  info.js # GET / — API info
  health.js # GET /health — uptime, user, provider, model
  models.js # GET /v1/models, GET /v1/models/:model — OpenAI-compatible model listing
  claude.js # POST /v1/chat/completions — Claude router: instructions, tools, limit handling
  deepseek.js # POST /v1/chat/completions — DeepSeek router: PoW, session creation, retry
  chatgpt.js # POST /v1/chat/completions — ChatGPT router: sentinel, prepare, instructions
 utils/
  cookie-jar.js # CookieJar — shared cookie store, seed/capture/serialize
  errors.js # classifyError, toOpenAIError — provider error → OpenAI-compatible error
  extract-files.js # decodeContentParts — base64 data-URI → Buffer[] for file upload
  find-port.js # findPort, isPortActive — port scanning
  har-to-capture.js # harToCapture — HAR JSON → network-capture format
  capture-request.js # captureRequest — dumps req.body to temp/captures/*.json ($req skill + unconditional on DeepSeek requests)
  ephemeral-session.js # ephemeralSession — clones session with chatSessionId/parentMessageId nulled, for ephemeral/utility calls
  sequential-queue.js # sequentialQueue — Express middleware serializing all requests through one app instance, one in flight at a time
  session-classifier.js # isRealChatSession — per-IDE fingerprinted system-prompt prefix match; default-deny classifies non-matching system-first calls as ephemeral
  logger.js # console color wrappers (debug, info, success, warn, error)
  rate-limiter.js # acquireSlot — per-provider rate limiting (5 req / 15s window)
  route-helpers.js # validateMessages — shared route middleware
  sse-reader.js # readSSE — generic SSE stream reader for both Web and Node streams
  sync-ide-config.js # syncIdeConfig — writes ZeroKey model entry into VS Code chatLanguageModels.json
 scripts/
  check-modules.js # Dependency integrity check
 server.js # Express app entry: session selection, IDE config sync, route mounting, graceful shutdown

## BUILD
 pnpm 10.13.1
 start: node server.js

## ENTRY-POINTS
 server.js # main entry: node server.js / pnpm start
 start.bat # Windows launcher
 zerokey.bat # npm runner for Windows
 zerokey.sh # npm runner for Unix

## MODULES
 server.js
  → express
  → routes/info, routes/health, routes/models, core/chat-router
  → core/session-selector
  → utils/find-port, utils/sync-ide-config, utils/logger, utils/errors, utils/sequential-queue
 chat-router.js
  → routes/claude, routes/chatgpt, routes/deepseek
 session-selector.js
  → prompts (TUI)
  → core/claude/api, core/deepseek/api, core/chatgpt/api
  → config/constants
 claude.js
  → engine/pipeline (StreamPipeline, passes messages → pipeline.session/rawMode)
  → core/claude/api, core/claude/stream-handler, core/claude/set-instructions
  → utils/rate-limiter, utils/route-helpers
 deepseek.js
  → engine/pipeline (StreamPipeline, passes messages → pipeline.session/rawMode)
  → core/deepseek/api, core/deepseek/stream-handler
  → utils/rate-limiter, utils/route-helpers
 chatgpt.js
  → engine/pipeline (StreamPipeline, passes messages → pipeline.session/rawMode)
  → core/chatgpt/api, core/chatgpt/stream-handler
  → utils/rate-limiter, utils/route-helpers
 pipeline.js
  → engine/compiler (ToolCompiler)
  → engine/triggers (restoreMcpInjections, showAvailableMcpTags, handleSkill)
  → utils/errors (classifyError)
  → utils/session-classifier (isRealChatSession), utils/ephemeral-session (ephemeralSession)
 compiler.js
  → engine/instructions, engine/tool-defs
  → engine/triggers (matchMcpTrigger)
  → utils/extract-files

## RUNTIME-GRAPH
 startup:
  server.js → findPort → SessionSelector.select (TUI wizard)
  → syncIdeConfig (writes VS Code chatLanguageModels.json)
  → buildRouter(selected) → per-provider route mounted at /v1/chat/completions
 per-request (POST /v1/chat/completions):
  sequentialQueue middleware serializes all requests through the mounted router (one in flight at a time, promise-chained)
  route handler → new StreamPipeline(res, session, provider, ide, messages)
  → pipeline ctor: isRealChatSession(ide, messages) classifies real chat turn vs ephemeral utility call (title-gen, tool-optimizer, etc.)
   → real: pipeline.session = session, pipeline.ephemeralMode = false, pipeline.rawMode = !pipeline.toolCalling
   → ephemeral: pipeline.session = ephemeralSession(session) (clone, chatSessionId/parentMessageId nulled, mutations discarded), pipeline.ephemeralMode = true, pipeline.rawMode = true
  → route uses pipeline.session (activeSession) for all chatSessionId/parentMessageId/stream-handler calls downstream
  → route sets pipeline.onFinalChunk (when pipeline.ephemeralMode) to delete the provider-side ephemeral chat session once the response finishes
  → pipeline.setup(messages, tools, req):
   → ephemeralMode: compiler.uploadAndFormatPromptForRaw(messages, pipeline, upload=false) — flat ROLE: content prompt, attachments decoded but not uploaded, skips skill-matching/MCP-tag scan
   → rawMode (non-ephemeral, !toolCalling): compiler.uploadAndFormatPromptForRaw(messages, pipeline, upload=true) — flat prompt, attachments uploaded via pipeline.upload
   → toolCalling: registerAutoMcpServers → restoreMcpInjections → showAvailableMcpTags (new session) → compiler.uploadAndFormatPrompt (uploads attachments, skill check) → buildPrompt
  → acquireSlot (rate limit)
  → providerApi.chatCompletion → stream
  → streamHandler → pipeline.scan (rawMode: emits text straight through, skips BPI tool-call parser; else BPI TOOL parsing, SSE chunk emission)
  → pipeline.sendFinalChunk → activeSession.lastUsed updated (ephemeral clone never persisted to user.sessions) → pipeline.onFinalChunk fires if set

## SCHEMA
 # OpenAI-compatible chat completions (subset)
 POST /v1/chat/completions
  body: {
    model: string,
    messages: [{ role: "system"|"user"|"assistant"|"tool", content: string|array }],
    tools?: [{ type: "function", function: { name, description, parameters } }],
    reasoning_effort?: "low"|"medium"|"high"|"max"
  }
  content parts: { type: "image_url", image_url: { url: "data:mime;base64,..." } } | { type: "file", file: { file_data: "data:mime;base64,...", filename: "..." } }
  response: SSE stream of { id, object: "chat.completion.chunk", created, model, choices: [{ delta: {}, finish_reason }] }

 # Models
 GET /v1/models → { object: "list", data: Model[], activeModel }
 Model: { id, name, object: "model", created, owned_by, context_length, max_output_length }

 # Health
 GET /health → { status, uptime, timestamp, username, provider, model }

 # IDE detection
 Authorization: Bearer <vscode|terax|opencode> (default: vscode)

 # users.json (temp/users.json)
 {
   provider (deepseek|claude|chatgpt): {
     username: {
       username: string,
       parsedFetch: { headers: object, body: object, url: string },
       sessions: [{ name, chatSessionId, parentMessageId, createdAt, lastUsed, toolCalling, vision, model, dynamicToolsHash?, mcpInjected?: object }],
       waitUntil?: number (epoch ms),
       waitReason?: string
     }
   }
 }

## ENV
 PORT # default 7250

## DEPENDENCIES
 express ^5.2.1
 node-fetch ^2.7.0
 prompts ^2.4.2

## CONFIG
 config/constants.js:
  CONFIG.PORT → env PORT or 7250
  MODEL_HASH → per-provider model metadata (id, name, vision, context_length, max_output_length)
  MODELS → flattened model registry keyed by id

## KNOWN-INVARIANTS
 MODELS keyed by meta.id (slug), not display name; MODEL_HASH: id = canonical slug, name = display label
 No API keys — all auth via browser session cookies captured from DevTools fetch()
 SessionSelector._parseFetchDirect extracts URL + headers + body from browser "Copy as fetch" string
 ToolCompiler is a singleton per IDE×provider (cached in ToolCompiler.objects)
 Session state (chatSessionId, parentMessageId, lastUsed, todos) is mutated in-memory; persisted to users.json only on shutdown via selector.flush()
 CookieJar is shared per API client instance; cookies captured from response Set-Cookie headers
 DeepSeek requires PoW challenge per request (WASM-based sha3); retries on SSE error exactly once
 uploadAndFormatPrompt is async, signature (messages, pipeline); returns { prompt, skill }; uploadAndFormatPromptForRaw(messages, pipeline, upload) returns { prompt } only — both share the file-decode/upload loop via uploadAndGetMessages(messages, pipeline, upload)
 buildPrompt signature (userPrompt, pipeline); inlines instructions on new session unless pipeline.haveInstructionsAPI
 skill check happens before provider call; handled in pipeline.setup(), triggering message never reaches provider
 session.mcpInjected populated by restoreMcpInjections from current reqTools; once injected, tags stay for session lifetime
 pipeline.isNewSession, pipeline.toolCalling, pipeline.haveInstructionsAPI, pipeline.ephemeralMode set by StreamPipeline constructor; Claude sets haveInstructionsAPI=true
 Auto MCP registration: mcp_<server>_<tool> naming → $<server> tag, merged into MCP_ALIAS_MAPS
 StreamPipeline defers tool-call emission for terax/opencode (batched at flush), emits immediately for vscode
 Rate limiter: 5 requests per 15-second window per provider label
 Error logs append to temp/errors.txt, rotated at 1MB
 VS Code model sync writes to %APPDATA%/Code/User/chatLanguageModels.json
 sequentialQueue (utils/sequential-queue.js) serializes every /v1/chat/completions request app-wide; no concurrent handling
 Ephemeral chat sessions are deleted provider-side via pipeline.onFinalChunk (set per-route when pipeline.ephemeralMode), fired from pipeline.sendFinalChunkflush(), emits immediately for vscode
 Rate limiter: 5 requests per 15-second window per provider label
 Error logs append to temp/errors.txt, rotated at 1MB
 VS Code model sync writes to %APPDATA%/Code/User/chatLanguageModels.json

## EXTENSION-POINTS
 New IDE: add entry in IDES_PROMPT_OPTIMIZER (tool-defs.js), add IDE name to VALID_IDES (server.js)
 New provider: add BUILDERS entry (chat-router.js), add to SessionSelector provider list + PROVIDER_URLS/PROVIDER_STEPS, add MODEL_HASH entry (constants.js)
 New tool: add entry to TOOLS object (tool-defs.js), add per-IDE mapping
 New skill: add entry to triggers array (triggers.js), with trigger word + bpi template
 Stream pipeline: StreamPipeline owns the SSE lifecycle; ToolCompiler is a stateless service created by StreamPipeline
 MCP integration: tools with mcp_<server>_<tool> naming auto-register as $<server> skill tag
 Dynamic tools: passed via req.body.tools[], hashed per session for change detection
 Agent instructions: edit instructions.md (base) or skills-extra.md (grammar appends)

---
> Source: [downloaddoctor/zerokey](https://github.com/downloaddoctor/zerokey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
