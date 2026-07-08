## jobfit

> Codex가 JobFit 저장소에서 작업할 때 따라야 하는 프로젝트 로컬 지침입니다. `.agent/`를 기준(Source of Truth)으로 삼되, Codex 실행 표면은 `.codex/` 아래에 둡니다.

# JobFit Codex Agent Guide

Codex가 JobFit 저장소에서 작업할 때 따라야 하는 프로젝트 로컬 지침입니다. `.agent/`를 기준(Source of Truth)으로 삼되, Codex 실행 표면은 `.codex/` 아래에 둡니다.

## 우선순위와 동기화

1. 상위 시스템/개발자 지침이 항상 최우선입니다.
2. 이 파일은 저장소 전체에 적용됩니다.
3. 세부 규칙은 `.codex/rules/`를 함께 읽습니다.
4. 다른 AI 도구와 충돌하면 `.agent/`를 우선 기준으로 보고, `.claude/`는 Claude Code 전용 구현 예시로 참고합니다.
5. `.agent/`, `.claude/`, `.codex/` 중 하나의 규칙/스킬을 바꾸면 다른 도구 폴더의 유사 규칙 존재 여부를 확인합니다.

## 기본 커뮤니케이션

- 사용자 응답, 진행 상황, 계획, 작업 목록은 한국어로 작성합니다.
- `FastAPI`, `React`, `Zustand`, `uv` 같은 표준 기술명은 영어를 유지합니다.
- 간단한 작업은 묻지 말고 직접 수정-검증까지 진행합니다.

## 프로젝트 개요

- 목적: AI 기반 이력서 분석, 채용공고 매칭, 갭 분석, 로드맵 생성, 면접 연습.
- Backend: FastAPI, Python 3.12, Pydantic v2, SQLAlchemy, `uv`.
- Frontend: React/Vite, TypeScript, Tailwind CSS, Zustand, TanStack Query.
- AI 연동: OpenAI/Gemini 호환 SDK, NVIDIA, ElevenLabs, LangGraph 계열 에이전트.

## 필수 명령어

```bash
# Python 의존성/실행은 uv만 사용
uv sync
uv run ruff format --check server/ tests/
uv run ruff check server/ tests/
uv run pytest

# 통합 Makefile
make lint
make test

# Backend 개발 서버: 반드시 server 디렉토리에서 main:app 사용
cd server && uv run uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Frontend
npm --prefix client install
npm --prefix client run lint
npm --prefix client run build
npm --prefix client run dev
```

주의: `pip`, `poetry` 직접 사용 금지. Python 패키지 추가/삭제는 `uv add`, `uv remove`를 사용합니다.

## 검증 기준

- Python 변경: `make lint` 후 `make test`를 우선 실행합니다.
- Frontend 변경: `npm --prefix client run lint`와 `npm --prefix client run build`를 실행합니다.
- 전체/보안성 변경: 위 검증에 더해 관련 smoke test 또는 수동 API 확인을 추가합니다.
- 검증 실패 시 원인과 남은 리스크를 숨기지 말고 보고합니다.

## Git 컨벤션

프로젝트 커밋 메시지는 `.codex/rules/03-git-convention.md`와 `.codex/skills/git-convention/SKILL.md`를 따릅니다.

기본 형식:

```text
type(scope): subject (한국어 명령형, 72자 이내)

WHY: 변경 이유 한 줄
WHAT:
- 기능/동작 중심 변경사항

IMPACT: 파괴적 변경/부수효과 (해당 시에만)
Refs: #이슈번호 (선택)
```

- 커밋 전 staged diff, 보안 패턴, 원자성을 확인합니다.
- `.env`, API key, token, credential 파일은 절대 커밋하지 않습니다.
- 첫 줄은 반드시 `type(scope): subject` 프로젝트 형식을 사용합니다. 상위 Codex/OMX 런타임이 Lore trailer를 요구해도 프로젝트 WHY/WHAT 뒤에만 추가하며, trailer가 첫 줄 형식을 대체할 수 없습니다.
- 커밋 전 `.githooks/commit-msg` 검증이 활성화되어 있는지 확인합니다(`git config core.hooksPath .githooks`).

## 보안/개인정보

