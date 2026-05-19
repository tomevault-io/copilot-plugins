## stock-analysis-mvp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 구조

- `backend/` — SEC 보고서를 다운로드하고 재무제표를 추출하며 실적 발표 캘린더를 노출하는 FastAPI 서비스.
- `frontend/` — Next.js 16 앱 (TypeScript + Tailwind, 정적 `output: "export"` 모드).
- `main.py`, `downloader.py` — 저장소 루트에 있는 레거시/단독 스크립트. 실제 운영 서비스의 정본은 `backend/`.
- `requirements.txt` (루트) 와 `backend/requirements.txt` — 백엔드 Dockerfile은 `backend/requirements.txt` 만 설치함.

## 실행 명령

### 백엔드 (저장소 루트에서 실행)

- 의존성 설치: `pip install -r backend/requirements.txt`
- 개발 서버: `uvicorn backend.api:app --reload --port 8000`
  (`from backend.analyzer import ...` 와 같은 패키지 경로 import 때문에 `python backend/api.py` 로는 실행되지 않음.)
- 테스트: `pytest backend/tests` — `conftest.py` 가 `sys.path` 에 저장소 루트를 주입하여 `backend.*` import 를 해결함.
- 단일 테스트: `pytest backend/tests/test_analyzer.py::test_schema_shape -v`
- 실 엔드포인트 검증 (127.0.0.1:8000 으로 uvicorn 띄우고 SEC 포함 실 호출): `python backend/scripts/verify_contract.py`
- 도커: `docker build -f backend/Dockerfile -t stock-analyzer-backend .` 후 `-p 8000:8000` 로 실행.

### 프론트엔드 (`frontend/` 디렉토리에서 실행)

- 설치: `npm install`
- 개발 서버: `npm run dev`
- 빌드 (정적 결과물 `frontend/out/` 출력): `npm run build`
- 린트: `npm run lint`
- 타입 체크: `npx tsc -p tsconfig.json` (npm script 미연결)
- 프론트엔드는 `NEXT_PUBLIC_API_BASE_URL` 가 없으면 백엔드를 `http://localhost:8000` 으로 가정함.

## 환경 변수

백엔드:
- `ALLOWED_ORIGINS` — 콤마 구분 CORS 허용 목록. 기본값 `http://localhost:3000,http://127.0.0.1:3000`.
- `EARNINGS_CACHE_TTL_S` (기본 21600), `ANALYZE_CACHE_TTL_S` (기본 43200), `EARNINGS_EMPTY_TTL_S` / `EARNINGS_ERROR_TTL_S` (기본 300) — 인-프로세스 캐시 TTL.
- `SEC_RPS` / `MARKETBEAT_RPS` / `NASDAQ_RPS` — `pyrate-limiter` 기반 소스별 레이트 리밋.
- `SEC_CALL_TIMEOUT_S` (기본 90), `SEC_MAX_FILINGS` (기본 5).
- `DATABASE_URL` / `POSTGRES_DSN` / `POSTGRES_URL` — `section_history` 용 선택적 Postgres 연결. 미설정 시 동작 안 함 (`postgres_store.py` 가 연결 오류를 무시함).
- `MONGODB_URI` / `MONGODB_DB` / `MONGODB_COLLECTION` — 레거시 `main.py` 저장 경로에서만 사용.

프론트엔드: `NEXT_PUBLIC_API_BASE_URL`.

## 아키텍처

### 백엔드 분석 파이프라인 (`GET /analyze?ticker=&form=`)

`backend/api.py` 는 얇은 FastAPI 레이어로, 입력 검증 (`_normalize_ticker` 는 1–10자 대문자, `_normalize_form` 은 `10-Q | 10-K | 6-K | 8-K | 20-F` 만 허용) 후 인-프로세스 `TTLCache` 를 확인하고, `backend.analyzer.run_analysis` 에 위임한 뒤 `backend/models.py` 의 Pydantic 모델로 직렬화함.

