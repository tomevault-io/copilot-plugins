## contractrisk-guard

> 이 파일은 Claude Code(claude.ai/code)가 이 저장소에서 작업할 때 참고하는 지침입니다.

# CLAUDE.md

이 파일은 Claude Code(claude.ai/code)가 이 저장소에서 작업할 때 참고하는 지침입니다.

## 언어 규칙
모든 응답, 코드 주석, 출력 메시지는 반드시 **한국어**로 작성한다.

## 프로젝트 개요
**ContractRisk-Guard**: AI Agent 기반 기업 법무 플랫폼 (계약 해지 + 책임제한 조항 리스크 예측)
- 공정위 시정조치 사례(Seed) 기반 부트스트래핑
- GPT-4o Forward 라벨링 + KoELECTRA Backward 검증으로 노이즈 자동 탐지·제거
- 전문가 없이 고품질 법률 데이터셋 자동 구축 → 분류 모델 학습 → Agent 엔진

## 분석 도메인
- **해지 조항**: 일방적 해지, 해지 통보 기간, 위약금, 해지 사유 모호성 (약관규제법§9, 민법§543~553)
- **책임제한 조항**: 전면면책, 간접손해 배제, 손해배상 상한, 고의·중과실 배제 (약관규제법§7, 민법§750~766)

## 아키텍처
```
scripts/       데이터 수집 스크립트
data/          raw/ → processed/ → labels/ → seed/
models/        v1/(Seed학습), v2/(CLEAN학습)...
backend/       FastAPI + LangGraph Agent
frontend/      대시보드, 리포트
configs/       설정 파일
tests/         테스트
```

## 데이터 소스
| 소스 | 경로 | 건수 |
|------|------|------|
| 법령 | `data/raw/laws/` | 5,585건 |
| 판례 | `data/raw/precedents/` | 171,449건 |
| 해석례 | `data/raw/interpretations/` | 8,666건 |
| 표준계약서 | `data/raw/contracts/` | 285건 (6카테고리) |
| **시정조치(Seed)** | `data/seed/` | **2,488건** (Ground Truth) |

## 개발 파이프라인

### 0단계: EDA
키워드 히트율·텍스트 길이 분포 확인 → 도메인 필터링 기준 확정
- 산출물: `data/eda/eda_report.json`

### 1단계: 전처리
각 데이터를 조항/섹션 단위 chunking + 도메인 필터링 + 메타데이터 부착
- 법령 → 조문 단위 | 판례 → 판시사항/판결요지/이유 섹션 | 해석례 → 질의/회답/이유 섹션
- 표준계약서 → 조항 단위 | 시정조치 → 기존 파싱 활용
- 산출물: `data/processed/chunks/{law,case,interpretation,contract,ftc_cases}/`, `all_chunks.json`

### 2단계: Seed 라벨링
시정조치 사례에 Multi-label 패턴 라벨 부착 → Ground Truth 완성
- 패턴: 전면면책, 고의중과실면책, 손해배상상한, 간접손해배제, 일방적해지, 해지사유모호, 과도한위약금
- 산출물: `data/labels/seed_labeled.json`

### 3단계: DB 적재
PostgreSQL + pgvector 구축 (chunks, labels, error_logs 테이블)
- 임베딩: text-embedding-3-large → embedding 컬럼

### 4단계: 1차 모델 학습
Seed 데이터로 KoELECTRA 파인튜닝 → Backward 검증용 모델
- 출력: Multi-label 패턴 + 3-class 리스크 수준 (Low/Medium/High)
- 산출물: `models/v1/`

### 5단계: Forward-Backward Loop
표준계약서 + 판례 + 법령 대상으로 CLEAN/NOISE 분류
```
Forward  : GPT-4o CoT → (label, patterns, evidence_span)
Backward : evidence_span ⊂ 원문 확인 + 1차 모델 재라벨링
Decision : L == L' → CLEAN | L ≠ L' → NOISE
```
- 산출물: `data/labels/clean_labeled.json`, `data/labels/noise_rejected.json`

### 6단계: CLEAN 데이터 DB 추가 적재
CLEAN 데이터 + 임베딩 → chunks/labels 테이블 추가

### 7단계: 2차 모델 학습 (Data Flywheel)
Seed + CLEAN 합쳐서 KoELECTRA 재파인튜닝 → 5단계 반복
- 산출물: `models/v2/`, `models/v3/`...

### 8~11단계: 시스템 구현
- **8단계**: RAG (Hybrid Dense+BM25, Reranking)
- **9단계**: LangGraph Agent (분석/검색/판단/Red-team/근거검증/재검색)
- **10단계**: 실험·평가 (Baseline A/B vs Proposed, RAGAS)
- **11단계**: 프론트엔드 + PDF 리포트

## 핵심 알고리즘: Forward-Backward Consistency Check
```
Forward  : C → L         (GPT-4o가 조항 텍스트로 라벨 생성)
Backward : (C, L) → E    (학습된 모델이 근거 문구 추출, E ⊂ C)
Verify   : E → L'        (문구만 보고 재라벨링)
Decision : L == L' → CLEAN / L ≠ L' → NOISE
```

## 기술 스택
FastAPI · GPT-4o · KoELECTRA · text-embedding-3-large · PostgreSQL+pgvector · LangGraph · Hybrid Retrieval

## 코드 규칙
- Python 3.11+, 타입 힌트 필수, f-string 사용
- 파일 첫 줄에 파일 경로 주석 필수
- 함수명은 동사로 시작 (`fetch_`, `crawl_`, `save_`, `build_`)
- `print()` 금지 → `logger` 사용
- 모든 API 호출은 `try-except`로 감싸기
- 경로 하드코딩 금지 → `.env` 또는 `Path()` 사용
- 민감 정보는 `.env` 관리, Git 업로드 금지

## 데이터 수집
```bash
python scripts/crawl_law.py --key <인증키>           # 법령
python scripts/crawl_precedent.py --key <인증키>      # 판례
python scripts/crawl_interpretation.py --key <인증키>  # 해석례
python scripts/crawl_ftc_seed.py                      # 공정위 시정조치
```

## Git 규칙
- `main`: 안정 버전 / `dev`: 개발용
- 대용량 데이터 Git 업로드 금지

---
> Source: [gksmfly/ContractRisk-Guard](https://github.com/gksmfly/ContractRisk-Guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
