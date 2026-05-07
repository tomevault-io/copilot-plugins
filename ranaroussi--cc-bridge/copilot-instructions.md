## cc-bridge

> handleStreamingRequest(w, req)

# CLAUDE.md - CC-Bridge Development Guide

## Project Overview

**CC-Bridge** is an HTTP service that provides 100% Anthropic API compatibility while using the official Claude Code CLI under the hood. This preserves OAuth access legitimately without authentication hacks.

**Problem:** Anthropic is shutting down third-party OAuth access. Apps using OAuth tokens directly now fail with:
```
LLM request rejected: This credential is only authorized for use with
Claude Code and cannot be used for other API requests.
```

**Solution:** Wrap `claude -p` (Claude CLI print mode) as an HTTP service. Let the official CLI handle authentication.

---

## Quick Start

### Prerequisites

1. **Claude Code CLI installed**:
   ```bash
   npm install -g claude-code
   # Verify
   claude --version
   ```

2. **Claude Code authenticated**:
   ```bash
   claude setup-token
   # Follow OAuth flow or provide API key
   ```

3. **Go 1.21+**:
   ```bash
   go version
   ```

### Development Setup

```bash
# Clone/create project
cd cc-bridge

# Build
make build

# Run server (default port 8321)
./build/ccbridge

# Or with custom port
./build/ccbridge --port 9000

# Test
curl -X POST http://localhost:8321/v1/messages \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-sonnet-4-20250514", "messages": [{"role": "user", "content": "Hello"}]}'
```

---

## Architecture

```
Client (Anthropic SDK)
  ↓ POST /v1/messages
Go HTTP Server (cc-bridge)
  ↓ spawn("claude", ["-p", ...])
Claude CLI
  ↓ OAuth or API key
Anthropic API
```

**Key insight:** We're not fighting OAuth - we're delegating auth to the official tool that's allowed to use it.

---

## Implementation Guide

### Phase 1: Non-Streaming MVP (10-12 hours)

**Goal:** Basic request/response works with Anthropic SDK

#### File Structure
```
cc-bridge/
├── src/
│   ├── main.go          # HTTP server entry point
│   ├── handler.go       # Request handler
│   ├── types.go         # Type definitions
│   ├── cli.go           # Execute claude -p
│   ├── mapper.go        # Map API params ↔ CLI flags
│   ├── streaming.go     # SSE streaming support
│   └── *_test.go        # Tests
├── build/               # Compiled binaries
├── Makefile             # Build commands
└── go.mod
```

#### main.go
```go
package main

import (
    "log"
    "net/http"
    "os"
)

func main() {
    port := os.Getenv("CC_BRIDGE_PORT")
    if port == "" {
        port = "8080"
    }

    http.HandleFunc("/v1/messages", handleMessages)
    http.HandleFunc("/health", handleHealth)

    log.Printf("CC-Bridge starting on port %s", port)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}

func handleHealth(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("OK"))
}
```

#### handler.go
```go
func handleMessages(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    // 1. Parse Anthropic request
    var req AnthropicRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeErrorResponse(w, "invalid_request_error", err.Error())
        return
    }

    // 2. Check if streaming
    if req.Stream {
        handleStreamingRequest(w, req)
        return
    }

    // 3. Execute CLI
    cliResp, err := executeCLI(req)
    if err != nil {
        writeErrorResponse(w, "api_error", err.Error())
        return
    }

    // 4. Format response
    anthropicResp := formatAnthropicResponse(cliResp, req)

    // 5. Return JSON
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(anthropicResp)
}
```

#### request.go
```go
type AnthropicRequest struct {
    Model       string          `json:"model"`
    Messages    []Message       `json:"messages"`
    System      string          `json:"system,omitempty"`
    MaxTokens   int             `json:"max_tokens,omitempty"`
    Temperature float64         `json:"temperature,omitempty"`
    Tools       []Tool          `json:"tools,omitempty"`
    Stream      bool            `json:"stream,omitempty"`
}

type Message struct {
    Role    string `json:"role"`
    Content string `json:"content"`
}

type Tool struct {
    Name        string                 `json:"name"`
    Description string                 `json:"description"`
    InputSchema map[string]interface{} `json:"input_schema"`
}
```

