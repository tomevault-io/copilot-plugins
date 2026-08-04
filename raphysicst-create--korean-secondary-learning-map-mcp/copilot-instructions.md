## korean-secondary-learning-map-mcp

> 한국 중등 2022 개정 교육과정 학습 그래프 MCP 서버 (npm: `korean-secondary-learning-map-mcp`, 레지스트리: `io.github.raphysicst-create/korean-secondary-learning-map-mcp`). 성취기준 2,887(중 714 + 고 보통교과 2,173, 공식 원문 수록) · 주제 4,333 · 선수관계 213 · 클러스터 632 · 중→고 전이 175 · 과목관계 37을 패키지에 내장, stdio 완전 로컬 동작. 도구 11종.

# CLAUDE.md

한국 중등 2022 개정 교육과정 학습 그래프 MCP 서버 (npm: `korean-secondary-learning-map-mcp`, 레지스트리: `io.github.raphysicst-create/korean-secondary-learning-map-mcp`). 성취기준 2,887(중 714 + 고 보통교과 2,173, 공식 원문 수록) · 주제 4,333 · 선수관계 213 · 클러스터 632 · 중→고 전이 175 · 과목관계 37을 패키지에 내장, stdio 완전 로컬 동작. 도구 11종.

## 명령

- `npm test` — 테스트 58개 (`node --test tests/*.test.mjs`)
- `npm run pipeline:fetch` — DECK6 원본 16개 파일(middle 7 + high 8 + bridges 1) + NCIC PDF 4권을 확보하고 `pipeline/sources.json`의 SHA-256과 대조 (`.cache/`, git 미추적). PDF는 `PDF_DIR` 환경 변수(기본: 저장소 부모 폴더)에서 찾음
- `npm run pipeline:build` — `.cache/`의 DECK6 원본에서 중학교 전체 + 고교 보통교과만 범위 필터링해 `data/kr/*.json` 생성 (특성화고 전문교과·특목고 예술/과학/체육 계열 제외, 「연극」만 예외 포함)
- `npm run pipeline:extract` — PDF 4권에서 `pdftotext -layout`으로 성취기준 원문 절취. 실패분은 `pipeline/exceptions.json`(수식 포함 12건)으로 수동 보정
- `npm run pipeline:verify` — 성취기준 수 게이트(중 714 + 고 2,173) + 참조 무결성 전수 검증 + `manifest.json` 해시 기록 (**데이터 변경 후 필수**)
- `node src/cli.mjs` — 서버 직접 실행 (initialize JSON-RPC 파이프로 스모크 가능)

파이프라인은 반드시 `fetch → build → extract → verify` 순서로 실행한다. `data/kr/*.json`은 커밋되어 있으므로 일반 개발에는 재현이 필요 없다.

## 규칙

- 순수 ESM `.mjs`만, TypeScript·빌드 금지. 런타임 의존성은 `@modelcontextprotocol/sdk` + `zod` **2종만**.
- stdout은 MCP 프로토콜 전용 — 사람용 로그는 반드시 `console.error`.
- 도구 이름 영어 snake_case, 설명·에러 메시지 한국어. 검색 limit 기본 20 최대 50. NFC 정규화 + 부분 문자열 일치만 (형태소 분석기 금지).
- **원문 수록 범위는 성취기준 본문 문장만** — 해설·적용 시 고려사항 추가 금지. 법적 근거·출처 표기는 NOTICE.md.
- 아키텍처: data-store(로드·인덱스·SHA-256 재검증) → search/graph/roadmap/transitions(순수 함수) → server(도구 정의) → cli(stdio 진입점). 역방향 의존 금지.

## 데이터 특성 (가정 금지)

- 성취기준 수는 **중 714 + 고 보통교과 2,173 = 2,887 고정 게이트**다. `pipeline:build`·`pipeline:verify` 모두 이 수치와 다르면 실패(throw/exit 1)한다 — 늘거나 줄면 범위 필터 로직을 먼저 의심할 것.
- 과목명·코드에 로마숫자(Ⅰ·Ⅱ·Ⅲ 등)가 섞여 있고 표기가 일관되지 않다("영어 I" vs "미적분Ⅰ"). `src/normalize.mjs`의 `normalizeRoman`/`normalizeCode`/`normalizeText`를 거치지 않은 원시 문자열 비교 금지.
- 11개 성취기준 코드는 두 과목이 공유한다(`standardsByCodeAll`은 코드당 배열). `get_standard`는 `subject`로만 소거 가능 — 코드 단독으로는 유일하게 정해지지 않을 수 있다.
- topic ID·cluster ID는 DECK6 원본 ID를 그대로 쓴다(재채번 금지) — `deck6CourseId`로만 과목을 역참조한다.
- PDF 원본 4권(별책3, 별책4 Ⅰ·Ⅱ·Ⅲ)은 저장소 밖 `PDF_DIR`(기본: 저장소 부모 폴더)에 있어야 하며 저장소에 커밋하지 않는다.
- 모든 데이터 파일은 `data/kr/manifest.json`의 SHA-256으로 서버 기동 시 재검증된다 — 데이터 수정 후 `pipeline:verify` 없이는 서버가 기동 거부.

## 배포 절차 (버전 릴리스)

1. `package.json` version + `server.json` version **2곳**(서버·packages) 동시 bump
2. `npm test && npm run pipeline:verify`
3. `npm publish --browser=false`
4. `mcp-publisher publish` — MCP 공식 레지스트리 반영 (인증 만료 시 `mcp-publisher login github`)

## 관련 저장소

- 기반 데이터셋(학습 주제·선수관계·클러스터·전이·과목관계, MIT): https://github.com/DECK6/korean-secondary-learning-map — 고정 커밋은 `pipeline/sources.json` 참조.

---
> Source: [raphysicst-create/korean-secondary-learning-map-mcp](https://github.com/raphysicst-create/korean-secondary-learning-map-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
