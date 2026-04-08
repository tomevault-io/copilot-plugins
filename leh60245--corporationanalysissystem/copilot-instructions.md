## corporationanalysissystem

> **목적:** AI 협업 히스토리 관리, 오류 추적, 해결 방안 아카이빙 (개발자 및 AI의 "기억 저장소")


### 📂 2. CLAUDE.md
**목적:** AI 협업 히스토리 관리, 오류 추적, 해결 방안 아카이빙 (개발자 및 AI의 "기억 저장소")

# 🤖 CLAUDE.md (Developer Journal & Error Log)

이 파일은 프로젝트 진행 중 발생하는 이슈, 해결책, 주요 의사결정 사항을 기록하여 컨텍스트를 유지하는 데 사용됩니다.

## 📌 현재 진행 상황 (Current Context)
* **단계:** 시스템 고도화 및 소스 확장 (Phase 2)
* **현재 작업:** AI 생성 리포트 저장소 구축 완료 (Task 011)
* **최근 업데이트:** 
  - Generated_Reports 테이블 생성 및 데이터 삽입 인터페이스 구현
  - 효율적인 DART 보고서 검색 방식 구현 (Task 010)

---

## 🛠️ 주요 명령어 (Commands)
```bash
# 가상환경 활성화
conda activate CorporationAnalysis

# 테스트 실행
pytest tests/

# 특정 테스트 파일 실행
pytest tests/test_news_agent.py

# 메인 파이프라인 실행
python main.py

```

---

## 🐛 오류 및 해결 로그 (Issue & Solution Log)

### [2026-01-09] Task 011: Generated_Reports 테이블 구축

* **작업:** AI 생성 리포트 저장소 구축
* **구현 내용:**
  - `Generated_Reports` 테이블 생성 (10개 컬럼, JSONB 활용)
  - `insert_generated_report()` 메서드 구현
  - DB 통계에 `generated_reports` 항목 추가
  - 테스트 스크립트 작성 및 검증 완료
* **관련 파일:** 
  - `src/core/db_manager.py`
  - `scripts/test_generated_reports.py`
  - `docs/TASK_011_GENERATED_REPORTS.md`
* **검증 결과:** ✅ 모든 테스트 통과

### [2026-01-09] Task 010: 효율적인 DART 보고서 검색

* **이슈:** 기존 방식은 전체 상장사(~2,600개)를 순회하며 개별 API 호출 (비효율)
* **해결:** 
  - `dart.filings.search()` 기반 기간별 일괄 검색 방식 구현
  - `search_all_reports()` 메서드 신규 추가
  - `get_corps_with_reports()` 메서드 리팩토링
  - `run_efficient()` 파이프라인 메서드 추가
  - Report 객체 속성 접근 방식 수정 (`.get()` → `getattr()`)
* **결과:** API 호출 횟수 90% 이상 감소, 실행 시간 대폭 단축
* **관련 파일:** 
  - `src/core/dart_agent.py`
  - `src/core/pipeline.py`
  - `config.py`
  - `main.py`

### [Template]

* **날짜:** YYYY-MM-DD
* **이슈:** (발생한 에러 메시지 또는 버그 현상 요약)
* **원인:** (분석된 원인)
* **해결:** (적용한 코드 수정 또는 해결 방법)
* **관련 파일:** `src/core/xxx.py`

### 202X-XX-XX (예시)

* **이슈:** `psycopg2.errors.UndefinedTable: relation "analysis_reports" does not exist`
* **원인:** DB 초기화 스크립트가 실행되지 않아 테이블이 생성되지 않음.
* **해결:** `db_manager.py`의 `create_tables()` 메서드를 `main.py` 시작 시 호출하도록 수정함.
* **관련 파일:** `src/core/db_manager.py`

---

## 💡 아키텍처 결정 사항 (ADR Summary)

* **[ADR-001] 데이터 소스 확장 전략:** `rcept_no`에 의존하던 PK 전략을 수정하여, 뉴스 및 웹 데이터는 URL 해시 등을 ID로 사용하고 `source_type` 컬럼으로 구분하기로 결정함.
* **[ADR-002] 임베딩 전략:** 테이블 데이터의 문맥 손실을 막기 위해 테이블 바로 앞의 텍스트 블록을 합쳐서 임베딩하는 'Context Look-back' 방식 유지.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leh60245)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/leh60245)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