#### cli.go
```go
import (
    "bytes"
    "encoding/json"
    "os/exec"
    "strings"
)

func executeCLI(req AnthropicRequest) (*CLIResponse, error) {
    // Build command args
    args := []string{"-p"}

    // Model
    args = append(args, "--model", mapModelName(req.Model))

    // Output format
    args = append(args, "--output-format", "json")

    // Temperature
    if req.Temperature > 0 {
        args = append(args, "--temperature", fmt.Sprintf("%.1f", req.Temperature))
    }

    // System prompt
    if req.System != "" {
        args = append(args, "--append-system-prompt", req.System)
    }

    // Tools
    if len(req.Tools) > 0 {
        toolNames := mapToolNames(req.Tools)
        args = append(args, "--tools", strings.Join(toolNames, ","))
    }

    // Build prompt from messages
    prompt := buildPrompt(req.Messages)

    // Execute
    cmd := exec.Command("claude", args...)
    cmd.Stdin = strings.NewReader(prompt)

    var stdout, stderr bytes.Buffer
    cmd.Stdout = &stdout
    cmd.Stderr = &stderr

    if err := cmd.Run(); err != nil {
        return nil, fmt.Errorf("CLI error: %s", stderr.String())
    }

    // Parse output
    var cliResp CLIResponse
    if err := json.Unmarshal(stdout.Bytes(), &cliResp); err != nil {
        return nil, fmt.Errorf("parse error: %w", err)
    }

    return &cliResp, nil
}

type CLIResponse struct {
    Type      string  `json:"type"`
    Subtype   string  `json:"subtype"`
    IsError   bool    `json:"is_error"`
    Result    string  `json:"result"`
    SessionID string  `json:"session_id"`
    Usage     Usage   `json:"usage"`
}

type Usage struct {
    InputTokens  int `json:"input_tokens"`
    OutputTokens int `json:"output_tokens"`
}
```

#### mapper.go
```go
// Map Anthropic model names to CLI short names
func mapModelName(model string) string {
    switch {
    case strings.Contains(model, "opus"):
        return "opus"
    case strings.Contains(model, "sonnet"):
        return "sonnet"
    case strings.Contains(model, "haiku"):
        return "haiku"
    default:
        return "sonnet" // default
    }
}

// Build prompt from message array
func buildPrompt(messages []Message) string {
    var parts []string
    for _, msg := range messages {
        parts = append(parts, msg.Content)
    }
    return strings.Join(parts, "\n\n")
}

// Map tool schemas to CLI tool names
func mapToolNames(tools []Tool) []string {
    // Simple mapping - expand as needed
    toolMap := map[string]string{
        "bash":  "Bash",
        "read":  "Read",
        "write": "Write",
        "grep":  "Grep",
        "glob":  "Glob",
    }

    var names []string
    for _, tool := range tools {
        if cliName, ok := toolMap[strings.ToLower(tool.Name)]; ok {
            names = append(names, cliName)
        }
    }
    return names
}
```

#### response.go
```go
type AnthropicResponse struct {
    ID          string         `json:"id"`
    Type        string         `json:"type"`
    Role        string         `json:"role"`
    Content     []ContentBlock `json:"content"`
    Model       string         `json:"model"`
    StopReason  string         `json:"stop_reason"`
    Usage       Usage          `json:"usage"`
}

type ContentBlock struct {
    Type string `json:"type"`
    Text string `json:"text,omitempty"`
}

func formatAnthropicResponse(cliResp *CLIResponse, req AnthropicRequest) *AnthropicResponse {
    return &AnthropicResponse{
        ID:   generateMessageID(),
        Type: "message",
        Role: "assistant",
        Content: []ContentBlock{
            {Type: "text", Text: cliResp.Result},
        },
        Model:      req.Model,
        StopReason: "end_turn",
        Usage:      cliResp.Usage,
    }
}

func generateMessageID() string {
    // Simple UUID or random ID
    return "msg_" + randomString(24)
}
```

---

### Phase 2: Streaming Support (8-10 hours)

**Goal:** SSE streaming works