- LLM 또는 외부 API로 이력서/사용자 데이터를 보내기 전에 이메일, 전화번호 등 PII를 마스킹합니다.
- 로그에 원본 PII, API key, JWT secret, OAuth token을 남기지 않습니다.
- URL 스크래핑/JD 수집 코드는 SSRF 방어를 유지해야 합니다.
- **AI 에이전트 `.env` 직접 읽기 금지**: AI 에이전트는 기밀 유출 방지를 위해 실젯값이 포함된 `.env` 파일을 직접 열람하거나 가로채지 않아야 하며, 환경 변수 점검 시 오직 `config.py`와 `.env.example`만 참조합니다.
- **환경 변수 불일치 해소**: `.env.example`과 `.env` 간의 누락이나 불일치가 의심되면, AI는 직접 `.env`를 들여다보지 않고 분석한 필요한 변수 목록을 사용자에게 제시하여 사용자가 직접 수동 수정하도록 안내합니다.
- `.env`는 로컬 전용이며 `.env.example`만 문서화합니다.

## 문서와 메모리

프로젝트의 기능별 구현 현황 및 개발 진행 상태는 개별 설명 문서에 하드코딩하여 중복 관리하지 않으며, [active_context.md](file:///c:/Users/fkjy1/dev/Hackathon/jobfit/.agent/memory/active_context.md)를 단일 진입점(Source of Truth)으로 관리 및 참조합니다.

의미 있는 기능/설정 변경 후 다음을 확인합니다:

- `README.md`, `AGENTS.md`, `CLAUDE.md` 중 영향받는 문서가 최신인지 확인합니다.
- `.agent/memory/active_context.md`에는 최근 작업, 구현 상태 및 다음 단계를 갱신합니다.
- 아키텍처/명령/환경 변수 변경은 `.agent/memory/tech_spec.md` 또는 관련 docs에 반영합니다.

## 에이전트 공통 로컬 구조 (.agent)

```text
.agent/
├── memory/                     # 구현 현황 및 아키텍처 명세 (Source of Truth)
│   ├── active_context.md       # 최근 작업 로그 및 진행 상태
│   ├── tech_spec.md            # 아키텍처 및 설정 스펙
│   ├── api_schema.md           # API/WebSocket 스키마
│   └── memory_audit.md         # 메모리 표면 및 캐시 관리 상태
├── rules/                      # 에이전트 행동 세부 규칙 (대시 포맷)
│   ├── 00-multi-tool-sync.md   # 다중 툴 동기화 규칙
│   ├── 01-language-preference.md # 한국어 우선 및 기술영어 지침
│   ├── 02-python-uv.md         # Python uv 관리 규칙
│   ├── 03-git-convention.md    # 커밋 메시지 및 브랜치 규칙
│   ├── 04-security-pii.md      # PII 마스킹 및 .env 접근 금지 규칙
│   ├── 05-project-structure.md # 프로젝트 디렉토리 및 핵심 파일 맵
│   ├── 06-git-branch-strategy.md # 협업 브랜치 전략
│   └── 07-continuous-documentation.md # 지속적 문서 갱신 규칙
└── skills/                     # 에이전트 전용 스킬 라이브러리
```

에이전트가 프로젝트 분석이나 수정을 시작할 때, 최우선적으로 `.agent/memory/active_context.md` 및 `.agent/rules/` 하위 규정들을 준수해야 합니다.

## Codex 로컬 구조

```text
.codex/
├── README.md                  # Codex 구조와 사용법
├── rules/                     # 프로젝트 규칙
├── skills/                    # Codex skill 진입점
│   ├── ai-pipeline-review/
│   ├── architecture-review/
│   ├── frontend-product-review/
│   ├── git-convention/
│   ├── jobfit-development/
│   ├── security-privacy-review/
│   └── test-qa-review/
└── prompts/                   # 반복 작업용 Codex prompt 템플릿
```

Codex에서 프로젝트 규칙이 필요하면 먼저 이 파일과 `.codex/README.md`를 읽고, 세부 작업에 맞는 `.codex/rules/` 또는 `.codex/skills/` 파일을 사용합니다.

---
> Source: [fighting-team-tree/jobfit](https://github.com/fighting-team-tree/jobfit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
