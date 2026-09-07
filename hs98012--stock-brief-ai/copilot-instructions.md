## stock-brief-ai

> - 이 저장소는 다른 프로젝트와 연동하지 않는 독립형 금융 문서 RAG 포트폴리오다.

# AGENTS.md

## 프로젝트 원칙

- 이 저장소는 다른 프로젝트와 연동하지 않는 독립형 금융 문서 RAG 포트폴리오다.
- 사용자에게 보이는 핵심 주장은 evidence 없이 생성하거나 노출하지 않는다.
- 근거가 없으면 정확히 `확인할 수 없습니다`라고 표현한다.
- 가짜 PDF, 임의 재무 수치, 하드코딩된 호재·악재를 만들지 않는다.
- 전망과 사실, 과거와 현재, 추정치와 실제 실적을 구분하고 원문의 단위를 보존한다.
- 투자 권유, 매수·매도 추천, 자동매매를 구현하지 않는다.

## 개발 규칙

- 비밀값과 API 키는 로컬 `.env`에만 두며 커밋하지 않는다. `.env.example`에는 안전한 예시 또는 빈 값만 둔다.
- frontend는 Next.js + TypeScript + Tailwind CSS, backend는 FastAPI, DB는 PostgreSQL 16 + pgvector를 사용한다.
- 임베딩은 맥 호스트 Ollama `bge-m3`를 유지한다. 근거 기반 생성은 설정에 따라 Gemini Flash 또는 Ollama `gemma3:4b`를 사용하며 API 키는 루트 `.env`에만 둔다. BM25, pgvector cosine 검색과 RRF를 유지하며 채팅 UI와 투자 판단은 추가하지 않는다.
- 변경 전 기존 파일과 `git status`를 확인하고 사용자 변경을 보존한다.
- API나 데이터 모델 변경 시 `api-contract.md`, `acceptance.md`, `qa-rules.md`를 함께 갱신한다.
- 완료 전 `./scripts/check.sh`를 실행한다.

## 데이터 모델 계약

현재 migration은 Ollama `bge-m3` 기준 1024차원 embedding을 사용한다. 차원 변경 시 비어 있지 않은 기존 벡터를 자동 삭제·변환하지 않는다.

### documents

| 필드 | 권장 타입 | 규칙 |
|---|---|---|
| id | uuid | PK |
| company_name | text | not null |
| stock_code | varchar(12) | 6자리 숫자 또는 null |
| document_type | enum | `broker_report`, `dart_filing` |
| issuer | text | 증권사 또는 공시기관, not null |
| published_at | date | not null |
| filename | text | 원본 파일명, not null |
| file_hash | char(64) | SHA-256, unique, not null |

### pages

| 필드 | 권장 타입 | 규칙 |
|---|---|---|
| id | uuid | PK |
| document_id | uuid | documents FK, cascade delete |
| page_number | integer | 1 이상 |
| native_text | text | nullable |
| ocr_text | text | nullable |
| final_text | text | not null |
| text_source | enum | `native`, `ocr`, `none` |
| page_image_path | text | nullable |
| ocr_error | text | nullable, 페이지 단위 OCR 실패 기록 |

`(document_id, page_number)`는 unique다.

### chunks

| 필드 | 권장 타입 | 규칙 |
|---|---|---|
| id | uuid | PK |
| document_id | uuid | documents FK, cascade delete |
| page_id | uuid | pages FK, cascade delete |
| page_number | integer | 해당 document의 pages 참조 |
| section_title | text | nullable |
| content | text | not null |
| char_start | integer | final_text 기준 시작 위치 |
| char_end | integer | final_text 기준 끝 위치(exclusive) |
| embedding | vector(1024) | nullable, 생성 결과는 Git에 저장하지 않음 |
| embedding_provider | text | 기본 `ollama` |
| embedding_model | text | 청크를 생성한 모델 |
| embedding_dimensions | integer | 기본 1024 |
| embedding_error | text | 청크 단위 실패 사유 |

### evidence

| 필드 | 권장 타입 | 규칙 |
|---|---|---|
| id | uuid | PK |
| answer_item_id | uuid | 답변 항목 FK, not null |
| chunk_id | uuid | chunks FK, not null |
| quote | text | 실제 인용 문장, not null |
| page_number | integer | 1 이상, chunk 페이지와 일치 |

답변 항목과 chunk는 evidence를 통한 다대다 관계다. 서비스 계층과 DB 제약/검증 작업은 핵심 답변이 최소 1개의 유효한 evidence를 갖기 전 `published` 상태가 되지 못하게 해야 한다.

현재 4차 분석은 `analysis_runs`에 처리 상태, provider/model, 생성 시각, 실패 사유와 검증된 결과 JSON을 기록한다. 모델이 선택한 chunk ID는 검색 후보에 포함된 경우에만 citation으로 복원하며 원문 quote를 모델이 작성하게 하지 않는다.

---
> Source: [hs98012/stock-brief-ai](https://github.com/hs98012/stock-brief-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
