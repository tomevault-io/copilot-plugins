## seoulhiim-dashboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

서울안녕내과(Seoul Hi-IM Internal Medicine) 환자 데이터 대시보드. Streamlit 멀티페이지 앱으로 Google Sheets에서 환자 진료 데이터를 읽어 시각화한다. Streamlit Cloud에 배포됨.

## Running Locally

```bash
streamlit run 환자정보.py
```

## Architecture

- **`환자정보.py`** — 메인 엔트리포인트. KPI(전년 동기 delta 포함), 전년 동기 비교, 월간 성장률, 내원 추이(area+MA 토글), 요일×시간대 히트맵, 환자 지도 분포, 연령대 수평 바 차트.
- **`pages/2_지역장악도.py`** — 행정동·연령대별 시장 침투율(장악도) 분석. 계층형 필터(시/도→시/군/구→행정동), 하위 지역 랭킹 차트(클릭-투-드릴다운), 인구 대비 환자 비율 시각화.
- **`pages/3_마케팅성과분석.py`** — 캠페인 전후 성과 비교. 단일 스크롤 역삼각형 레이아웃. 캠페인 순수 효과(타겟 vs 비타겟 Lift), 신환 트렌드, 지역별 성과, 신환 인구통계, 재방문 분석. 신환 수는 nunique 기반. 타겟 지역은 시/군/구→행정동 계층형 필터.

모든 페이지는 동일한 패턴을 따른다:
1. `authenticate()` — 사이드바 비밀번호 인증 (`st.secrets["general"]["APP_PASSWORD"]`)
2. `@st.cache_data`로 Google Sheets 데이터 로드 (`gspread` + GCP 서비스 계정)
3. 사이드바 필터 → pandas 전처리 → Altair 차트 렌더링

## Data Source

Google Sheets (ID: `st.secrets["google_sheets"]["sheet_id"]`)의 워크시트:
- **Sheet1** — 환자 진료 데이터 (컬럼: 진료일자, 진료시간, 나이, 성별, 환자번호, 초/재진, 행정동, 시/도, 시/군/구, x, y 등)
- **연령별인구현황** — 행정동별 연령대 인구 데이터 (지역장악도 페이지에서 사용)

## Secrets

로컬: `.streamlit/secrets.toml` (TOML 형식, `.gitignore`에 등록됨)
배포: Streamlit Cloud secrets 탭

섹션 구조:
- `[gcp_service_account]` — GCP 서비스 계정 인증 정보
- `[google_sheets]` — sheet_id, worksheet_name
- `[general]` — APP_PASSWORD

## Key Libraries

- **streamlit** — 웹 앱 프레임워크
- **altair** — 차트 (모든 시각화에 사용)
- **folium / streamlit-folium** — 지도 시각화 (환자정보 페이지)
- **gspread** — Google Sheets API 클라이언트
- **pandas** — 데이터 처리

## Conventions

- 모든 컬럼명, 변수명, UI 텍스트는 한국어
- 연령대 분류: 10세 단위 (`9세이하`, `10대`, ..., `90대`, `100세이상`)
- 진료일자 포맷: 원본은 `%Y%m%d` 문자열 → `pd.to_datetime`으로 변환
- 페이지 아이콘: `서울안녕내과.ico`

### 차트 디자인 원칙

- 레이아웃 순서: KPI → 비교/평가 → 상세 탐색 → 패턴/분포 (역삼각형 구조)
- 차트는 패턴을 보여주고, 정확한 숫자는 툴팁(인터랙션)으로 제공
- 시계열 x축: `labelExpr`로 줌 레벨별 자동 포맷 (연초→연도, 월초→월, 그 외→월-일)
- 날짜 표시: 영어 월명 사용 금지. 숫자 포맷 사용 (`%Y-%m-%d`, `%Y-%m`)
- 요일 표시: 한글 (월요일~일요일)
- 내원 추이: 진료횟수는 area(배경), MA는 line(범례 토글)
- 환자 지도: 진료횟수가 아닌 고유 환자수 기준 (1인 1점)
- KPI help 툴팁: "정의 → 해석 가이드 → 비교값" 3단 구조
- 환자 수 집계: 모든 페이지에서 고유 환자 기준 (`nunique` / `drop_duplicates`)
- 지역 필터: 빈 문자열 포함 제거 (`str.strip().astype(bool)`)
- 툴팁 날짜: temporal 타입(`:T`) 사용 금지. 문자열 컬럼(`.dt.strftime()`)으로 변환 후 `:N`으로 표시
- 성별 표시: 원본 M/F → 한글 남성/여성으로 변환 (`.replace({'M': '남성', 'F': '여성'})`)

---
> Source: [hey738/seoulhiim-dashboard](https://github.com/hey738/seoulhiim-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
