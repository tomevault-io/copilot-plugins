## goeq-bi

> CQMS 프로젝트 전체에 적용되는 아키텍처 표준, 폴더 구조, 명명 규칙을 정의합니다.

---
alwaysApply: true
---

# 프로젝트 표준

CQMS 프로젝트 전체에 적용되는 아키텍처 표준, 폴더 구조, 명명 규칙을 정의합니다.

## 프로젝트 아키텍처

### 계층별 책임 분리

```
CQMS (Customer Quality Management System)
├── _00_database/           # 데이터베이스 추상화 계층
│   ├── 역할: 데이터베이스 연결 및 클라이언트 관리
│   └── 제약: 순수 연결 관리, 비즈니스 로직 금지
│
├── _01_query/              # SQL 쿼리 생성 계층
│   ├── 역할: SQL 문자열 생성 (순수 함수)
│   ├── 반환: str (SQL 쿼리 문자열)
│   └── 제약: 데이터베이스 실행 금지, pandas 사용 금지
│
├── _02_preprocessing/      # 데이터 처리 계층
│   ├── 역할: 비즈니스 로직, 데이터 집계, 계산
│   ├── 반환: pd.DataFrame
│   └── 제약: SQL 생성 금지
│
├── _03_visualization/      # 시각화 계층
│   ├── 역할: Plotly 차트 생성
│   ├── 반환: plotly.graph_objects.Figure
│   └── 제약: 데이터 처리 금지
│
├── _04_pages/              # UI 계층
│   ├── 역할: Streamlit UI 컴포넌트
│   └── 제약: 직접적인 데이터 처리 금지
│
├── _05_commons/            # 공통 모듈
│   ├── 역할: 설정, 상수, 데이터클래스
│   └── 제약: 계층별 의존성 없는 순수 모듈
│
├── _06_assets/             # 정적 자원
├── _07_docs/               # 문서
├── _08_automation/         # 자동화 스크립트
└── _09_test/               # 테스트 유틸리티
```

### 단방향 의존성 규칙

```
_04_pages (UI)
    ↓
_03_visualization (Charts)
    ↓
_02_preprocessing (Data Processing)
    ↓
_01_query (SQL Generation)
    ↓
_00_database (DB Connection)

_05_commons ← 모든 계층에서 참조 가능
```

## 폴더 구조 표준

### 시스템별 하위 폴더

각 시스템별로 일관된 하위 폴더 구조를 유지합니다:

```
_01_query/
├── CQMS/                   # Customer Quality Management System
├── GMES/                   # Global Manufacturing Execution System
├── HOPE/                   # Hankook OE Product Environment
├── HGWS/                   # Hankook Global Warranty System
├── SAP/                    # SAP 시스템
├── CTMS/                   # Component Testing Management System
├── config_sql.py           # 공통 SQL 유틸리티
└── test_framework.py       # 테스트 프레임워크

_02_preprocessing/
├── CQMS/                   # CQMS 데이터 처리
├── GMES/                   # GMES 데이터 처리  
├── HOPE/                   # HOPE 데이터 처리
└── test_preprocessing_common.py  # 공통 테스트

_04_pages/
├── _01_DASHBOARD/          # 대시보드 페이지
├── _02_ANALYSIS/           # 분석 도구
├── _03_MONITORING/         # 모니터링
├── _04_COLLABORATION/      # 협업 기능
├── _05_USER_GUIDE/         # 사용자 가이드
├── _06_WORKPLACE/          # 작업 관리
├── _07_SETTINGS/           # 설정
├── _08_ADMIN/              # 관리자 기능
└── _09_SYSTEM/             # 시스템 유틸리티
```

## 명명 규칙

### 함수 명명 표준

#### _01_query/ 계층
```python
# Dispatch 함수 (공개 인터페이스)
def dispatch_{system}_{domain}_query(params: ParamsClass) -> str:
    pass

# 내부 CTE 함수
def _cte_{system}_{domain}(params: ParamsClass) -> str:
    pass

# 내부 뷰 함수
def _view_{aggregation}_{domain}(params: ParamsClass) -> str:
    pass

# 예시
def dispatch_cqms_quality_issue_query(params: QualityIssueTasksParams) -> str:
def _cte_cqms_quality_issue(params: QualityIssueTasksParams) -> str:
def _view_rawdata_quality_issue(params: QualityIssueTasksParams) -> str:
```

