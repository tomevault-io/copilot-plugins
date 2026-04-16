## crm-analytics-lab

> **SQL로 배우는 CRM 분석 학습 플랫폼**

# CRM Analytics Lab - 프로젝트 컨텍스트

## 프로젝트 개요

**SQL로 배우는 CRM 분석 학습 플랫폼**

Streamlit 기반의 인터랙티브 학습 플랫폼으로, 사용자가 직접 SQL 쿼리를 작성하여 CRM 핵심 지표를 산출하고 배우는 시스템입니다.

### 학습 목표
- 모바일 앱/O2O 플랫폼 CRM 실무 역량 강화
- 실무 SQL 쿼리 작성 능력 향상

---

## 학습 모듈 구성

### 총 30개 문제

| 모듈 | 문제 수 | 난이도 | 핵심 개념 |
|------|--------|--------|----------|
| LTV & CAC | 5문제 | 기초~중급 | LTV, CAC, LTV:CAC 비율, Payback Period, ROI |
| Funnel 분석 | 5문제 | 기초~중급 | 전환율, 이탈률, 병목 지점, 디바이스별 분석 |
| Cohort 분석 | 5문제 | 중급~고급 | 코호트 생성, M+N 리텐션, 리텐션 매트릭스 |
| RFM 세그먼트 | 5문제 | 중급~고급 | R/F/M 계산, NTILE, 세그먼트 분류, 파레토 분석 |
| A/B 테스트 | 10문제 | 중급~고급 | Z-test, p-value, 표본 크기, 세그먼트별 분석, RPU |

### 문제 구성 요소

각 문제는 다음 요소로 구성:
- **상황**: 비즈니스 맥락 (왜 이 분석이 필요한지)
- **과제**: 구체적인 요구사항
- **힌트**: SQL 작성 가이드
- **정답 쿼리**: 모범 답안
- **해설**: 개념 설명
- **면접 TIP**: "Q: [질문]" 형태의 개념 설명 (면접 대비)

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| 웹 프레임워크 | Streamlit |
| 데이터베이스 | SQLite |
| 시각화 | Plotly |
| 언어 | Python 3.10+ |

---

## 프로젝트 구조

```
crm-analytics-lab/
├── app/                      # Streamlit 앱
│   ├── app.py                # 메인 앱
│   ├── components/           # 공통 컴포넌트
│   │   ├── question_card.py  # Question 데이터클래스, QuestionCard 컴포넌트
│   │   ├── sql_editor.py     # SQL 에디터
│   │   └── result_viewer.py  # 결과 뷰어/차트
│   └── modules/              # 학습 모듈
│       ├── ltv_cac.py        # LTV & CAC (5문제)
│       ├── funnel.py         # Funnel 분석 (5문제)
│       ├── cohort.py         # Cohort 분석 (5문제)
│       ├── rfm.py            # RFM 세그먼트 (5문제)
│       ├── ab_test.py        # A/B 테스트 (10문제)
│       └── dashboard.py      # 결과 대시보드
├── learning/                 # 학습 데이터
│   ├── data/
│   │   └── crm.db            # SQLite 데이터베이스
│   ├── setup_database.py     # DB 생성 스크립트
│   └── 0X_*/                 # 모듈별 TIP 문서
├── requirements.txt
├── README.md
└── CLAUDE.md
```

---

## 실행 명령어

```bash
# 데이터베이스 생성 (처음 한 번)
python learning/setup_database.py

# Streamlit 앱 실행
streamlit run app/app.py
```

---

## 데이터베이스 스키마

### customers
| 컬럼 | 타입 | 설명 |
|------|------|------|
| customer_id | INTEGER | 고객 ID (PK) |
| signup_date | DATE | 가입일 |
| acquisition_channel | TEXT | 획득 채널 (organic, google_ads 등) |

### transactions
| 컬럼 | 타입 | 설명 |
|------|------|------|
| transaction_id | INTEGER | 거래 ID (PK) |
| customer_id | INTEGER | 고객 ID (FK) |
| amount | REAL | 거래 금액 |
| transaction_date | DATE | 거래일 |

### events
| 컬럼 | 타입 | 설명 |
|------|------|------|
| event_id | INTEGER | 이벤트 ID (PK) |
| user_id | INTEGER | 사용자 ID |
| event_type | TEXT | 이벤트 유형 (page_view, purchase 등) |
| device | TEXT | 디바이스 (mobile, desktop) |
| channel | TEXT | 유입 채널 |
| event_date | DATETIME | 이벤트 시간 |

