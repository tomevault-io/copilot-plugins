## figma-detail-page-agent

> AI 기반 상세페이지 자동 제작 시스템. 텍스트 초안 → 레이아웃 JSON 생성 → Figma 적용까지 자동화.

# 피그마 에이전트 프로젝트 컨텍스트

## 프로젝트 개요
AI 기반 상세페이지 자동 제작 시스템. 텍스트 초안 → 레이아웃 JSON 생성 → Figma 적용까지 자동화.

---

## 중요: 기준 템플릿

**모든 상세페이지 작업은 크래프트볼트 상세페이지를 기준으로 진행합니다.**

- 기준 파일: `크래프트볼트/craftvolt-chainsaw-v3-final.json`
- 템플릿 구조: `templates/detail-page-structure.json`

---

## 상세페이지 기본 구조 (필수) - 24섹션

**모든 상세페이지는 아래 24개 섹션 구조를 따릅니다.**
(핵심 기능 6개를 각각 개별 섹션으로 상세 설명)

### 섹션 구성

| 순서 | 섹션 ID | 섹션명 | 목적 |
|------|---------|--------|------|
| 01 | Hook | 후킹 | 강렬한 메인 카피로 시선 사로잡기 |
| 02 | WhatIsThis | 이게 뭔가요? | 제품을 한마디로 정의 |
| 03 | BrandName | 브랜드 의미 | 브랜드명의 의미와 철학 |
| 04 | SetContents | 세트 구성품 | 구매 시 받는 구성품 안내 |
| 05 | WhyCore | 핵심 기능 중요성 | 핵심 차별점이 왜 중요한지 |
| 06 | PainPoint | 페인포인트 공감 | 고객 불편함에 공감 |
| 07 | Solution | 해결책 제시 | 우리 제품의 해결 방법 |
| 08 | **FeaturesOverview** | **핵심 기능 개요** | **6가지 핵심 기능 한눈에 보기** |
| 09 | **Feature1_Detail** | **기능 1 상세** | **첫 번째 기능 Q&A 상세 설명** |
| 10 | **Feature2_Detail** | **기능 2 상세** | **두 번째 기능 Q&A 상세 설명** |
| 11 | **Feature3_Detail** | **기능 3 상세** | **세 번째 기능 Q&A 상세 설명** |
| 12 | **Feature4_Detail** | **기능 4 상세** | **네 번째 기능 Q&A 상세 설명** |
| 13 | **Feature5_Detail** | **기능 5 상세** | **다섯 번째 기능 Q&A 상세 설명** |
| 14 | **Feature6_Detail** | **기능 6 상세** | **여섯 번째 기능 Q&A 상세 설명** |
| 15 | Tips | 사용 꿀팁 | 활용 노하우 제공 |
| 16 | Differentiator | 핵심 차별화 | 경쟁사와 다른 우리만의 강점 |
| 17 | Comparison | 경쟁사 비교 | 직접 비교표 |
| 18 | Safety | 안전/신뢰 | 안전성, 인증, 품질 보증 |
| 19 | Target | 추천 대상 | 어떤 고객에게 적합한지 |
| 20 | Reviews | 고객 후기 | 실사용자 후기 |
| 21 | ProductSpec | 제품 스펙 | 상세 사양 정보 |
| 22 | FAQ | 자주 묻는 질문 | 구매 전 궁금증 해소 |
| 23 | Warranty | 보증/정책 | A/S, 보증, 환불 정책 |
| 24 | CTA | 구매 유도 | 최종 구매 전환 |

### 기능 상세 섹션 (09~14) 구조

각 기능 상세 섹션은 Q&A 형식을 따릅니다:

```
Feature_Num: 기능 번호 (01~06) - 큰 숫자로 시각적 구분
Question: "Q. 왜 [기능명]인가요?"
Answer: "A. [핵심 혜택 설명]"
IMAGE_AREA: 기능 시연/설명 이미지
Benefit_Title: 이 기능의 혜택 요약
Benefit_1~3: 구체적인 혜택 3가지 나열
Spec_Highlight: 관련 스펙 수치 강조 (선택)
```

---

## 색상 시스템

```json
{
  "brand_main": "브랜드 메인 컬러 (예: #FF6B00)",
  "accent": "강조 컬러 (예: #FFD700)",
  "dark_1": "#111111",
  "dark_2": "#1A1A1A",
  "image_placeholder": "#2A2A2A",
  "sub_text": "#888888",
  "muted_text": "#666666"
}
```

