## ai-chat-tui

> Terminal AI chat client — works with any OpenAI-compatible API.

# AI Chat TUI

Terminal AI chat client — works with any OpenAI-compatible API.

Built with Bubble Tea, Bubbles, Lip Gloss, Glamour.

## Build & run

```bash
go mod tidy && go build -o ai-chat . && ./ai-chat
```

## Examples

| Provider | Command |
|----------|---------|
| Ollama (local) | `./ai-chat --api http://localhost:11434 --model llama3.2` |
| code-inference | `./ai-chat --api http://api:8000 --model /models/model.gguf` |
| OpenAI | `./ai-chat --api https://api.openai.com/v1 --model gpt-4o-mini --api-key sk-...` |

Flags: `--model`, `--api`, `--api-key`, `--history` (see `./ai-chat -h`).

## Keybindings

| Key | Action |
|-----|--------|
| Enter | Send |
| Shift+Enter | Newline |
| Tab | Focus toggle |
| Up/Down | Scroll |
| Ctrl+N | New session |
| Ctrl+D | Delete session |
| q / Ctrl+C | Quit |

## Files

- `main.go` — entry, flags
- `model.go` — Bubble Tea model
- `update.go` — event handling
- `view.go` — layout
- `styles.go` — Lip Gloss
- `api.go` — SSE streaming
- `render.go` — Glamour markdown
- `history.go` — session persistence

---
> Source: [jeanmachuca/ai-chat-tui](https://github.com/jeanmachuca/ai-chat-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