### campaigns
| 컬럼 | 타입 | 설명 |
|------|------|------|
| campaign_id | INTEGER | 캠페인 ID (PK) |
| channel | TEXT | 채널명 |
| spend | REAL | 지출액 |
| conversions | INTEGER | 전환 수 |

---

## 코딩 컨벤션

### Question 데이터클래스

```python
@dataclass
class Question:
    id: str              # 고유 ID (예: "ltv_1")
    title: str           # 제목 (예: "Q1. 전체 고객의 평균 LTV 계산")
    description: str     # 문제 설명 (상황 + 과제 + 요구사항)
    hint: str            # 힌트 (SQL 예시 포함)
    answer_query: str    # 정답 SQL 쿼리
    explanation: str     # 해설
    interview_tip: str   # 면접 TIP (Q: 질문 / 답변 형식)
    difficulty: int      # 난이도 (1-5)
```

### 면접 TIP 형식

```
**Q: [면접관 질문]**

"[개념 설명]

예를 들어...

이것이 중요한 이유는..."
```

---

## 현재 상태

- [x] 데이터베이스 구축
- [x] Streamlit 앱 기본 구조
- [x] 공통 컴포넌트 (QuestionCard, SQLEditor, ResultViewer)
- [x] LTV & CAC 모듈 (5문제)
- [x] Funnel 분석 모듈 (5문제)
- [x] Cohort 분석 모듈 (5문제)
- [x] RFM 세그먼트 모듈 (5문제)
- [x] A/B 테스트 모듈 (10문제)
- [x] 결과 대시보드
- [x] 개념 학습 페이지 (AARRR, Lifecycle, 지표 관계도, 의사결정 프로세스)

---

## 고도화 진행 상황 (2025-01)

### 완료된 기능

#### 1. 학습 진행 저장 ✅
- `app/components/progress_manager.py` 신규 생성
- `user_progress` 테이블: question_id, is_completed, attempts, last_query, solved_at
- 앱 시작 시 DB에서 진행 데이터 로드
- 문제 풀이 시 자동 저장 (정답/오답 모두)
- 새로고침해도 진행률 유지

#### 2. 결과 기반 채점 ✅
- `app/components/result_checker.py` 신규 생성
- 쿼리 문자열 비교 → 실행 결과 비교로 변경
- 채점 기준: 행 수(30점) + 컬럼 수(20점) + 값(50점)
- 부분 점수 지원 (0-100점)
- 상세 피드백 제공 ("행 수가 적습니다", "값이 80% 일치합니다" 등)
- 옵션: 컬럼명 무시, 순서 무시, 소수점 2자리 반올림

#### 3. 힌트 단계별 제공 ✅
- `question_card.py`에 `_render_step_hints()`, `_parse_hint_steps()` 추가
- 힌트 형식: "---" 구분자로 3단계 분리
  - 1단계: 🎯 접근 방향 (노랑)
  - 2단계: 🔧 필요한 함수/문법 (파랑)
  - 3단계: 📝 쿼리 뼈대 (보라)
- 버튼 클릭으로 순차 공개
- 힌트 분리 완료 모듈: LTV & CAC, Funnel

### 다음 작업 (TODO)

#### 힌트 단계별 분리 (남은 모듈)
- [ ] `app/modules/cohort.py` - 5문제 힌트를 "---" 구분자로 분리
- [ ] `app/modules/rfm.py` - 5문제 힌트를 "---" 구분자로 분리
- [ ] `app/modules/ab_test.py` - 10문제 힌트를 "---" 구분자로 분리

#### boost.md 참고 - 추가 고도화 후보
- [ ] 오답노트 기능
- [ ] 난이도/태그 필터링
- [ ] 문제 네비게이션 (이전/다음)
- [ ] SQL 에디터 개선 (하이라이트, 템플릿 버튼)
- [ ] 에러 메시지 한국어 번역
- [ ] 배포 (Streamlit Cloud)

### 힌트 분리 가이드

기존 힌트를 아래 형식으로 수정:
```python
hint="""접근 방향 설명 (무엇을 해야 하는지)
---
필요한 함수/문법: SUM(), GROUP BY, JOIN 등
---
쿼리 뼈대 예시:
SELECT ...
FROM ...
GROUP BY ..."""
```

---

## 향후 계획

1. 고도화 기능 완성 (힌트 분리, 오답노트, 필터링)
2. Streamlit Cloud 배포
3. 포트폴리오 정리 및 공개
4. 면접 실전 연습

---

## 참고 문서

- `PROJECT_SUMMARY.md`: 프로젝트 요약 (AI 상담용)
- `boost.md`: GPT/Gemini 고도화 제안 (참고용)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GeonYul2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
