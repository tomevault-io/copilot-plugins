## business-card-maker

> 이 문서는 Gemini와 함께 구상한 명함 제작 사이트의 개발 계획 및 아키텍처 설계를 담고 있습니다. `src` 디렉토리 구조와 `shadcn/ui` 사용을 반영한 최신 버전입니다.

# 명함 제작 사이트 개발 계획 by Gemini (v2)

이 문서는 Gemini와 함께 구상한 명함 제작 사이트의 개발 계획 및 아키텍처 설계를 담고 있습니다. `src` 디렉토리 구조와 `shadcn/ui` 사용을 반영한 최신 버전입니다.

## 1. 디렉토리 구조

`src` 폴더를 사용하는 표준 Next.js 프로젝트 구조입니다.

```
business-card-maker/
├── 📂 src/
│   ├── 📂 app/              # 페이지 및 레이아웃
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx        # 메인 페이지
│   │
│   ├── 📂 components/        # 재사용 가능한 모든 컴포넌트
│   │   ├── 📂 business-card/ # 명함 제작 관련 기능 컴포넌트
│   │   │   ├── CardEditor.tsx
│   │   │   └── CardPreview.tsx
│   │   │
│   │   ├── 📂 templates/     # 실제 명함 디자인 템플릿
│   │   │   ├── TraditionalTemplate.tsx
│   │   │   └── QRTemplate.tsx
│   │   │
│   │   └── 📂 ui/            # shadcn/ui로 추가되는 UI 컴포넌트
│   │       ├── button.tsx
│   │       ├── input.tsx
│   │       └── card.tsx
│   │
│   ├── 📂 lib/             # 재사용 가능한 함수, 로직 (shadcn utils 포함)
│   │   ├── download.ts
│   │   └── utils.ts
│   │
│   └── 📂 types/             # TypeScript 타입 정의
│       └── index.ts
│
├── 📂 public/
├── components.json         # shadcn/ui 설정 파일
├── tailwind.config.ts      # Tailwind CSS 설정 파일
└── ...
```

---

## 2. 기술 스택 및 설정

- **프레임워크:** `Next.js` (App Router)
- **UI:** `React` + `Tailwind CSS`
- **UI 컴포넌트 라이브러리:** `shadcn/ui`
  - **설치:** `npx shadcn@latest init` 으로 `components.json` 생성 및 설정.
  - **컴포넌트 추가:** `npx shadcn@latest add [component]` 명령어로 필요한 컴포넌트(button, input, card 등)를 `src/components/ui`에 추가.
- **HTML → 이미지 변환:** `html2canvas`
- **QR 코드 생성:** `qrcode.react`

---

## 3. 컴포넌트 설계 및 모듈화 계획

1.  **`src/components/ui` (shadcn/ui 제공):**
    *   **역할:** `Button`, `Input`, `Card` 등 앱의 기반이 되는 UI 요소.
    *   **설계:** `shadcn/ui`에 의해 생성된 코드를 그대로 사용하거나 필요시 커스터마이징.

2.  **`src/components/templates` (디자인 도면):**
    *   **역할:** 명함의 최종 디자인을 보여주는 순수 프레젠테이셔널 컴포넌트.
    *   **설계:** `props`로 명함 데이터를 받아 `shadcn/ui`의 `Card` 컴포넌트 등을 조합하여 디자인을 완성.

3.  **`src/components/business-card` (기능 조립):**
    *   **역할:** 명함 제작 기능의 핵심 상호작용을 담당.
    *   **`CardEditor.tsx`:** `Input` 컴포넌트들을 사용해 사용자 입력을 받고, `Button`으로 템플릿을 변경하는 등 상태 변경을 처리.
    *   **`CardPreview.tsx`:** 현재 상태(`state`)에 따라 적절한 템플릿 컴포넌트를 동적으로 렌더링.

4.  **`src/app/page.tsx` (최종 지휘자):**
    *   **역할:** 전역 상태 관리, 데이터와 이벤트 핸들러를 하위 컴포넌트에 전달.

---

## 4. 기능 및 UX 확장 계획

(이전 계획과 동일)

#### 기본 (MVP)
- 실시간 미리보기, 템플릿 선택, PNG 다운로드, 반응형 디자인

#### 심화 (고도화)
- 색상 변경, 폰트 변경, 로고 이미지 업로드, 레이아웃 변경

---

## 5. 디자인 컨셉

*   **전체적인 미학:** 모던하고 깔끔한 미니멀리즘
*   **주력 스타일:** 미니멀리즘 (Minimalism) + 플랫 디자인 (Flat Design)
    *   사용자가 기능에 집중할 수 있는 깨끗하고 정돈된 인터페이스를 제공합니다.
*   **포인트 스타일:** 글래스모피즘 (Glassmorphism)
    *   미리보기 영역이나 특정 UI 요소에 적용하여 깊이감과 세련미를 더합니다.
*   **옵션:** 다크 모드 중심 (Dark UI)
    *   사용자에게 다크 모드 선택 옵션을 제공하여 사용성을 높입니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mrbaeksang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
