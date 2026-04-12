## web-publish-reformedts

> Claude Code가 이 프로젝트를 열 때마다 자동으로 읽는 파일입니다.

# CLAUDE.md — 프로젝트 컨텍스트

Claude Code가 이 프로젝트를 열 때마다 자동으로 읽는 파일입니다.

---

## 1. 보일러플레이트 구조 (템플릿 공통)

이 프로젝트는 **NUGABOX 웹 퍼블리싱 보일러플레이트** 기반입니다.

### 기술 스택
- **Bootstrap 5.3** — 로컬 파일 사용 (CDN 절대 사용 금지)
- **Bootstrap Icons** — `<i class="bi bi-{아이콘명}">`
- **Font Awesome Pro** — `<i class="fa-solid fa-{아이콘명}">` 등
- **Pretendard** — 로컬, 한국어 최적화 subset
- **Swiper** — 슬라이더 라이브러리 (로컬)

### 디렉토리 구조
```
src/
├── assets/
│   ├── css/        style.css(전역), main.css(메인전용), sub.css(서브전용)
│   ├── js/         script.js(전역), main.js(메인전용), sub.js(서브전용)
│   ├── images/
│   └── fonts/
├── include/
│   ├── header.html     (id="site-header", data-include로 로드)
│   ├── footer.html     (id="site-footer")
│   └── topmenu.html    (header 안에서 data-include로 로드)
├── index.html          (메인 페이지)
├── sub/                (소개·안내 서브 페이지들)
└── board/              (게시판 페이지들)
```

### 작업 규칙
1. Bootstrap 유틸리티 클래스를 최대한 활용
2. 스타일 추가 시 `<style>` 태그나 인라인 스타일 사용 금지 — 반드시 CSS 파일에 작성
3. CDN 링크 절대 사용 금지
4. 외부 JS 라이브러리 필요 시 npm 설치 방법 별도 안내
5. header, footer, `<head>` 는 별도 파일이므로 페이지 작업 시 `<main>` 내부만 수정
6. 페이지 전용 JS는 `script.js`가 아닌 해당 페이지명 JS 파일에 작성

### CSS 변수 (style.css 정의)
```css
/* Brand Colors */
--primary-navy: #000613
--primary-container: #001f3f
--secondary-gold: #775a19
--accent-gold: #e9c176
--gold-light: #fed488
--soft-gold: #ffdea5

/* Surface */
--surface-bg: #f8f9fa
--surface-container: #edeeef
--surface-container-low: #f3f4f5

/* Text */
--text-main: #191c1d
--text-muted: #43474e
--border-color: #e1e3e4

/* Misc */
--spacing-section: 80px
--font-base: "Pretendard", "Apple SD Gothic Neo", "Noto Sans KR", sans-serif
--radius-md: 8px
--transition-base: 0.2s ease
```

### 주요 유틸리티 클래스 (style.css 정의)
| 클래스 | 설명 |
|---|---|
| `.section` | 상하 padding: var(--spacing-section) |
| `.section-sm` | 상하 padding 절반 |
| `.section-soft` | 배경색 var(--surface-bg) |
| `.section-dark` | 배경색 var(--primary-navy), 텍스트 밝게 |
| `.section-title` | 섹션 h2 제목 스타일 |
| `.section-subtitle` | 섹션 설명 텍스트 스타일 |
| `.btn-gold` | 골드 배경 버튼 |
| `.btn-outline-white` | 흰색 아웃라인 버튼 (히어로용) |
| `.badge-navy` / `.badge-gold` | 뱃지 컴포넌트 |
| `.text-primary-navy` / `.text-secondary-gold` | 텍스트 색상 유틸리티 |

---

## 2. 이 프로젝트 고유 정보 (개혁신학연구회)

### 사이트 개요
- **사이트명**: 개혁신학연구회
- **영문 슬로건**: The Modern Archivist
- **목적**: 서철원 박사의 신학적 유산을 계승하는 개혁주의 신학 연구 단체 공식 웹사이트
- **톤앤매너**: 신뢰감 있는 학술 기관 — Navy(딥 다크) + Gold(권위·학술) 2색 기조

### 사이트 메뉴 구조
```
연구회 소개
  ├── 연구회 소개        sub/about.html
  ├── 서철원 박사 소개   sub/dr-seo.html
  ├── 연혁               sub/history.html
  ├── 섬기는 분들        sub/members.html
  ├── 회칙 및 규정       sub/rules.html
  └── 후원 안내          sub/support.html

교육 및 세미나
  ├── 세미나 안내        board/seminar-list.html
  ├── 논문 및 발제문     board/papers-list.html
  └── 자료실             board/resources-list.html

연구회 소식
  ├── 공지사항           board/notice-list.html
  ├── 언론보도           board/press-list.html
  ├── 회원동정           board/member-news-list.html
  └── 앨범               board/album-list.html
```

### 기타 페이지
```
sub/privacy.html    개인정보처리방침
sub/terms.html      이용약관
sub/sitemap.html    사이트맵

board/general-view.html   일반 게시글 뷰
board/general-write.html  일반 게시글 작성
board/media-view.html     미디어 게시글 뷰
board/media-write.html    미디어 게시글 작성
```

### 추가된 라이브러리 (보일러플레이트 기본 외 추가분)
- **Swiper** (`swiper-bundle.min.css`, `swiper-bundle.min.js`) — 앨범 슬라이드 등에 사용

### 컬러 기조 요약
- **Navy** (`--primary-navy: #000613`, `--primary-container: #001f3f`): 헤더, 다크 섹션, 강조 텍스트
- **Gold** (`--secondary-gold: #775a19`, `--accent-gold: #e9c176`): CTA 버튼, 액션 링크, 포인트

### 명령어
```bash
npm run dev      # 개발 서버 (http://localhost:3000)
npm run build    # dist 빌드
npm run setup    # Bootstrap/Pretendard 파일 재복사
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nugaBox)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nugaBox)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
