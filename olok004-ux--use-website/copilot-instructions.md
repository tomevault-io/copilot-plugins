## use-website

> > 이 문서는 AI(Codex)가 USE 모바일 웹 프로젝트 작업 시 반드시 따라야 하는 **작업 규칙**입니다.

# AGENTS.md — AI Context Document

> 이 문서는 AI(Codex)가 USE 모바일 웹 프로젝트 작업 시 반드시 따라야 하는 **작업 규칙**입니다.
> 디자인 토큰·컴포넌트 스펙·색상 등 디자인 관련 내용은 **`DESIGN.md`** 를 참조하세요.

---

## 작업 원칙

- **확장성/유연성 검토**: 현재 요구사항을 해결하되, 향후 확장이 막히지 않는 구조 확인
- **기존 코드 재사용**: 새로 만들기 전에 `components/`, `theme.js` 등 기존 리소스를 먼저 탐색
- **커뮤니케이션**: 항상 **개요(왜, 무엇을)** → **상세 구현 계획** 순서로 설명

---

## 작업 프로세스 (필수!)

> ⚠️ **추측을 사실처럼 말하지 말 것!** 모든 가설은 반드시 검증 후 결론.
> ⛔ **코드 작성 전 반드시 4단계까지 완료하고 사용자 승인을 받을 것.**

### 1단계: 문제/요청 이해
- 문제 현상을 명확히 기술
- 불분명한 부분이 있으면 사용자에게 질문
- "~일 것 같습니다"가 아니라 실제 코드를 확인

### 2단계: 원인 분석 (문제 해결의 경우)
- 가설 수립 → 가설 검증 → 원인 확정
- ❌ "이게 원인입니다" (검증 없이)
- ✅ "가설: ~일 수 있습니다. 검증해보겠습니다." → "확인 결과, ~가 원인입니다"

### 3단계: 해결책 탐색
- 해결 방안 2-3개 제시, 각 방안의 영향 범위 분석
- 사전 검증 가능하면 검증

### 4단계: 작업 계획 보고 (코드 작성 전 필수!)

> ⛔ 사용자가 "그냥 빨리 해줘"라고 해도, 이 보고를 먼저 하세요.

```
📋 작업 계획 보고

🔍 문제 상황 (What's wrong?)
어떤 상황에서 어떤 증상이 발생하는지, 왜 이 작업이 필요한지.

🎯 목표 (What we want to achieve)
이 작업이 완료되면 어떤 상태가 되어야 하는지.

🔬 원인 분석 (Why it happens) - 문제 해결의 경우
검증된 원인만 기술. 추측은 "가설"이라고 명시.

📁 변경 예정 파일
| 파일 경로 | 변경 내용 | 비고 |
|-----------|----------|------|

⚡ Before → After
[Before] 현재 상태
[After] 작업 후 기대 상태

🎨 디자인 토큰 사용 계획 (DESIGN.md 참조)
- 사용할 JS 토큰: colors.*, spacing.*, typography.* 등
- 재사용할 컴포넌트: Button(primary), Badge(dday) 등
- 새로 필요한 토큰: 있으면 명시 (없으면 "없음")

이대로 진행해도 될까요?
```

### 5단계: 작업 실행
- 승인받은 계획대로 진행
- 예상치 못한 상황 → 중단 후 보고

### 6단계: 결과 검증

| # | 확인 항목 | 구체적 검증 방법 |
|---|-----------|-----------------|
| 1 | 헥스 코드 미사용 | 새 코드에 `#` + 6자리 패턴 없는지 확인 |
| 2 | 토큰 외 spacing 미사용 | `8px`, `15px`, `24px` 등 토큰에 없는 값 없는지 확인 |
| 3 | 토큰 외 font-size 미사용 | 8단계 scale 외 임의 값 없는지 확인 |
| 4 | 기존 컴포넌트 재사용 | Button/Badge/CouponCard로 대체 가능한 부분 없는지 확인 |
| 5 | 토큰 추가 시 3파일 동기화 | theme.css, theme.js, tailwind.config.js 모두 수정했는지 확인 |
| 6 | 기존 기능 정상 동작 | 기존 컴포넌트/페이지 깨지지 않았는지 확인 |

### 7단계: 작업 완료
- 6단계 검증 **전부 통과** 후에만 "완료" 선언
- 변경 사항 요약 보고

---

## 금지 사항

| 금지 | 이유 | 올바른 대안 |
|------|------|-------------|
| 허락 없이 새 파일/컴포넌트 생성 | 프로젝트 구조 임의 변경 방지 | 사용자에게 먼저 제안 후 승인 |
| 기존 아키텍처 임의 변경 | 설계 의도 훼손 방지 | 변경 필요 시 이유와 함께 제안 |
| 요청 범위 밖 리팩토링 | 스코프 크립 방지 | "이 부분도 개선하면 좋겠는데, 할까요?" |
| 문제 발견 시 바로 수정 | 사용자가 다른 해결책을 원할 수 있음 | 문제 보고 → 해결책 2-3개 제시 → 승인 후 수정 |
| 디자인 토큰 없이 스타일링 | 디자인 시스템 일관성 파괴 | 항상 DESIGN.md 토큰 사용 |

---

## 이전 세션 작업 이어받을 때

1. "완료됐다"는 요약을 그대로 믿지 말 것
2. 실제 코드 상태를 직접 확인 (`Read`, `Grep`으로 검증)
3. 동작 테스트로 검증 후 진행

---

## 📁 프로젝트 구조

```
C:\Users\COM\
├── AGENTS.md                    ← 이 파일 (AI 작업 규칙)
├── DESIGN.md                    ← 디자인 토큰·컴포넌트·금지사항 (디자인 관련 모든 것)
├── theme.css                    ← CSS 변수 정의 (원본 헥스값, 폰트 import)
├── theme.js                     ← JS 토큰 객체 (colors, typography, spacing, radius, shadow)
├── tailwind.config.js           ← Tailwind ↔ CSS 변수 매핑
├── DesignSystemPage.jsx         ← 디자인 시스템 쇼케이스 페이지
└── components/
    ├── Button/
    │   └── Button.jsx           ← variant: primary / secondary / tertiary
    ├── Badge/
    │   └── Badge.jsx            ← type: dday / dn / price / usage / distance / channel / number
    └── CouponCard/
        └── CouponCard.jsx       ← layout: vertical 3종 / horizontal 4종
```

---

## 🛠️ 기술 스택

- **언어**: JavaScript (JSX)
- **스타일링**: Inline Style (theme.js 토큰 참조) + CSS Variables (theme.css)
- **Tailwind**: tailwind.config.js에 매핑 구성됨 (선택적 사용 가능)
- **폰트**: SUIT (CDN 또는 로컬 설치, theme.css에서 import)
- **디자인 소스**: Figma "USE 모바일 웹" (fileKey: 76JViat4fOexiK6F77cuBJ)
- **컴포넌트**: Inline Style 기반 React 함수형 컴포넌트 (theme.js 토큰 참조)

---
> Source: [olok004-ux/use.website](https://github.com/olok004-ux/use.website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
