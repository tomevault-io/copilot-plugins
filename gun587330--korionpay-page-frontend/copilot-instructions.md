## korionpay-page-frontend

> > 이 파일은 작업 중 **반드시 지켜야 할 규칙**을 정의한다.

# KORION PAY — 프로젝트 작업 규칙 (CLAUDE.md)

> 이 파일은 작업 중 **반드시 지켜야 할 규칙**을 정의한다.
> 상세 배경/이유는 [docs/PROJECT_PLAN.md](docs/PROJECT_PLAN.md) 참고.
> 외주 인계용 결과물이므로 **"받는 사람이 코드만 보고 이해/이식할 수 있게"** 가 최우선.

---

## 1. 작업 범위
- **프론트엔드 UI(퍼블리싱)만** 구현한다.
- 결제 API 연동, 백엔드 통신, 비즈니스 로직, 인증은 **만들지 않는다.**
- 버튼 동작 등 인터랙션은 Figma 확인 후 별도 협의 전까지 **UI 상태만** 구현.

## 2. 기술 스택 (확정 — 임의 변경 금지)
- Vite + React + TypeScript
- 스타일: **CSS Modules** (`*.module.css`)
- 추가 라이브러리 도입 시 **이유를 PROJECT_PLAN.md에 기록하고** 최소화한다.

## 3. 폴더 구조 — Atomic Design
```
src/
├─ components/{atoms,molecules,organisms}/
├─ templates/        # 데이터 없는 페이지 레이아웃
├─ pages/            # 실제 화면(라우트 단위)
├─ styles/           # tokens.css, global.css
└─ types/            # 공용 타입
```
- 컴포넌트 위치 판단 기준:
  - **atom**: 더 못 쪼갬 (Button, Input, Icon)
  - **molecule**: atom 묶음 (LabeledInput)
  - **organism**: molecule 묶은 큰 덩어리 (Header, PaymentMethodCard)
- 컴포넌트 1개 = 폴더 1개 = `Component.tsx` + `Component.module.css` (+ `index.ts`).

## 4. 컴포넌트 작성 규칙
- 함수형 컴포넌트 + TypeScript. **props는 `interface`로 명시** (사용법이 타입으로 드러나게).
- **UI와 로직 분리**: 데이터 흐름/상태는 가능한 한 props로 주입받아 재사용·이식이 쉽게.
- 하드코딩 금지: 색/간격/폰트/radius는 **`styles/tokens.css`의 CSS 변수**를 참조.
- 파일/컴포넌트 네이밍: 컴포넌트는 `PascalCase`, 그 외 `camelCase`. 일관성 유지.
- 매직 넘버/문자열은 의미 있는 상수로.

## 5. 스타일 규칙
- 전역 스타일 오염 금지 → 모든 컴포넌트 스타일은 `*.module.css`로 격리.
- 디자인 토큰은 `tokens.css`에 모으고 컴포넌트는 참조만.

## 6. 반응형 규칙
- 반응형 범위는 휴리스틱하게 적용하되(모바일 고려),
- **반응형 때문에 들어간 코드(미디어쿼리, 정렬 전환 등)에는 반드시 주석으로 의도를 명시.**
  - 예: `/* 반응형: 모바일(<768px)에서 가로→세로 정렬 */`

## 7. 주석 / 가독성 규칙
- 주석은 **한국어**로 (인계 대상이 한국 발주처).
- 주석은 "무엇(what)"이 아니라 **"왜(why)"** 위주. 코드로 자명한 건 달지 않는다.
- 반응형 처리, 예외 처리, 비직관적 결정에는 짧고 정확한 주석을 단다.
- 함수/컴포넌트는 짧고 단일 책임. 깊은 중첩보다 분리.

## 8. 인계(이식) 우선 원칙
- 발주처 코드베이스 정보가 없으므로 **특정 레포 병합이 아니라 "떼어다 쓰기 쉬운"** 구조를 지향.
- 컴포넌트 간 결합도를 낮추고, 외부 의존(전역 상태 등)을 최소화한다.

## 9. 작업 절차
- 모호하거나 와이어프레임에 정보가 부족하면 **임의로 추측해 진행하지 말고 먼저 질문**한다.
- 새 페이지/컴포넌트 추가 시 PROJECT_PLAN.md의 8번(분류표)을 갱신한다.
- 큰 결정(스택 추가, 구조 변경)은 사용자 컨펌 후 진행.

## 10. 비밀/보안
- Figma 토큰·API 키 등 비밀값을 코드/문서/깃에 커밋하지 않는다.

## 11. 다국어 (i18n)
- 화면의 **UI 텍스트는 하드코딩하지 않고** `useTranslation().t('키')`로 출력한다.
  번역 문구는 `src/i18n/ko.json` / `en.json`에 "키 → 문구"로 추가한다.
- 우측 상단 버튼으로 한↔영 전환(`LanguageProvider`). 기본은 한국어.
- **번역 대상 = UI 텍스트만** (제목, 라벨, 버튼, 컬럼명, 메뉴 등).
  **데이터 값은 번역하지 않는다** (행의 이름·코드·날짜·숫자, 상태/태그/액션 enum 등).
- 데이터 훅(JSON)에 들어가는 라벨은 `labelKey`로 저장하고, 훅에서 `t()`로 번역해 반환한다.

## 12. 데이터 패턴 (JSON + 훅)
- 화면 데이터는 `*.json`(하드코딩)에 두고, `use*` 훅이 읽어 반환한다.
- 추후 실데이터 연동 시 **훅 내부만** API 호출로 교체하면 화면 컴포넌트는 그대로 동작하게 한다.

---
_규칙 변경 시 이 파일과 PROJECT_PLAN.md를 함께 갱신한다._

---
> Source: [gun587330/KORIONPAY_PAGE_FRONTEND](https://github.com/gun587330/KORIONPAY_PAGE_FRONTEND) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
