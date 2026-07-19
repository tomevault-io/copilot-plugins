## nextpartner-signin-edit

> **넥스트파트너** 시니어 구직 플랫폼의 반응형 웹사이트.

# 넥스트파트너 홈 — CLAUDE.md

## 프로젝트 개요

**넥스트파트너** 시니어 구직 플랫폼의 반응형 웹사이트.  
주 사용자는 **50세 이상 시니어**이며, 모든 디자인·개발 결정은 시니어 UX 원칙을 최우선으로 한다.

---

## 기술 스택

| 항목 | 내용 |
|------|------|
| CSS 프레임워크 | **Tailwind CSS v3** (v4 아님) |
| 빌드 | Tailwind CLI (`npx tailwindcss`) |
| 폰트 | Pretendard Variable (CDN) |
| 언어 | 한국어 UI |
| 아이콘 | 인라인 SVG (혼용) | **Lucide Icons** |

---

## 디자인 시스템 경로

```
C:\Users\Administrator\Desktop\nextpartnerds-tw\
├── tailwind.config.js     ← 토큰 정의 (색상·간격·반경 등)
├── src\index.css          ← 컴포넌트 클래스 (@layer components)
└── sections\
    ├── foundation\        ← 색상·타이포그래피
    └── components\        ← 버튼·텍스트필드·칩 등 모든 컴포넌트 HTML
```

---

## 핵심 제약 — 디자인 시스템 준수

### 1. 컴포넌트 코드는 반드시 그대로 가져온다

`nextpartnerds-tw`에서 만들어진 HTML 스니펫과 CSS 클래스는 **절대 가공하지 않는다**.  
복사해서 그대로 사용하여 디자인 일관성을 유지한다.

- `np-btn`, `np-chip`, `np-job-card` 등 `np-` 접두사 클래스는 수정 금지
- `src/index.css`의 `@apply` 정의를 임의로 변경하지 않는다
- 디자인 시스템의 `tailwind.config.js` 토큰 값을 이 프로젝트에서 재정의하지 않는다

### 2. 예외 — 일회성 요소

디자인 시스템에 없고, 사이트 전체에서 한두 번만 쓰이는 요소는 Tailwind 유틸리티 클래스를 직접 사용해도 된다.  
단, 같은 패턴이 2회 이상 반복될 조짐이 보이면 디자인 시스템에 추가를 검토한다.

### 3. 이 프로젝트의 `tailwind.config.js`는 디자인 시스템과 동일하게 유지

```js
// nextpatner-home/tailwind.config.js
module.exports = {
  content: ['./**/*.html', './src/**/*.js'],
  theme: {
    extend: {
      // nextpartnerds-tw/tailwind.config.js 와 동일한 토큰 사용
      colors: { /* 동일 */ },
      spacing: { /* 동일 */ },
      borderRadius: { /* 동일 */ },
      fontFamily: { /* 동일 */ },
      maxWidth: { content: '1212px' },
    },
  },
  plugins: [],
}
```

---

## 디자인 토큰 요약

### 색상

| 그룹 | 주요 값 |
|------|---------|
| `primary` | 500: `#ff7f65` / 400: `#ff9a7a` / 100: `#ffe3dc` / 50: `#fff4f1` |
| `neutral` | 900: `#1a1a1a` … 0: `#ffffff` |
| `warm-grey` | 300–25 |
| semantic | `success` / `warning` / `error` / `info` |

### 간격 (spacing)

`4px(1)` · `8px(2)` · `12px(3)` · `16px(4)` · `20px(5)` · `24px(6)` · `32px(8)` · `40px(10)` · `48px(12)` · `64px(16)`

### 반경 (borderRadius)

`button: 8px` · `card: 8px` · `search: 24px` · `badge: 4px` · `pill: 999px`

---

## 사용 가능한 컴포넌트 클래스 목록

아래 클래스들은 `nextpartnerds-tw/src/index.css`에 이미 정의되어 있다.  
이 프로젝트에서는 동일한 CSS를 그대로 복사해서 사용한다.

### 버튼
- `np-btn` + `np-btn-{sm|md|lg}` + `np-btn-{primary|secondary|ghost|grey|grey-stroke}`
- `np-btn-icon` + `np-btn-icon-{sm|md|lg}` + `np-btn-icon-{primary|secondary|ghost|grey|grey-stroke}`
- `np-fab`