#### streaming.go
```go
func handleStreamingRequest(w http.ResponseWriter, req AnthropicRequest) {
    // Set SSE headers
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")

    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }

    // Execute CLI with stream-json
    args := buildStreamingArgs(req)
    cmd := exec.Command("claude", args...)
    cmd.Stdin = strings.NewReader(buildPrompt(req.Messages))

    stdout, _ := cmd.StdoutPipe()
    cmd.Start()

    scanner := bufio.NewScanner(stdout)

    // Send message_start
    writeSSEEvent(w, "message_start", createMessageStart(req))
    flusher.Flush()

    contentBlockIndex := 0
    sentBlockStart := false

    for scanner.Scan() {
        line := scanner.Text()

        var event CLIStreamEvent
        if err := json.Unmarshal([]byte(line), &event); err != nil {
            continue
        }

        // Transform CLI event to Anthropic SSE event
        if event.Type == "stream_event" {
            delta := event.Event.Delta
            if delta.Text != "" {
                if !sentBlockStart {
                    writeSSEEvent(w, "content_block_start", createContentBlockStart(contentBlockIndex))
                    sentBlockStart = true
                }

                writeSSEEvent(w, "content_block_delta", createContentBlockDelta(contentBlockIndex, delta.Text))
                flusher.Flush()
            }
        }
    }

    // Send final events
    writeSSEEvent(w, "content_block_stop", createContentBlockStop(contentBlockIndex))
    writeSSEEvent(w, "message_delta", createMessageDelta())
    writeSSEEvent(w, "message_stop", createMessageStop())
    flusher.Flush()

    cmd.Wait()
}

func writeSSEEvent(w http.ResponseWriter, eventType string, data interface{}) {
    jsonData, _ := json.Marshal(data)
    fmt.Fprintf(w, "event: %s\n", eventType)
    fmt.Fprintf(w, "data: %s\n\n", jsonData)
}
```

---

## Testing Strategy

### Manual Testing with Anthropic SDK

**Node.js test:**
```javascript
// test.js
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: 'dummy-ignored',
  baseURL: 'http://localhost:8321',
});

// Non-streaming
const message = await client.messages.create({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Hello!' }],
});
console.log(message.content[0].text);

// Streaming
const stream = await client.messages.create({
  model: 'claude-haiku-3-5-20241022',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Count to 5' }],
  stream: true,
});

for await (const event of stream) {
  if (event.type === 'content_block_delta') {
    process.stdout.write(event.delta.text);
  }
}
```

### Integration Tests

```go
func TestNonStreamingRequest(t *testing.T) {
    // Start test server
    server := httptest.NewServer(http.HandlerFunc(handleMessages))
    defer server.Close()

    // Make request
    reqBody := `{
        "model": "claude-sonnet-4-20250514",
        "messages": [{"role": "user", "content": "Say hello"}],
        "max_tokens": 1024
    }`

    resp, err := http.Post(
        server.URL+"/v1/messages",
        "application/json",
        strings.NewReader(reqBody),
    )

    assert.NoError(t, err)
    assert.Equal(t, http.StatusOK, resp.StatusCode)

    var anthropicResp AnthropicResponse
    json.NewDecoder(resp.Body).Decode(&anthropicResp)

    assert.Equal(t, "message", anthropicResp.Type)
    assert.Equal(t, "assistant", anthropicResp.Role)
    assert.NotEmpty(t, anthropicResp.Content[0].Text)
}
```

---

## Debugging Tips

### Enable Debug Logging

```go
type DebugLogger struct {
    Enabled bool
}

func (d *DebugLogger) Log(format string, args ...interface{}) {
    if d.Enabled {
        log.Printf("[DEBUG] "+format, args...)
    }
}

// Usage
debug := &DebugLogger{Enabled: os.Getenv("CC_BRIDGE_DEBUG") == "true"}
debug.Log("Executing CLI: %v", args)
```

### Inspect CLI Output

```bash
# Test CLI directly
echo "Hello" | claude -p --model sonnet --output-format json

# Test streaming
echo "Count to 5" | claude -p --model haiku --output-format stream-json --verbose
```

### Common Issues