`run_analysis` (`backend/analyzer.py`) 가 핵심 파이프라인이며 다음 순서로 동작함:

1. **다운로드.** `_get_best_filing_html` 가 `clients.sec_get_filing_html` 를 호출. `6-K` / `8-K` 의 경우 최신 공시가 비재무 문서(거버넌스 통지 등)인 경우가 많아, 최근 N건의 공시를 순회하며 `clients.sec_get_exhibit_urls` 가 파일명 키워드/사이즈 점수로 첨부 URL 을 랭킹함.
2. **테이블 선정.** `extract_raw_tables` 는 헤딩 텍스트에 의존하지 않음. 모든 `<table>` 이 손익/대차/현금흐름 후보로 각각 `_score_income`, `_score_balance`, `_score_cashflow` 로 점수화됨. 각 채점기는 공통 베이스 점수(`_score_base`: 목차/링크 위주 네비게이션 페널티, 숫자 밀집도 + `ix:nonfraction` 보너스) 위에 재무제표별 키워드/항목 보너스를 더함. 표는 재무제표별 최소점(`INCOME_MIN_SCORE=5`, `BALANCE_MIN_SCORE=4`, `CASHFLOW_MIN_SCORE=4`) 을 넘겨야 채택됨. 파싱을 손볼 때는 임의의 헤딩 검사를 추가하지 말고 점수 가중을 조정할 것.
3. **배지 주석.** `annotate_income_html` 이 손익계산서 "현재 기간" 셀에 `<span class="delta-badge delta-up|delta-down|delta-flat|delta-na">` 를 삽입함. Python 주석기는 4-숫자-셀 행 레이아웃(3M 현재/직전, 9M 현재/직전) 만 처리. 그 외 레이아웃 (연 단위만, 2년 초과 분기, "Revenues + Other income" 2열 그루핑 등) 은 `frontend/lib/filingTables.ts::annotateTableHTML` 가 클라이언트 측에서 다시 주석을 붙임. 이 함수는 연도 헤더 행을 보고 오름/내림차순을 판단해 현재/이전 컬럼 페어링을 결정함. 양쪽이 같은 CSS 클래스 계약을 따르므로 변경 시 동기화 필수.
4. **메트릭.** `extract_metrics` 는 각 표에서 키워드 매칭으로 행을 찾아 `_row_numeric_values` 로 숫자를 뽑고, `_infer_year_order` + `_maybe_reverse_by_year_order` 로 컬럼 방향과 무관하게 `MetricValue.current` 가 항상 최신 기간이 되도록 정렬함. `free_cash_flow` 는 `cfo + capex` 로 계산 (현금흐름표의 capex 는 이미 음수).
5. **`no_financial_data` 단락 처리.** `_has_financial_data` 는 캐시 저장 직전과 캐시 히트 직후 모두에서 실행됨. 다운로드는 성공했으나 표/메트릭이 모두 비어있으면 `code=no_financial_data` 422 를 반환하며, 해당 응답을 정상 캐시로 저장하지 말 것.
6. **`last_updated` 는 "오늘만".** `_last_updated_if_today` 는 어제 날짜로 캐시된 페이로드의 해당 필드를 제거하므로, UI 가 캐시-스테일 타임스탬프를 노출하지 않음.

### 실적 발표 파이프라인 (`GET /earnings`)

`clients.get_weekly_earnings` 는 Nasdaq (`api.nasdaq.com/api/calendar/earnings`) 을 1순위로, Marketbeat HTML 스크랩을 폴백으로 시도. Nasdaq 결과의 상태(`reported` vs `upcoming`) 는 `_nasdaq_status` 가 ET 시각 임계로 판정함 (장전 11:00 ET, 시간외 18:00 ET). 각 행에는 SEC 8-K 검색 URL (`sec_company_filings_url` → `https://www.sec.gov/files/company_tickers.json` CIK 매핑, 24시간 캐시) 과 Seeking Alpha 트랜스크립트 URL 을 부착함.