#### _02_preprocessing/ 계층
```python
# Assembly 함수 (공개 인터페이스)
def assemble_{domain}_df(params: ParamsClass) -> pd.DataFrame:
    pass

# 내부 집계 함수
def _agg_{purpose}_{domain}(df: pd.DataFrame) -> pd.DataFrame:
    pass

# 내부 계산 함수
def _calc_{metric}_{domain}(df: pd.DataFrame) -> pd.DataFrame:
    pass

# 예시
def assemble_quality_issue_df(params: QualityIssueTasksParams) -> pd.DataFrame:
def _agg_monthly_quality_issue(df: pd.DataFrame) -> pd.DataFrame:
def _calc_oeqi_quality_issue(df: pd.DataFrame) -> pd.DataFrame:
```

#### _03_visualization/ 계층
```python
# 차트 생성 함수
def create_{chart_type}_{domain}_chart(df: pd.DataFrame) -> go.Figure:
    pass

# 예시
def create_bar_quality_issue_chart(df: pd.DataFrame) -> go.Figure:
def create_line_trend_chart(df: pd.DataFrame) -> go.Figure:
```

### 변수 명명 표준

#### 데이터프레임 변수
```python
# 원본 데이터
raw_df = pd.DataFrame()
source_df = pd.DataFrame()

# 처리된 데이터
processed_df = pd.DataFrame()
aggregated_df = pd.DataFrame()
filtered_df = pd.DataFrame()

# 시스템별 접두사 사용
cqms_df = pd.DataFrame()
gmes_df = pd.DataFrame()
hope_df = pd.DataFrame()
```

#### 매개변수 클래스
```python
# 시스템 + 도메인 + TasksParams
QualityIssueTasksParams     # CQMS 품질이슈
ChangeTaskParams            # CQMS 4M변경
SellinTasksParams          # HOPE 셀인
NonconformityParams        # GMES 부적합
```

## 문서화 표준

### 모듈 독스트링 표준

#### _01_query/ 계층 독스트링 템플릿

```python
"""[시스템명] [도메인명] SQL 쿼리 생성 모듈

[모듈의 핵심 목적을 1-2줄로 간단히 설명]

주요 테이블: [TABLE1, TABLE2, TABLE3]
지원 모드: [mode1, mode2, mode3] ([추가 설명이 필요한 경우])
"""
```

#### 예시
```python
"""CQMS 품질 이슈 관리 SQL 쿼리 생성 모듈

현장 품질 이슈의 전체 생명주기를 추적하고 분석하기 위한
종합적인 쿼리 솔루션을 제공합니다.

주요 테이블: CQMS_QUALITY_ISSUE, CQMS_ISSUE_CATEGORY_DATA, ZHRT90041
지원 모드: rawdata (이슈 + 카테고리 + 자재 + 담당자 통합)
"""

"""GMES 롤링 저항(RR) SQL 쿼리 생성 모듈

다양한 테스트 방법별 롤링 저항 성능 분석과
예상 합격률 계산을 위한 쿼리를 생성합니다.

주요 테이블: QLT_D_LQLTTR309, QLT_F_LQLTTR316, QLT_D_LQLTTR510, MAS_D_LMASTR101
지원 모드: rawdata, standard, oe_spec (ISO28580, SVP, SAE J1269 테스트 방법)
"""

"""HOPE 셀인(Sell-in) SQL 쿼리 생성 모듈

Oracle 및 SQLite 데이터베이스에서 월별 셀인 데이터를 조회하는 
쿼리를 생성합니다.

주요 테이블: VW_SF_HOPE_SELLIN_SUMMARY (Oracle), sellin_monthly_agg (SQLite)
지원 모드: Oracle BI 전체 데이터, SQLite 월별/연도별 집계
"""
```