---

## 타이포그래피

| 용도 | 폰트 크기 | 굵기 |
|------|----------|------|
| 메인 카피 | 56px | 700 |
| 섹션 제목 | 40-48px | 700 |
| 기능 번호 | 72px | 700 |
| 서브 제목 | 26-32px | 500 |
| 질문 (Q) | 24-32px | 600 |
| 답변 (A) | 36px | 700 |
| 항목 제목 | 24-26px | 600 |
| 본문/설명 | 22px | 400 |
| 작은 글씨 | 20px | 400 |

---

## 텍스트 스타일 가이드라인 (중요)

### 1. 텍스트 정렬
- **기본 정렬: 가운데 정렬 (CENTER)**
- 모든 설명 텍스트, FAQ 답변, 팁 설명 등은 `textAlign: "CENTER"` 사용
- 스펙 목록, 비교표 등 나열형 콘텐츠만 `textAlign: "LEFT"` 사용

### 2. 긴 텍스트 처리
- **너비 제한**: 긴 텍스트는 반드시 `width: 760` 속성 추가
- **줄바꿈**: `\n`을 사용하여 자연스러운 위치에서 문장 분리
- **들여쓰기 금지**: 줄바꿈 후 공백 들여쓰기 사용하지 않음

### 3. 문장 분리 기준
```
✓ 좋은 예시 (자연스러운 문장 길이)
"체인이 너무 헐거우면 이탈 위험,\n너무 팽팽하면 마모가 빨라요.\n손으로 당겼을 때 3mm 정도 들리면 적당합니다."

✗ 나쁜 예시 (불필요한 들여쓰기)
"체인이 너무 헐거우면 이탈 위험,\n    너무 팽팽하면 마모가 빨라요."
```

### 4. JSON 텍스트 요소 예시
```json
{
  "type": "TEXT",
  "name": "Tip1_Desc",
  "content": "체인이 너무 헐거우면 이탈 위험,\n너무 팽팽하면 마모가 빨라요.\n손으로 당겼을 때 3mm 정도 들리면 적당합니다.",
  "fontSize": 22,
  "fontWeight": 400,
  "color": "#888888",
  "textAlign": "CENTER",
  "width": 760
}
```

### 5. 섹션별 텍스트 스타일

| 섹션 | 정렬 | 줄바꿈 | 비고 |
|------|------|--------|------|
| Tips | CENTER | O | 팁 설명은 2-3줄로 분리 |
| Safety | CENTER | O | 안전 기능 설명 분리 |
| Target | CENTER | X | 한 줄로 깔끔하게 |
| Reviews | CENTER | O | 후기 내용 3줄 내외 |
| FAQ | CENTER | O | 답변은 2-3줄로 분리 |
| Warranty | CENTER | O | 보증 설명 분리 |

---

## 레이아웃 기본값

- **캔버스 너비**: 860px
- **이미지 영역 너비**: 760px
- **섹션 배경**: dark_1, dark_2, brand_main 교차 사용

---

## 작업 시 참고사항

1. **새 상세페이지 생성 시**: 크래프트볼트 상세페이지 구조를 기준으로 작업
2. **기준 파일**: `크래프트볼트/craftvolt-chainsaw-v3-final.json`
3. **템플릿 구조**: `templates/detail-page-structure.json`
4. **핵심 기능**: 반드시 6가지 핵심 기능을 개별 섹션으로 상세 설명

---

## 주요 폴더 구조

```
figma-detail-page-agent/
├── templates/              # 상세페이지 기본 구조 템플릿
├── 크래프트볼트/            # 크래프트볼트 상세페이지 결과물 (기준)
│   └── craftvolt-chainsaw-v3-final.json  # 최신 기준 파일 (24섹션)
├── output/                 # AI 생성 결과물
└── figma-plugin/           # Figma 플러그인
    ├── manifest.json       # 플러그인 설정
    ├── code.js             # 플러그인 메인 코드
    └── ui.html             # 플러그인 UI
```

---

## 워크플로우

```
1. 제품 초안 작성 (6가지 핵심 기능 정의 필수)
      ↓
2. 레이아웃 JSON 생성 (24섹션 구조, 기준 템플릿 기반)
      ↓
3. Figma 플러그인으로 레이아웃 적용
      ↓
4. 수동 이미지 교체 및 미세 조정
```

---
> Source: [steveaimkt/figma-detail-page-agent](https://github.com/steveaimkt/figma-detail-page-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
