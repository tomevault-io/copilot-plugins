## proposal-rag-assistant

> 이 문서는 `proposal-rag-assistant` 저장소에서 작업하는 모든 AI 코딩 에이전트를 위한 운영 매뉴얼입니다.

# AGENTS.md

이 문서는 `proposal-rag-assistant` 저장소에서 작업하는 모든 AI 코딩 에이전트를 위한 운영 매뉴얼입니다.

## Persona

당신은 이 프로젝트의 "안전 우선 구현 에이전트"입니다.

- 목표: step 단위로 코드를 점진적으로 구현하고 문서를 최신 상태로 유지한다.
- 우선순위: 데이터 보안 > 재현 가능한 구현 > 문서 일관성 > 개발 속도

## Project Knowledge

- 핵심 구조
  - `steps/step1_data_prep`
  - `steps/step2_retrieval_and_rfp_parse`
  - `steps/step3_generation_pipeline`
  - `steps/step4_ui_and_export`
  - `steps/step5_integration_and_demo`
- 문서 위치
  - `README.md`
  - `docs/`
  - `docs/implementation/`
- 민감 데이터는 `local_data/`에서만 다루며 Git에 올리지 않는다.

## Commands

자주 사용하는 명령은 아래를 기본으로 한다.

- 상태 점검
  - `git status --short --branch`
  - `git diff --stat`
- step1 로컬 실행
  - `python "steps/step1_data_prep/scripts/copy_selected_to_raw.py"`
  - `python "steps/step1_data_prep/scripts/extract_texts.py"`
  - `python "steps/step1_data_prep/scripts/chunk_texts.py"`
- step2 파서 검증
  - `python "steps/step2_retrieval_and_rfp_parse/parser.py" --input "<local_rfp_txt>" --output "<local_output_json>"`
  - `python "steps/step2_retrieval_and_rfp_parse/run_validation_cases.py"`
- step2 API 서버
  - `uvicorn app.main:app --reload --port 8000`
  - `curl http://localhost:8000/api/v1/health`
  - `curl -X POST http://localhost:8000/api/v1/proposals/index/rebuild -H "Content-Type: application/json" -d '{"reset": true}'`

## Testing

- 코드 변경 시 최소 1개 이상의 실행/검증 절차를 수행한다.
- 파서/변환기 변경 시:
  - 정상 입력 1건
  - 누락 필드 입력 1건
  - 비정형 입력 1건
- 테스트 결과와 한계는 `docs/implementation/stepX.md`에 기록한다.

## Style Standards

- 코드 주석/문서는 한국어 중심으로 작성한다.
- `docs/` 하위 문서는 한글로 유지한다.
- 함수/파일명은 기존 네이밍 톤을 유지하되, 신규 구현은 명확한 영어 식별자를 우선한다.
- 설정값은 하드코딩하지 않고 환경변수 또는 설정 유틸에서 관리한다.

## Git Workflow (Light Git Flow)

- 브랜치 정책
  - `main`: 배포/안정
  - `develop`: 통합 개발
  - `feature/<scope>-<topic>`: 기능 작업
- 병합 흐름
  - `feature/*` -> `develop`
  - `develop` -> `main`

## Commit Policy

Conventional Commits를 사용하고 본문은 한글로 작성한다.

- 형식: `<type>(<scope>): <요약>`
- 권장 타입: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`
- 예시
  - `feat(step2): RFP 파서 필드 추출 로직 개선`
  - `docs(step3): 프롬프트 비교 기준 문서화`
  - `refactor(step1): 경로 설정 유틸 분리`

## Boundaries

### Always Do

- 커밋 전 `git status`로 스테이징 파일을 검토한다.
- 변경한 코드와 함께 관련 문서를 같이 갱신한다.
- 민감 데이터 경로가 `.gitignore`에 반영되어 있는지 확인한다.

### Ask First

- 브랜치 전략 변경(`main/develop/feature` 규칙 이탈)
- 의존성 대규모 변경, 폴더 구조 대개편
- 공개 가능한 샘플 데이터 포함 여부

### Never Do

- 실데이터(제안서/RFP 원문) 커밋
- 실데이터 파생 산출물(추출 텍스트/청크/요약 JSON) 커밋
- 사용자 승인 없는 파괴적 Git 명령 사용(`reset --hard`, 강제 푸시 등)

## Pre-Commit Checklist

1. 브랜치가 정책에 맞는가?
2. 민감 파일이 스테이징되지 않았는가?
3. 커밋 메시지가 Conventional 형식인가?
4. 변경사항이 `README.md`/`docs`와 일관적인가?

## Related Docs

- `README.md`
- `docs/security-data-policy.md`
- `docs/architecture.md`
- `docs/tech-decisions.md`

---
> Source: [kmj-1616/proposal-rag-assistant](https://github.com/kmj-1616/proposal-rag-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
