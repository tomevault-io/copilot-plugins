## open-managed-agents

> Go로 구현한 Claude Managed Agents 오픈소스 클론. 프로바이더 무관, 셀프호스트, 단일 바이너리.

# open-managed-agents (OMA)

Go로 구현한 Claude Managed Agents 오픈소스 클론. 프로바이더 무관, 셀프호스트, 단일 바이너리.

## Quick Reference

```bash
make build    # bin/oma 바이너리 빌드
make dev      # go run으로 서버 실행
make test     # 전체 테스트
go build ./... # 컴파일 확인
```

## Architecture

- `cmd/oma/` — CLI 진입점 (main.go)
- `internal/api/` — Echo HTTP 서버, REST API 핸들러 (/v1/agents, /v1/environments, /v1/sessions)
- `internal/agent/` — Agent 도메인 타입
- `internal/environment/` — Environment 도메인 타입
- `internal/session/` — Session 엔진, EventBus (SSE pub/sub)
- `internal/llm/` — LLM Provider 인터페이스 + OpenAI 호환 구현
- `internal/sandbox/` — Sandbox 인터페이스 (Docker/Local)
- `internal/tools/` — 빌트인 도구 (bash, file_read/write/edit, glob, grep)
- `internal/mcp/` — MCP (Model Context Protocol) 클라이언트 통합
- `internal/store/` — Store 인터페이스 + SQLite (sqlc)
- `internal/config/` — 환경변수 → Config
- `cli/` — cobra CLI 커맨드

## Conventions

- Go 1.22+, `internal/` 패키지 구조
- REST API: Claude Managed Agents 호환 shape
- LLM: OpenAI 호환 API (`/chat/completions`)
- DB: SQLite + sqlc
- 환경변수 prefix: `OMA_`

## Design Spec

Design spec is tracked in-repo via issues and docs.

---
> Source: [domuk-k/open-managed-agents](https://github.com/domuk-k/open-managed-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
