## langchain-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Portable autonomous agent loop using LangChainGo + Ollama/Gemini. Uses JSON tool calling (not ReAct) for reliability with small models.

## Status

**Working:**
- ✅ Agent loop with tool dispatch
- ✅ SSH tool (remote command execution, ssh-agent + interactive password fallback)
- ✅ Shell tool (local command execution)
- ✅ MCP tool (multiple servers, stdio/SSE/HTTP transport, via mark3labs/mcp-go)
- ✅ Conversation history/memory
- ✅ Tool selection rules in prompt
- ✅ Honest error reporting (no hallucination on failures)
- ✅ Wiki RAG tool (Confluence HTML export with diagram support)
- ✅ Ollama backend (local or remote via `--ollama-url`; default model `qwen2.5:32b`)
- ✅ Gemini backend (Google AI, via `--backend gemini`, requires `GOOGLE_API_KEY`)
- ✅ Edge sensor tools (`edge_temp`, `edge_gpio` — SSH-based, portable across Pi and amd64 Linux, via `--edge user@host`)
- ✅ HTTP webhook listener (`--webhook-port N` — `POST /webhook` runs the agent)

**TODO:**
- ✅ Streaming output
- ✅ Event-driven automation (HTTP webhook; cron/file-watch still open)
- [ ] Domain knowledge improvements (command patterns)
- [ ] `edge_camera` tool (SSH capture via libcamera-still / ffmpeg-v4l2 fallback, scp back) — designed in `PLAN-event-sensor.md`, deferred for now

**Out of scope (by design):** Running the agent *on* the Pi (ARM64-resident autonomous agent). This project is a workstation agent that *operates* edge devices remotely over SSH — the `edge_*` tools are SSH verbs in the multi-hop loop. An agent that runs on the Pi itself would be a separate project.

## Use Cases

```
"ssh to x@y.z and tell me what platform it is"   # → ssh tool
"ssh to x@y.z and see why pods are failing"       # → ssh tool
"list running processes"                          # → shell tool
"check disk space"                                # → shell tool
"use mcp to list files in /tmp"                   # → mcp tool (requires --mcp)
"use mcp to read the file /tmp/test.txt"          # → mcp tool (requires --mcp)
"search wiki for deployment architecture"         # → wiki tool (requires --wiki)
"what does the network diagram show"              # → wiki tool (requires --wiki)
"what is the cpu temperature on the edge box"     # → edge_temp tool (requires --edge)
"read gpio pin 17"                                # → edge_gpio tool (requires --edge)
"what is a container?"                            # → direct answer (no tool)
"is Go faster than Python?"                       # → direct answer (no tool)
```

**Note:** MCP requires explicitly saying "mcp" in the prompt. Tool routing keywords are hardcoded in the system prompt (`llm/ollama.go:BuildSystemPrompt`). The MCP routing line is dynamically generated based on registered MCP tool names (`llm/ollama.go:mcpRoutingLine`).

## Backends

Two LLM backends are supported, selected via `--backend`. Both implement the same `llm.ChatClient` + `llm.StreamingChatClient` interface, so all tools (ssh, shell, mcp, wiki) and the agent loop behave identically across backends.

### Ollama (default, local)

```bash
./langchain-agent                       # default model: qwen2.5:32b
./langchain-agent --model llama3.1      # smaller, reliable floor for tool calling
./langchain-agent --model llama3.2      # smaller/faster, less reliable for tool calling
./langchain-agent --model qwen2.5:32b --ollama-url http://big-tower.local:11434  # remote Ollama host
```
- Requires an Ollama server (default `http://localhost:11434`).
- Default model is `qwen2.5:32b` — it needs a decent GPU, so on a modest local box pass `--model llama3.1` (or point `--ollama-url` at a GPU host that has qwen pulled).
- `--ollama-url` points at a remote Ollama host (e.g. a GPU tower on the LAN). Also honors `$OLLAMA_HOST` when the flag is unset. Ignored for the gemini backend.
- The model must be pulled on the target server and expose the `tools` capability for reliable JSON tool calling — `qwen2.5:32b` and `llama3.1` both do; `llama3.2` (3B) works but is less reliable.
- No API key needed.
- `llama3.1` is the recommended floor for reliable JSON tool calling; `qwen2.5:32b` is the default and most reliable when a GPU is available.

### Gemini (Google AI, cloud)