**1. "claude: command not found"**
```bash
# Check PATH
which claude

# Set explicit path
export CLAUDE_CLI_PATH=/usr/local/bin/claude
```

**2. OAuth token expired**
```bash
# Refresh token
claude setup-token
```

**3. CLI hangs**
- Check for missing `--verbose` flag on stream-json
- Verify stdin is properly closed
- Add timeout to cmd.Run()

---

## Configuration

### Command-Line Flags

```bash
./build/ccbridge --port 9000 --host 127.0.0.1
```

| Flag | Default | Description |
|------|---------|-------------|
| `--port` | `8321` | Server port |
| `--host` | `0.0.0.0` | Bind address |

### Environment Variables

Environment variables are used as fallbacks when flags are not provided.

```bash
# Server
export CC_BRIDGE_PORT=8321
export CC_BRIDGE_HOST=0.0.0.0

# Claude CLI
export CLAUDE_CLI_PATH=claude  # or /usr/local/bin/claude
export CLAUDE_CLI_TIMEOUT=1h  # optional, no timeout by default

# Debugging
export CC_BRIDGE_DEBUG=true
```

---

## Deployment

### Local Development

```bash
make build
./build/ccbridge --port 8321
```

### Docker

```dockerfile
# Dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o build/ccbridge ./src

FROM node:20-alpine
# Install Claude CLI
RUN npm install -g claude-code

COPY --from=builder /app/build/ccbridge /usr/local/bin/

EXPOSE 8321
CMD ["ccbridge"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  cc-bridge:
    build: .
    ports:
      - "8321:8321"
    environment:
      - CC_BRIDGE_PORT=8321
      - CC_BRIDGE_DEBUG=false
    volumes:
      - ~/.claude:/root/.claude  # Claude config
```

---

## Performance Optimization

### Process Pooling (Future)

```go
// Keep Claude sessions alive and reuse
type CLIPool struct {
    sessions chan *exec.Cmd
}

func (p *CLIPool) Get() *exec.Cmd {
    select {
    case cmd := <-p.sessions:
        return cmd
    default:
        return exec.Command("claude", "-p", "--session-id", uuid.New().String())
    }
}
```

### Request Batching (Future)

```go
// Batch multiple requests to single CLI invocation
type RequestBatch struct {
    requests []AnthropicRequest
    results  []chan AnthropicResponse
}
```

---

## Production Checklist

- [ ] Graceful shutdown (SIGTERM handling)
- [ ] Request timeouts (prevent hanging)
- [ ] Resource limits (max concurrent requests)
- [ ] Health checks (/health, /ready)
- [ ] Metrics (request count, latency, errors)
- [ ] Structured logging (JSON logs)
- [ ] Error recovery (auto-restart on crash)
- [ ] Docker image built and tested
- [ ] README with setup instructions
- [ ] Integration tests passing

---

## Known Limitations

1. **Tool mapping incomplete** - Only basic tools supported initially
2. **No session persistence** - Each request is independent
3. **Single-user** - No multi-tenant support yet
4. **No caching** - Every request spawns new process
5. **Limited error details** - CLI errors may be opaque

---

## Future Enhancements

### v1.1: Performance
- Process pooling (reuse sessions)
- Response caching
- Connection pooling

### v1.2: Features
- Full tool schema support
- Session persistence
- Request queuing

### v1.3: Multi-User
- API key auth
- Per-user rate limits
- Usage tracking

---

## Resources

### Anthropic API
- Messages API: https://docs.anthropic.com/en/api/messages
- Streaming: https://docs.anthropic.com/en/api/messages-streaming
- SDK: https://github.com/anthropics/anthropic-sdk-typescript

### Claude CLI
- Documentation: https://code.claude.com/docs
- Print mode guide: https://x.com/dhasandev/status/2009529865511555506
- Experiments: https://gist.github.com/danialhasan/abbf1d7e721475717e5d07cee3244509

### Go Resources
- HTTP Server: https://pkg.go.dev/net/http
- Process Execution: https://pkg.go.dev/os/exec
- Testing: https://pkg.go.dev/testing

---

**Ready to build!** 🚀

---
> Source: [ranaroussi/cc-bridge](https://github.com/ranaroussi/cc-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