### 텍스트 필드
- `np-textfield-wrap` · `np-textfield-label` · `np-textfield`
- `np-textfield-error` · `np-textfield-hint` · `np-textfield-hint-error`

### 입력 스테퍼
- `np-stepper` + `np-stepper-{sm|md}` · `np-stepper-btn` · `np-stepper-value`
- `np-stepper-wrap` · `np-stepper-label` · `np-stepper-hint`

### 선택 컨트롤
- `np-checkbox` · `np-checkbox-input` · `np-checkbox-label`
- `np-radio` · `np-radio-input` · `np-radio-label`
- `np-switch` · `np-switch-input` + `np-switch-{sm|md}` · `np-switch-label`

### 드롭다운
- `details.np-dropdown` · `np-dropdown-chevron` · `np-dropdown-menu`
- `np-dropdown-item` (`.selected` / `.disabled`) · `np-dropdown-divider`

### 검색
- `np-search` · `np-search-input` · `np-search-icon` · `np-search-clear`
- `np-search-view` · `np-search-section-label` · `np-search-item`
- `np-search-item-keyword` · `np-search-divider`

### 칩 · 뱃지 · 구분선
- `np-chip` · `np-chip-active`
- `np-badge` + `np-badge-{primary|neutral|success|warning|error|info|gradient}`
- `np-divider` · `np-divider-thick`

### 내비게이션
- `np-header` · `np-header-inner` · `np-header-nav` · `np-header-actions`
- `np-header-menu-btn` · `np-nav-link` · `np-nav-link-job` · `np-header-divider`
- `np-app-bar` · `np-app-bar-title` · `np-app-bar-title-center` · `np-app-bar-btn`
- `np-bottom-nav` · `np-bottom-nav-item` (`.active`) · `np-bottom-nav-label` · `np-bottom-nav-badge`

### 탭 · 세그먼티드 컨트롤
- `np-tab-bar` · `np-tab-item` (`.active` / `.disabled`)
- `np-segmented` · `np-segmented-full` · `np-segmented-item` (`.active` / `.disabled`)

### 카드 · 토스트 · 푸터
- `np-job-card` · `np-job-card-recommended` · `np-job-card-badge`
- `np-job-card-{title|company|meta|hours|pay|pay-note|urgency|deadline|bookmark|reasons|reason}`
- `np-toast` · `np-toast-action` · `np-toast-close` · `np-toast-wrap`
- `np-footer` · `np-footer-inner` · `np-footer-{tagline|info|section-title|link}`

---

## 반응형 브레이크포인트

Tailwind 기본 브레이크포인트를 사용한다. **데스크탑 콘텐츠 최대 너비는 1212px**로 고정하고, 태블릿·모바일은 유동 너비로 처리한다.

| 접두사 | 범위 | 환경 | 설명 |
|--------|------|------|------|
| (기본) | 0 ~ 767px | 모바일 | 단일 컬럼, 좌우 패딩 `px-4` |
| `md:` | 768px ~ 1023px | 태블릿 | 2컬럼 허용, 좌우 패딩 `px-6` |
| `lg:` | 1024px 이상 | 데스크탑 | 3컬럼 허용, 좌우 패딩 `px-8`, 콘텐츠 최대 너비 1212px |

### 콘텐츠 컨테이너 패턴

모든 섹션 콘텐츠는 아래 래퍼를 공통으로 사용한다.

```html
<div class="w-full max-w-content mx-auto px-4 md:px-6 lg:px-8">
  <!-- 콘텐츠 -->
</div>
```

- `max-w-content` = 1212px (tailwind.config.js에 정의됨)
- 1212px보다 좁은 화면에서는 `w-full`로 전체 너비 사용
- 좌우 패딩은 브레이크포인트별로 달라짐 (모바일 16px → 태블릿 24px → 데스크탑 32px)

### 레이아웃 그리드 규칙

| 요소 | 모바일 | 태블릿(md) | 데스크탑(lg) |
|------|--------|-----------|-------------|
| 잡 카드 그리드 | 1열 | 2열 | 3열 |
| 기사 카드 그리드 | 2열 | 3열 | 4열 |
| 구직 정보 카드 | 1열 | 2열 | 3열 |
| 히어로 (카드+배너) | 세로 스택 | 가로 분리 | 가로 분리 |
| 공지+고객센터 | 세로 스택 | 가로 분리 | 가로 분리 |
| 이력서 배너 CTA | 세로 스택 | 가로 분리 | 가로 분리 |
| 헤더 내비게이션 | 숨김 (햄버거) | 표시 | 표시 |
| 하단 내비게이션 바 | 표시 (fixed) | 숨김 | 숨김 |