```bash
export GOOGLE_API_KEY="$(tr -d '[:space:]' < ~/gemini-api-key.txt)"
./langchain-agent --backend gemini                       # default model: gemini-2.5-flash
./langchain-agent --backend gemini --model gemini-2.5-pro
```
- Requires `GOOGLE_API_KEY` env var. The `langchaingo/llms/googleai` package reads it automatically.
- Get a key from https://aistudio.google.com/apikey.
- Verified working end-to-end: knowledge questions, shell tool dispatch, and SSH tool dispatch (including conversation context across turns).

**Known quirks:**
- `gemini-2.0-flash` returns HTTP 404 with langchaingo v0.1.14 even though it's listed in the API's models endpoint. Use `gemini-2.5-flash` (the project default) or newer.
- API keys auto-expire after ~30 days of inactivity. The error message is `"API key expired. Please renew the API key."` even when the AI Studio dashboard doesn't flag the key as expired.

## Edge Targets and Event Triggers

### Edge sensor tools (`--edge user@host`)

First-class sensor tools that internally SSH to the configured edge box. Designed to work on **any Linux SSH target** — Raspberry Pi, NUC, mini-PC, x86 thin client — not Pi-specific:

- `edge_temp` — reads `/sys/class/thermal/thermal_zone0/temp` and converts the millidegree integer to `XX.X°C`. Works on Pi 4/5 and any Intel/AMD Linux box exposing the standard thermal sysfs.
- `edge_gpio` — uses libgpiod's `gpioget` / `gpioset`. Default chip is `gpiochip0` (Pi 4 + most SBCs). Pi 5 needs `chip="gpiochip4"`. Will fail cleanly on hosts without GPIO hardware.

**Deferred (future):** `edge_camera` — SSH capture via `libcamera-still` (Pi camera) with an `ffmpeg -f v4l2` fallback for USB webcams, then `scp` the file back. Full design lives in `PLAN-event-sensor.md`; not yet wired into the tool registry.

Both reuse `tools.SSHTool` via `edge_helper.go:defaultSSHExec` (injectable as `sshExecutor` func for tests). System prompt routing is generated by `llm/ollama.go:edgeRoutingLine` only when the tools are registered.

The agent itself runs on the workstation; the edge target is a single host configured once via the flag and shared by all the edge tools.

### HTTP webhook listener (`--webhook-port N`)

Starts an HTTP server in a goroutine alongside the REPL:
- `POST /webhook` — body `{"prompt": "..."}` → runs the agent → response `{"answer": "..."}`
- `GET /health` — liveness probe, returns `OK`

REPL and webhook share the same `Agent`. `agent.Agent.Run()` and `ClearHistory()` are guarded by a `sync.Mutex` to keep the conversation history coherent across concurrent callers.

```bash
./langchain-agent --backend gemini --edge eagle@192.168.1.63 --webhook-port 8090 &
curl -s -X POST http://localhost:8090/webhook \
     -H 'Content-Type: application/json' \
     -d '{"prompt":"what is the cpu temp on the pi"}' | jq .
```

Against a remote Ollama host (the GPU tower serves both the LLM backend and is
the SSH target here). Closing stdin (`< /dev/null`) runs it headless — the REPL
hits EOF and the process stays alive on the webhook keep-alive path:

```bash
./langchain-agent --ollama-url http://big-tower.local:11434 --webhook-port 8090 < /dev/null &
curl -s http://localhost:8090/health                                    # → OK
curl -s -X POST http://localhost:8090/webhook \
     -H 'Content-Type: application/json' \
     -d '{"prompt":"ssh to rathore@big-tower.local and run ollama ps to check what model is running"}' | jq .
# → {"answer":"The model running on the server is qwen2.5:32b ..."}
```

## Build and Test Commands

