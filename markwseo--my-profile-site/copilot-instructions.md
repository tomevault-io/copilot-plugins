## my-profile-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 언어 및 커뮤니케이션 규칙

### 기본 응답 언어
- **모든 응답**: 한국어로 진행
- 기술 용어도 한국어로 설명

### 코드 작성 규칙
- **코드 주석**: 한국어로 작성
- **변수명/함수명**: 영어 (코딩 표준 준수)
- **클래스명/ID**: 영어 케밥-케이스 (예: `hero-section`, `btn-primary`)

### 깃 커밋 규칙
- **커밋 메시지**: 한국어로 작성
- **포맷**: `[유형] 설명` (예: `[추가] 네비게이션 바 구현`)
  - 유형: 추가, 수정, 삭제, 리팩토링, 스타일, 문서
  
### 문서화
- **README.md**: 한국어
- **주석/문서**: 한국어
- **이슈/PR 설명**: 한국어

---

## 프로젝트 개요

**프로젝트명**: 개발자 웹 포트폴리오 이력서

**기술 스택**:
- HTML5 (시맨틱 마크업)
- CSS3 / Tailwind CSS (반응형 디자인)
- JavaScript (ES6+, 인터랙션)
- 빌드 도구: Tailwind CSS CLI (필요시)

**프로젝트 목표**: 개인 포트폴리오 웹사이트를 통해 개발 경력과 기술을 전시

---

## 프로젝트 구조

```
web-resume/
├── index.html                 # 메인 페이지
├── styles/
│   ├── main.css              # 커스텀 CSS
│   └── tailwind.config.js    # Tailwind 설정
├── js/
│   ├── main.js               # 메인 스크립트
│   └── smooth-scroll.js      # 스무스 스크롤 기능
├── assets/
│   ├── images/               # 이미지 파일
│   ├── icons/                # SVG 아이콘
│   └── cv.pdf                # 이력서 PDF
├── CLAUDE.md                 # Claude 가이드
├── ROADMAP.md                # 개발 로드맵
└── README.md                 # 프로젝트 소개
```

---

## 개발 가이드

### 초기 설정

1. **Tailwind CSS 설치** (필요한 경우)
   ```bash
   npm install -D tailwindcss
   npx tailwindcss init
   ```

2. **Tailwind CSS 빌드**
   ```bash
   npx tailwindcss -i styles/main.css -o dist/output.css --watch
   ```

3. **로컬 개발 서버 실행**
   ```bash
   # Python 내장 서버 (권장)
   python -m http.server 8000
   
   # Node.js http-server (설치 필요)
   npx http-server
   ```

### 주요 개발 작업

#### 페이지 섹션 추가
- `index.html`에 새로운 `<section>`을 추가
- 해당 스타일을 `styles/main.css`에 작성
- 필요시 `js/main.js`에 인터랙션 로직 추가

#### Tailwind CSS 사용
- 유틸리티 클래스 우선 사용: `class="flex items-center justify-between"`
- 커스텀 스타일은 `@apply`를 사용하여 `main.css`에 작성
- 반응형: `sm:`, `md:`, `lg:`, `xl:` 프리픽스 사용

#### JavaScript 기능
- DOM 조작: `querySelector()`, `addEventListener()` 사용
- 이벤트: 클릭, 스크롤, 호버 등
- 애니메이션: Intersection Observer API, 트랜지션 활용

### 성능 및 접근성

- **이미지 최적화**: WebP 포맷 사용, 적절한 해상도
- **접근성 (A11y)**:
  - 시맨틱 HTML 사용
  - `alt` 텍스트 필수
  - 색상 대비 WCAG AA 표준 준수
  - 키보드 네비게이션 지원
- **모바일 반응형**: 모든 디바이스에서 테스트

### 배포

- **GitHub Pages**: 저장소 설정에서 활성화
- **Vercel/Netlify**: 간단한 배포 (권장)
  ```bash
  npm install -g vercel
  vercel
  ```

---

## 아키텍처 결정사항

### 왜 Tailwind CSS인가?
- 빠른 프로토타이핑
- 일관된 디자인 시스템
- 번들 크기 최소화 (프로덕션 빌드)
- CSS 작성 시간 단축

### 왜 Vanilla JavaScript인가?
- 포트폴리오 크기: 프레임워크 불필요
- 빠른 로딩 속도
- 학습 가치: 순수 JavaScript 능력 시연

### 싱글 페이지 vs 멀티 페이지
- **선택**: 싱글 페이지 (index.html만 사용)
- **이유**: 스크롤 기반 네비게이션, 빠른 로딩, 배포 간단

---

## 일반적인 작업 흐름

1. **기능 개발**
   - ROADMAP.md에서 해당 Phase 확인
   - HTML 구조 먼저 작성
   - Tailwind 클래스 적용
   - JavaScript 인터랙션 추가

2. **테스트**
   - 로컬 서버에서 브라우저로 확인
   - 반응형 테스트 (DevTools 사용)
   - 크로스 브라우저 확인

3. **커밋**
   ```bash
   git add .
   git commit -m "[유형] 간단한 설명"
   ```

4. **배포**
   - main 브랜치에 병합
   - 배포 서비스 자동 빌드 (GitHub Pages, Vercel 등)

---

## 참고 자료

- [ROADMAP.md](./ROADMAP.md) - 개발 로드맵 및 체크리스트
- [Tailwind CSS 공식 문서](https://tailwindcss.com/docs)
- [MDN Web Docs - HTML](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [MDN Web Docs - JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

---

## 주의사항

- ✅ 모든 이미지는 최적화 필수
- ✅ 접근성 검증 (WAVE, axe DevTools)
- ✅ 모바일 우선 설계 (Mobile-first approach)
- ✅ 성능: Lighthouse 점수 85점 이상 목표
- ❌ 외부 CDN에 의존하지 않기 (캐싱 목적 제외)
- ❌ 과도한 JavaScript 사용 금지

---
> Source: [markwseo/my-profile-site](https://github.com/markwseo/my-profile-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
