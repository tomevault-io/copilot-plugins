## fastapi-sample

> Claude Code가 이 프로젝트에서 작업할 때 참고하는 프로젝트 가이드.

# CLAUDE.md

Claude Code가 이 프로젝트에서 작업할 때 참고하는 프로젝트 가이드.

## 작업 시작 체크

1. `rg --files` 등으로 실제 파일 구조를 먼저 확인한다.
2. `app/`, `tests/`, `pyproject.toml`, `docker-compose.yml` 등이 문서와 다르면 현재 파일 구조를 우선하고, 차이를 사용자에게 짧게 알린다.
3. 요청과 직접 관련된 문서(`docs/*`)와 스킬(`.claude/skills/*`)만 읽어 컨텍스트를 과하게 쓰지 않는다.
4. 기존 사용자 변경을 되돌리지 않는다. 관련 파일에 예상 밖 변경이 있으면 그 위에서 조심스럽게 작업한다.

## 프로젝트 개요

CRUD 및 엑셀/CSV 처리를 확인하기 위한 학습·데모용 FastAPI 서버.
도메인 엔티티는 **Item(상품)** 하나이며, 표준 REST CRUD와 파일 일괄 업로드를 제공한다.

## 스택

- FastAPI + uvicorn
- SQLAlchemy 2.0 (async) + MySQL (Docker), 테스트는 SQLite(aiosqlite) 인메모리
- pandas / openpyxl (엑셀·CSV)
- uv (패키지 관리), pytest (테스트), ruff (린트/포맷)

## 자주 쓰는 명령

프로젝트 파일이 존재하고 의존성이 준비된 상태에서 사용한다.

```bash
docker compose up -d db                        # MySQL 기동 (DB만)
uv sync                                        # 의존성 설치
uv run uvicorn app.main:app --reload           # 서버 실행 (http://localhost:8000/docs)
uv run pytest -v                               # 테스트 (Docker 불필요)
uv run ruff check .                            # 린트
uv run ruff format .                           # 포맷
```

## 아키텍처 (레이어드)

```
app/
├── main.py            # 앱 생성, lifespan(테이블 생성), 라우터 등록
├── core/
│   ├── config.py      # pydantic-settings, .env 로드
│   └── database.py    # async engine/session, Base, get_db 의존성
├── models/item.py     # SQLAlchemy ORM 모델
├── schemas/item.py    # Pydantic (ItemCreate/Update/Read)
├── crud/item.py       # DB 접근 함수
├── services/file_processor.py  # 엑셀/CSV 파싱·검증
└── api/routes/
    ├── items.py       # /items CRUD
    └── files.py       # /items/upload
```

**데이터 흐름**: `api/routes → crud/services → models/schemas`

## 코드 작성 규칙 (요약)

전체 규칙은 `docs/CODING_CONVENTIONS.md` 참고. 핵심:

- **라우터는 얇게** 유지하고 DB 로직은 `crud`, 비즈니스 로직은 `services`에 둔다.
- 라우터에서 SQLAlchemy 쿼리를 직접 작성하지 않는다.
- ORM 모델을 API에 직접 노출하지 말고 `schemas`(예: `ItemRead`)로 변환한다.
- 모든 함수에 타입 힌트, 공개 함수/모듈에 한글 docstring.
- I/O 함수는 `async def` + `AsyncSession` 사용.

## 새 도메인(엔티티) 추가 시 순서

1. `app/models/<name>.py` — ORM 모델 (`Base` 상속, `main.py`/`conftest.py`에서 import 되게 등록)
2. `app/schemas/<name>.py` — `XxxCreate/Update/Read`
3. `app/crud/<name>.py` — create/get/list/update/delete
4. `app/api/routes/<name>.py` — 엔드포인트, `main.py`에 `include_router`
5. `tests/test_<name>.py` — 테스트 추가

> `.claude/skills/add-crud` 스킬이 이 절차를 안내한다.

## 엑셀/CSV 처리 규칙

- 파싱·검증 로직은 `app/services/file_processor.py`에 둔다.
- 필수 컬럼: `name, price, quantity` / 선택: `description`.
- 유효 행은 저장, 잘못된 행은 오류 리포트로 반환(부분 성공 허용).

## Git 컨벤션 (요약)

전체 규칙은 `docs/GIT_CONVENTIONS.md` 참고.
- Conventional Commits: `feat(items): 상품 목록 페이지네이션 추가`
- 브랜치: `feature/`, `fix/`, `refactor/`, `docs/`
- 머지 전 `uv run pytest`, `uv run ruff check .` 통과 필수.

## Claude Code Skills

`.claude/skills/` 에 정의됨:
- `run-server` — 서버 실행 및 엔드포인트 동작 검증
- `add-crud` — 새 도메인 CRUD 4계층 일괄 생성
- `process-file` — 엑셀/CSV 처리 로직 추가/확장
- `run-tests` — pytest 실행 및 결과 요약

## 에이전트 문서 동기화

- `AGENTS.md`와 `CLAUDE.md`는 플랫폼명과 스킬 경로만 다르게 유지한다.
- `.agents/skills/*/SKILL.md`와 `.claude/skills/*/SKILL.md`는 같은 작업 절차를 담도록 동기화한다.
- 에이전트 문서를 바꿀 때는 관련 경로, 명령, 파일명이 실제 구조와 맞는지 함께 확인한다.

## 검증 기준

- 문서만 바꾼 경우: 변경한 Markdown을 다시 읽어 경로·명령·오타를 확인한다.
- 코드나 테스트를 바꾼 경우: 관련 테스트를 먼저 실행하고, 마무리 전 `uv run pytest -v`와 `uv run ruff check .`를 실행한다.
- 서버 동작 확인이 필요한 경우: `.claude/skills/run-server` 절차를 따른다.

## 주의사항

- 데모 편의를 위해 `lifespan`에서 테이블을 자동 생성한다. 실제 서비스라면 Alembic 마이그레이션 도입 권장.
- 모델은 MySQL/SQLite 공통 타입만 사용해 테스트 이식성을 유지한다(새 컬럼 추가 시 동일 원칙 유지).

---
> Source: [kyeonghooon/fastapi-sample](https://github.com/kyeonghooon/fastapi-sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