```bash
go build -o langchain-agent .
./langchain-agent                                    # Run with default model (qwen2.5:32b)
./langchain-agent -model llama3.2                    # Use smaller/faster model (less reliable)
GOOGLE_API_KEY=... ./langchain-agent --backend gemini               # Gemini (default: gemini-2.5-flash)
GOOGLE_API_KEY=... ./langchain-agent --backend gemini --model gemini-2.5-pro  # Gemini with specific model
./langchain-agent --wiki ~/wiki/     # Enable wiki RAG (requires Qdrant)
./langchain-agent --wiki ~/wiki/ --index-only  # Index only, then exit
./langchain-agent --mcp "mcp-filesystem-server /tmp"      # Single MCP server (stdio)
./langchain-agent --mcp "fs:mcp-filesystem-server /tmp"   # Labeled MCP server → tool "mcp_fs"
./langchain-agent --mcp "mcp-filesystem-server /tmp" --mcp "http://localhost:8080"  # Multiple servers
./langchain-agent --mcp "http://localhost:8080/sse"        # SSE transport (URL ending in /sse)
./langchain-agent --mcp "http://localhost:8080"            # Streamable HTTP transport
./langchain-agent --edge eagle@192.168.1.63                # Enable edge_temp/edge_gpio/edge_camera tools (Pi or amd64 Linux)
./langchain-agent --webhook-port 8090                      # HTTP webhook listener: POST /webhook, GET /health

go test ./...                        # Run all tests
go test -v ./agent/...               # Agent loop tests (with mock LLM)
go test -v ./llm/...                 # JSON parsing tests
go test -v ./tools/...               # Tool unit tests
go test -v ./rag/...                 # RAG loader tests
go test -tags integration -v ./tools/...  # MCP integration tests (needs mcp-filesystem-server)
```

## Architecture

```
langchain-agent/
├── main.go              # REPL entry point
├── agent/
│   ├── agent.go         # Agent loop (tool dispatch, history)
│   └── agent_test.go    # Tests with mock LLM client
├── llm/
│   ├── ollama.go        # Ollama client, JSON tool call parsing, shared helpers
│   ├── gemini.go        # Gemini client (Google AI)
│   └── ollama_test.go   # Parsing tests
├── webhook/
│   └── server.go        # HTTP webhook listener (POST /webhook, GET /health)
├── rag/
│   ├── embeddings.go    # Ollama embeddings client (nomic-embed-text)
│   ├── store.go         # Qdrant vector store wrapper
│   ├── loader.go        # Confluence HTML parser
│   ├── vision.go        # LLaVA image description
│   ├── indexer.go       # Wiki indexing orchestration
│   └── loader_test.go   # Loader tests
└── tools/
    ├── tool.go          # Tool interface
    ├── ssh.go           # SSH remote execution
    ├── shell.go         # Local shell execution
    ├── mcp.go           # MCP client (real, via mcp-go SDK)
    ├── wiki.go          # Wiki RAG search tool
    ├── edge_helper.go   # Shared SSH executor for edge_* tools (injectable for tests)
    ├── edge_temp.go     # CPU temp via /sys/class/thermal (Pi + amd64 Linux)
    ├── edge_gpio.go     # GPIO read/write via libgpiod (gpioget/gpioset)
    └── *_test.go        # Tool tests
```

**Key design decisions:**
- JSON tool calling in system prompt (not ReAct) - reliable with llama3.2
- Explicit tool selection rules in prompt to prevent wrong tool choice
- Clear error messages distinguish "no output" from "command failed"
- `llm.ChatClient` interface allows mocking in tests
- SSH auth: tries ssh-agent → key files → interactive password prompt (like `ssh` itself)
- MCP: repeatable `--mcp` flag supports multiple servers with label syntax and auto-naming:
  - `label:command args` → tool name `mcp_<label>` (stdio transport)
  - `http://...` → Streamable HTTP transport; `http://.../sse` → SSE transport
  - No label: first server = `mcp`, subsequent = `mcp2`, `mcp3`, etc.
  - System prompt MCP routing line is dynamically built from registered tool names

## Research Findings

LangChainGo doesn't have first-class Ollama native tool calling in agent framework. Options:
1. **Manual JSON approach** (current) - embed tool schemas in prompt, parse JSON responses
2. ReAct agents - unreliable with small models
3. OpenAIFunctionsAgent - designed for OpenAI, not Ollama

## Key Packages Used

- `github.com/tmc/langchaingo/llms/ollama` - Ollama LLM integration
- `github.com/tmc/langchaingo/llms/googleai` - Google AI (Gemini) integration
- `github.com/tmc/langchaingo/embeddings` - Text embeddings
- `github.com/mark3labs/mcp-go` - MCP client (stdio, SSE, Streamable HTTP transport)
- `golang.org/x/crypto/ssh` - SSH client
- `golang.org/x/term` - Terminal password input (hidden)
- `golang.org/x/net/html` - HTML parsing for Confluence export

## Git

- Do NOT add "Co-Authored-By: Claude" to commits

---
> Source: [iovisor-goodies/langchain-agent](https://github.com/iovisor-goodies/langchain-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