#### 독스트링 작성 원칙

1. **간결성**: 핵심 정보만 포함, 상세 설명 최소화
2. **일관성**: 모든 쿼리 모듈에 동일한 템플릿 적용
3. **실용성**: 개발자가 모듈 용도를 즉시 파악 가능
4. **표준화**: 템플릿 구조 엄격 준수

#### 금지 사항

- 장황한 개요 및 기능 설명
- 상세한 데이터 아키텍처 다이어그램
- 활용 사례 나열
- 제한사항 및 기술적 세부사항
- 작성자, 날짜 등 메타데이터

## 설정 관리

### 환경 변수 표준

```python
# 필수 환경 변수
PROJECT_ROOT = "/home/jumasi/workstation"
CQMS_ENV = "development"  # development, production

# 데이터베이스 설정
SNOWFLAKE_ACCOUNT = "..."
SNOWFLAKE_USER = "..."
SNOWFLAKE_PASSWORD = "..."
SQLITE_DB_PATH = "..."

# 세션 설정
SESSION_TIMEOUT_MINUTES = 30
MAX_LOGIN_ATTEMPTS = 3
```

### 상수 관리 위치

```python
# 전역 상수
_05_commons/config.py

# 시스템별 상수
_01_query/config_sql.py      # SQL 관련 상수
_04_pages/config_pages.py    # 페이지 설정
```

## 데이터 플로우 패턴

### 표준 데이터 처리 흐름

```python
# 1. 매개변수 생성 (Dataclass)
params = QualityIssueTasksParams(
    view_mode="rawdata",
    start_date="20240101",
    end_date="20241231"
)

# 2. SQL 쿼리 생성 (_01_query/)
query = dispatch_cqms_quality_issue_query(params)

# 3. 데이터 처리 (_02_preprocessing/)
df = assemble_quality_issue_df(params)

# 4. 시각화 (_03_visualization/)
chart = create_bar_quality_issue_chart(df)

# 5. UI 표시 (_04_pages/)
st.plotly_chart(chart)
```

### 에러 처리 패턴

```python
# 매개변수 검증
def validate_params(params):
    if not params.start_date:
        raise ValueError("start_date is required")

# 데이터 검증
def validate_dataframe(df):
    if df.empty:
        raise ValueError("No data found")

# 연결 검증
def validate_connection(client):
    if not client.is_connected():
        raise ConnectionError("Database connection failed")
```

## 테스트 표준

### 테스트 파일 위치

```
_01_query/test_framework.py        # 쿼리 테스트 프레임워크
_02_preprocessing/test_preprocessing_common.py  # 전처리 테스트
각 모듈의 if __name__ == "__main__":  # 내장 테스트
```

### 테스트 명명 규칙

```python
def test_{module}_{function}_basic():           # 기본 기능 테스트
def test_{module}_{function}_with_filters():   # 필터 조건 테스트
def test_{module}_{function}_error_handling(): # 에러 처리 테스트
```

## 버전 관리

### 브랜치 전략

```
main                    # 프로덕션 브랜치
├── develop            # 개발 통합 브랜치
├── feature/{기능명}    # 기능 개발 브랜치
├── bugfix/{버그명}     # 버그 수정 브랜치
└── hotfix/{긴급수정}   # 핫픽스 브랜치
```

### 커밋 메시지 표준

```
feat: 새로운 기능 추가
fix: 버그 수정
docs: 문서 수정
style: 코드 스타일 변경
refactor: 코드 리팩토링
test: 테스트 추가/수정
chore: 빌드 프로세스 또는 보조 도구 변경
```

## 코드 리뷰 표준

### 필수 체크 항목

- [ ] **아키텍처 준수**: 계층별 책임 분리 확인
- [ ] **명명 규칙**: 함수명, 변수명 표준 준수
- [ ] **의존성 방향**: 단방향 의존성 확인
- [ ] **데이터 플로우**: 표준 처리 흐름 준수
- [ ] **테스트 포함**: 적절한 테스트 코드 포함
- [ ] **문서화**: 필요한 문서 업데이트

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jumasi-34) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