### 섹션 수직 패딩

| 구분 | 모바일 | 태블릿+ |
|------|--------|---------|
| 일반 콘텐츠 섹션 | `py-10` | `py-16` |
| 히어로 섹션 | `py-8` | `py-12` |
| 배너/이벤트 섹션 | `py-12` | `py-20` |
| 필터 조정 배너 | `py-8` | `py-10` |

### 타이포그래피 스케일

| 용도 | 모바일 | 데스크탑 |
|------|--------|---------|
| 페이지 대제목(히어로) | `text-2xl` | `text-4xl` |
| 섹션 타이틀 | `text-2xl` | `text-3xl` |
| 카드 제목 | `text-xl` | `text-xl` (고정) |
| 본문 | `text-base` | `text-base` (고정) |

---

## 시니어 UX 원칙

### 1. 간결한 정보 구조
- 한 화면에 핵심 정보 1~2개만 노출
- CTA는 화면당 최대 1개

### 2. 명료한 문구
- 짧고 직접적인 단어 사용 (전문 용어 지양)
- 버튼·레이블은 동작을 명확히 표현 ("확인" → "지원하기")

### 3. 시각적 단순함
- 기본 팔레트: **neutral 계열 + primary 강조** 2가지
- semantic 색상은 명확한 피드백 상황에만 사용
- 목적 없는 장식 금지

### 4. 충분한 터치 영역
- 버튼 최소 높이: MD(48px) 이상, 모바일은 LG(56px) 권장
- 탭 가능 요소 간 최소 간격: 8px

### 5. 읽기 편한 타이포그래피
- 본문 최소 `text-base`(16px), 중요 정보 `text-lg`(18px) 이상
- 줄 간격 `leading-relaxed` 기본

---

## 개발 환경

```bash
# 의존성 설치
npm install

# 개발 (Tailwind watch)
npm run dev
# = npx tailwindcss -i ./src/index.css -o ./dist/index.css --watch

# 프로덕션 빌드
npm run build
```

`index.html` `<head>`:

```html
<!-- Pretendard 폰트 -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable.min.css" />
<!-- Tailwind 빌드 아웃풋 -->
<link rel="stylesheet" href="dist/index.css" />
```

---

## 페이지 공통 레이아웃 구조

> 레퍼런스: `C:\Users\Administrator\Desktop\nextpatner-home\index.html`
>
> 모든 페이지는 홈에서 이동했을 때 어색함이 없도록 아래 구조를 반드시 동일하게 유지한다.

### 1. `<head>` 스크립트 순서

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable.min.css" />
<link rel="stylesheet" href="dist/index.css" />
<script src="https://unpkg.com/lucide@latest"></script>  <!-- head에 위치 -->
```

- Lucide 스크립트는 반드시 **`<head>`** 안에 선언한다 (body 말미 금지).

### 2. `<body>` 클래스

```html
<body class="font-base text-neutral-900 bg-neutral-50 pb-16 md:pb-0">
```

- `pb-16 md:pb-0` — 하단 내비게이션 바 영역을 body 레벨에서 확보한다.
- `<main>` 태그 사용 금지 — 콘텐츠 래퍼는 `<div>`를 사용한다.

### 3. 헤더 구조

디자인 시스템(`nextpartnerds-tw/sections/components/header.html`) **Desktop — Logged In** 코드를 그대로 가져온다. `sticky top-0 z-50`과 모바일 햄버거 버튼(`np-header-menu-btn`)만 페이지에서 추가한다.

```html
<header class="np-header sticky top-0 z-50">
  <div class="np-header-inner">
    <a href="#" class="flex items-center gap-2 shrink-0">
      <svg width="32" height="32" viewBox="0 0 32 32" fill="none">
        <path d="M16 2L28.7 9.5V24.5L16 32L3.3 24.5V9.5L16 2Z" fill="#ff7f65"/>
        <path d="M11 10h2.5l5 12H16l-1.2-3H10l-1.2 3H6.5L11 10Zm2.8 7-1.8-4.5L10.2 17h3.6Z" fill="white"/>
      </svg>
      <span class="text-xl font-bold text-neutral-900 tracking-tight">NextPartner</span>
    </a>
    <nav class="np-header-nav">
      <a class="np-nav-link">서비스소개</a>
      <div class="np-header-divider"></div>
      <a class="np-nav-link-job">채용공고</a>   <!-- 페이지별 항목 변경 가능 -->
      <a class="np-nav-link-job">프로젝트</a>
      <a class="np-nav-link-job">이력서</a>
    </nav>
    <div class="np-header-actions">
      <div class="flex items-center gap-2">
        <div class="w-8 h-8 rounded-full bg-primary-100 overflow-hidden shrink-0">
          <img src="…" alt="프로필" class="w-full h-full object-cover" />
        </div>
        <span class="text-base text-neutral-600">이름님</span>
      </div>
      <button class="flex items-center justify-center w-10 h-10 text-neutral-400 rounded-button hover:bg-neutral-50 hover:text-neutral-800 transition-colors" aria-label="로그아웃">
        <i data-lucide="log-out" class="w-5 h-5"></i>
      </button>
    </div>
    <!-- 모바일 햄버거 — 페이지에서 추가 -->
    <button class="np-header-menu-btn" aria-label="메뉴">
      <i data-lucide="menu" class="w-6 h-6"></i>
    </button>
  </div>