`/earnings` 엔드포인트는 `_earnings_last_success` 를 TTL 캐시 외부에 보관하여, 일시적 업스트림 실패나 빈 응답 시 `[]` 대신 최근 성공 페이로드를 반환함 (`EARNINGS_ERROR_TTL_S` 의 짧은 재시도 윈도우 사용).

### 레이트 리밋과 재시도

모든 외부 HTTP 호출은 `clients._retry` + 소스별 `pyrate_limiter.Limiter` 를 거침. 상태코드 429 인 `HttpStatusError` 는 지수 백오프로 재시도되며, 그 외 예외도 호출 지점별로 재시도 가능. SEC 다운로드는 `ThreadPoolExecutor` + `concurrent.futures.TimeoutError` 로 감싸서 리미터 자체가 `SEC_CALL_TIMEOUT_S` 를 넘는 데드락을 피함.

### 캐싱

두 계층:
- `backend/cache.py::TTLCache` — 프로세스 로컬, 스레드 안전. `analyze`, `earnings`, SEC 티커→CIK 맵, SEC accession 인덱스에 사용.
- `backend/postgres_store.py::section_history` — MD&A / 위험요인 공시간 차분을 위한 선택적 테이블. 현재 `extract_sections` 가 모두 `None` 을 반환하므로 이 경로는 휴면 상태. 섹션 추출을 다시 켤 경우 `save_section_history` 로 기록하고 다음 호출에서 읽어 사용할 것.

### 프론트엔드

`frontend/app/page.tsx` 는 단일 클라이언트 대시보드로 `/analyze` 와 `/earnings` 를 호출하고, 실적 목록을 페이지네이션 (`EARNINGS_PAGE_SIZE = 8`) 하며, 세 개의 표 HTML 블롭을 `DOMPurify` 와 `annotateTableHTML` 을 거쳐 렌더링함. `frontend/lib/apiTypes.ts` 는 `backend/models.py` 의 미러이므로 API 변경 시 함께 갱신할 것. UI 프리미티브는 `frontend/components/ui/` (shadcn 스타일) 에 있으며 Tailwind 클래스 머지는 `frontend/lib/utils.ts` 의 `cn()` 사용.

`next.config.ts` 가 `output: "export"` 를 설정하므로 요청 시점에 Node 런타임이 필요한 기능 (server actions, route handlers, `next/image` 최적화 등) 은 빌드를 깨뜨림. 동적 로직은 모두 FastAPI 백엔드에 둘 것.

## 코드 컨벤션

- Python: 4-스페이스 들여쓰기, snake_case, `UPPER_SNAKE_CASE` 상수. 타입 힌트 권장(강제 아님). 린터/포매터 미설정. 로깅은 `print(...)` 사용 (별도 logger 없음). API 에러는 `ApiError` 형태(`code`, `message`, 선택적 `details`) 의 `JSONResponse` 로 반환.
- TypeScript: `strict: true`, 더블 쿼트, `@/*` 절대 경로 import. 새로운 `any` 도입 금지. 작은 enum 은 `FilingForm` 같은 union 타입을 선호.
- `AGENTS.md` 는 gitignore 에 포함되어 신규 체크아웃에는 없을 수 있음. 그 내용에 의존하지 말 것.

## 테스트

`backend/tests/fixtures/` 의 픽스처는 네트워크 호출 없이 표 점수화와 연도 순서 로직을 검증하기 위해 손으로 만든 최소한의 SEC HTML 스니펫(`mini_filing.html`, `toc_then_income.html`, `balance_sheet_year_order.html` 등) 임. 점수 임계나 새로운 표 형태 엣지 케이스를 다룰 때는 BeautifulSoup 내부를 모킹하지 말고 픽스처를 추가할 것. `test_analyzer.py` 는 `backend.analyzer.sec_get_filing_html` 을 패치해 픽스처 HTML 을 반환하도록 함.

---
> Source: [korea7030/stock_analysis_mvp](https://github.com/korea7030/stock_analysis_mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
