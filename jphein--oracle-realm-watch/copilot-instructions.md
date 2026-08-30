## oracle-realm-watch

> Voice-first AI oracle with fantasy UI. Go backend, embedded static assets.

# Oracle Sanctum

Voice-first AI oracle with fantasy UI. Go backend, embedded static assets.

## Build

go build -o oracle-server .           # local
GOOS=linux GOARCH=amd64 go build -o oracle-server .  # cross-compile

## Run

cp config.yaml.example config.yaml
# Edit config.yaml with your settings
./oracle-server -config config.yaml

## Deploy

make deploy  # cross-compile + scp + restart service

## Project Structure

- main.go -- entry point, HTTP routes, static file serving
- config.go -- config struct, YAML load/save, env override
- chat.go -- /api/chat handler, SSE streaming, provider routing
- speech.go -- /api/stt and /api/tts handlers (Azure Speech REST)
- providers/ -- AI provider implementations (bedrock, openai, anthropic)
- static/ -- embedded frontend (HTML/CSS/JS)

## API

| Route | Method | Purpose |
|-------|--------|---------|
| / | GET | Embedded static files |
| /api/chat | POST | AI chat with SSE streaming |
| /api/stt | POST | Speech-to-text (audio body) |
| /api/tts | POST | Text-to-speech (returns audio) |
| /api/settings | GET/POST | Read/write config |
| /api/providers | GET | List available providers/models |
| /api/health | GET | Health check |

---
> Source: [jphein/oracle.realm.watch](https://github.com/jphein/oracle.realm.watch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
