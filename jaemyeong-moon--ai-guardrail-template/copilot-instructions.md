## ai-guardrail-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **template project** for managing an LLM Wiki (knowledge base) in Harness CI/CD environments. There is no application source code — the repository consists entirely of structured Markdown wiki files under `.wiki/` that LLM agents selectively load to perform tasks.

## Critical Workflow: Wiki-Driven Context Loading

**Never load all wiki files at once.** Follow this sequence:

1. Read `.wiki/INDEX.md` first (always)
2. For any change task, read `.wiki/conventions/ai-delivery-guardrails.md` before task-specific entries
3. Load only the remaining wiki entries relevant to the current task (use the keyword column in INDEX.md to decide)
4. Check each loaded entry's `related:` frontmatter field to decide if adjacent entries are needed (max 2 hops)
5. After completing work, update any wiki entries with new knowledge

See `.wiki/GUIDE.md` for scenario-based loading recommendations (e.g., which files to load for API work vs. pipeline changes vs. incident response).

## Wiki Authoring Rules

- Every `.md` file under `.wiki/` must have YAML frontmatter: `title`, `category`, `keywords`, `related`, `updated`
- Target 50–150 lines per entry; split if larger
- Start each entry with a 2–3 line summary section
- Mark incomplete sections with `TODO:`
- When adding a new entry: create the file, add it to `INDEX.md`, and update `related:` fields in linked entries
- When updating an entry: always bump the `updated:` date

## Directory Layout

```
.wiki/INDEX.md          # Master index — read this first
.wiki/GUIDE.md          # Wiki authoring rules and context-loading strategy
.wiki/harness/          # Harness platform (pipelines, triggers, connectors, delegates, templates, secrets)
.wiki/service/          # Service development (overview, API spec, data model, config, dependencies)
.wiki/architecture/     # System design, tech stack, design patterns
.wiki/operations/       # Deploy flow, monitoring, runbook
.wiki/conventions/      # Coding style, git workflow, naming, review checklist
.wiki/troubleshooting/  # Common errors, Harness-specific issues
```

## Agent Team Mode

이 프로젝트는 멀티 에이전트 팀 모드를 지원합니다. `.claude/agents/`에 정의된 전문 에이전트들이 역할별로 작업을 분담합니다.

### 에이전트 구성

| 에이전트 | 역할 | 모델 |
|----------|------|------|
| `project-manager` | 작업 분석, 태스크 분할, 에이전트 위임, 우선순위 관리 | opus |
| `developer` | 코드 작성, 버그 수정, 파이프라인 YAML, 기능 구현 | sonnet |
| `design-architect` | 시스템 설계, 기술 스택, API 설계, ADR | sonnet |
| `qa` | 코드 리뷰, 테스트, 버그 탐지, 품질/보안 검증 | sonnet |
| `ops-specialist` | 배포, 모니터링, 장애 대응, 인프라 관리 | sonnet |

### 사용 방식

- **Subagent 위임**: 단일 세션에서 전문 에이전트에게 작업 위임 (결과만 반환)
- **Agent Teams**: 여러 에이전트가 병렬 작업 + 상호 커뮤니케이션 (실험적 기능, `.claude/settings.json`에서 활성화됨)

### 에이전트 추가/수정

`.claude/agents/`에 마크다운 파일 추가. frontmatter에 `title`, `description`, `prompt`, `tools`, `model` 정의. `description`을 구체적으로 작성해야 자동 위임이 정확하게 동작합니다.

## Git Conventions

Branch naming: `feature/TICKET-123-desc`, `bugfix/TICKET-456-desc`, `hotfix/TICKET-789-desc`

Commit format:
```
type(scope): short description

Refs: TICKET-123
```
Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`

---
> Source: [jaemyeong-moon/ai-guardrail-template](https://github.com/jaemyeong-moon/ai-guardrail-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