</header>
```

- 헤더 `z-index`: **`z-50`** 고정.
- 내비게이션은 **단일 `np-header-nav`** 안에서 `np-header-divider`로 그룹을 구분한다. 별도 `<nav>` 분리 금지.
- 로그인 상태는 아바타 이미지 + 이름 텍스트 + 로그아웃 버튼 구조 (드롭다운 금지).

### 4. 하단 내비게이션 바

```html
<nav class="np-bottom-nav md:hidden fixed bottom-0 left-0 right-0 z-50">
  <div class="np-bottom-nav-item">
    <i data-lucide="home" style="width:24px;height:24px;"></i>
    <span class="np-bottom-nav-label">홈</span>
  </div>
  …
</nav>
```

- 클래스 선언 순서: `np-bottom-nav md:hidden fixed bottom-0 left-0 right-0 z-50`
- `z-index`: **`z-50`** 고정 (헤더와 동일).

### 5. Lucide 아이콘 크기 지정 방식

디자인 시스템 컴포넌트 코드(헤더 등)는 `class="w-X h-X"` 표기를 그대로 유지한다.  
페이지에서 직접 작성하는 아이콘은 `style` 속성으로 크기를 지정한다.

```html
<!-- ✅ 디자인 시스템 컴포넌트 코드 — class 그대로 유지 -->
<i data-lucide="log-out" class="w-5 h-5"></i>

<!-- ✅ 페이지 직접 작성 코드 — style 속성 사용 -->
<i data-lucide="chevron-right" style="width:16px;height:16px;"></i>

<!-- ❌ 페이지 직접 작성 시 Tailwind 클래스로 크기 지정 금지 -->
<i data-lucide="chevron-right" class="w-4 h-4"></i>
```

| 사용 위치 | 크기 |
|-----------|------|
| 헤더 액션 아이콘 (알림·로그아웃) | `22px` |
| 헤더 햄버거 | `24px` |
| 하단 내비게이션 | `24px` |
| 목록 항목 화살표 (`chevron-right`) | `16px` |
| 섹션 내 기능 아이콘 | `20px` |
| 강조형 피처 아이콘 | `26px` |

### 6. Lucide 초기화

```html
<!-- </body> 직전에 위치 -->
<script>
  lucide.createIcons({ nameAttr: 'data-lucide' });
</script>
```

- `nameAttr: 'data-lucide'` 옵션을 반드시 전달한다.

### 7. 푸터

모든 페이지에 `np-footer` 푸터를 포함한다. 하단 내비게이션 바 바로 위에 위치시킨다.

---

## 코드 스타일

- Tailwind 클래스 순서: **레이아웃 → 크기 → 간격 → 색상 → 타이포 → 기타**
- `np-` 컴포넌트 클래스를 먼저, Tailwind 유틸리티 클래스를 그 다음에 작성
- 주석은 WHY가 명확히 비자명한 경우에만 한 줄로 작성
- `!important` 금지 — 특이성 문제는 클래스 구조로 해결
- **Tailwind v3** 문법 사용 (`content`, `theme.extend`, `@layer`)

## 기획안

메인 페이지 UX 개선 기획안은 아래 경로에서 관리됩니다.

```
docs/ux-spec/nextpartner_nextpartner_ux_home_spec.md
```

---
> Source: [inhuiyang/nextpartner-signin-edit](https://github.com/inhuiyang/nextpartner-signin-edit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
