## ledeuxions-site

> LeDeuxions의 개인 웹사이트이자 웹 유틸리티 모음 사이트. 브라우저 기반 도구들(PDF 편집, 도장 추출기, 미디어 변환기, HEIC→JPG 변환기)과 포트폴리오/연락처 페이지를 제공한다. 한국어/영어 이중 언어를 지원하며, 브라우저 언어 설정에 따라 자동 리다이렉트된다.

# CLAUDE.md - LeDeuxions-Site AI 어시스턴트 가이드

## 프로젝트 개요

LeDeuxions의 개인 웹사이트이자 웹 유틸리티 모음 사이트. 브라우저 기반 도구들(PDF 편집, 도장 추출기, 미디어 변환기, HEIC→JPG 변환기)과 포트폴리오/연락처 페이지를 제공한다. 한국어/영어 이중 언어를 지원하며, 브라우저 언어 설정에 따라 자동 리다이렉트된다.

## 기술 스택

| 구분 | 기술 |
|------|------|
| **프론트엔드** | 순수 HTML5, CSS3, JavaScript (프레임워크 없음, NPM 의존성 없음) |
| **빌드** | 빌드 불필요 - 정적 파일 그대로 서빙 |
| **서버리스** | Cloudflare Workers (`_worker.js` 파일로 API 라우팅) |
| **백엔드 (선택)** | Flask/Python API `192.168.0.12:5000` (도장 추출용) |
| **호스팅** | 정적 파일 호스팅 (Cloudflare Pages / GitHub Pages) |
| **수익화** | Google AdSense |
| **분석** | Google Analytics |

## 저장소 구조

```
/
├── index.html                      # 메인 랜딩 페이지
├── ad-transition.html              # 광고 전환 페이지 (카운트다운)
├── ads.txt                         # AdSense 게시자 인증
├── robots.txt / sitemap.xml        # SEO 파일
├── review_report.md                # 내부 QA/감사 체크리스트
│
├── web-projects/                   # 주요 유틸리티 앱
│   ├── index.html                  # 프로젝트 목록 페이지
│   ├── pdf300/                     # PDF 편집 도구
│   ├── stamp/                      # 도장 PNG 추출 도구 (v1.1.0)
│   │   ├── js/stamp.js            # 도장 추출 핵심 로직
│   │   ├── js/pdf-stamp.js        # PDF 도장 삽입
│   │   ├── css/style.css           # 반디집 스타일 디자인 시스템
│   │   ├── _worker.js              # Cloudflare Worker API 라우팅
│   │   └── stamp-pages/            # 도장 도구 복제본 (동기화 필요)
│   ├── freecomfortlab/             # 미디어 변환 도구 모음
│   │   ├── *.html                  # 오디오/비디오 변환 페이지
│   │   ├── _worker.js              # Cloudflare Worker
│   │   └── en/                     # 영어 버전
│   └── iphone-heic/               # HEIC→JPG 변환기 (PWA)
│       ├── manifest.json           # PWA 설정
│       ├── ko/ & en/               # 언어별 버전
│       └── assets/                 # 스타일, 스크립트, 아이콘
│
├── applications/                   # 설치형 앱 버전
│   └── iphoneheic/                 # HEIC 변환기 (복제본, 동기화 필요)
│
├── contact/                        # 연락처 (Google Calendar 임베드)
├── privacy/                        # 개인정보처리방침
├── terms/                          # 이용약관
├── history/                        # 작업 이력
└── work/                           # 프로젝트 포트폴리오
```

## 개발 명령어

```bash
# 로컬 개발 서버 (빌드 불필요)
python -m http.server 8000
# 또는
npx http-server .
# 또는 VS Code Live Server 확장 사용

# Git 작업
git add <파일명>
git commit -m "변경 설명"
git push origin main
```

**package.json 없음**, 빌드 파이프라인 없음, 테스트 러너 없음, 린터 설정 없음. 변경사항은 브라우저에서 수동 테스트.

## 코드 규칙

### HTML
- 각 페이지는 인라인 `<style>` 블록으로 자체 완결 (stamp 프로젝트만 외부 CSS 사용)
- 모바일 우선 반응형 디자인 (`@media` 쿼리 사용)
- 시맨틱 HTML5 + Open Graph 메타 태그 (SEO)
- Google AdSense 스크립트 및 광고 유닛이 페이지 내에 직접 삽입
- 네이버/구글 사이트 인증 메타 태그 포함

### CSS
- CSS 커스텀 속성(변수)으로 테마 관리:
  ```css
  --primary: #0078d7;
  --secondary: #2d8659;
  --accent: #ff8c00;
  --bg-main: #f5f5f5;
  ```
- 시스템 폰트: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, ...`
- 그림자 단계: `--shadow-sm`, `--shadow-md`, `--shadow-lg`
- `box-sizing: border-box` 전역 적용
- CSS Grid, Flexbox 레이아웃
- stamp 프로젝트: 반디집 스타일의 깔끔한 디자인 시스템

### JavaScript
- 100% 순수 JS (프레임워크 없음, 트랜스파일 없음)
- 클라이언트 측 처리 우선 (Canvas API, FileReader API)
- 페이지 간 데이터 전달: `sessionStorage`
- HTTP 요청: `fetch` API → Cloudflare Workers
- 파일 업로드: 드래그 앤 드롭 지원
- 언어 감지 패턴:
  ```javascript
  const userLang = navigator.language || navigator.userLanguage;
  if (!userLang.includes('ko')) {
      window.location.href = 'en/index.html';
  }
  ```

### 이중 언어 지원
- 한국어가 기본 언어
- 영어 버전은 `en/` 하위 디렉토리에 위치
- `navigator.language` 기반 자동 리다이렉트
- 각 언어 버전은 별도 HTML 파일 (동적 i18n 아님)

## 핵심 알고리즘

### 도장 배경 제거 (`stamp/js/stamp.js`)
- 이미지 4개 모서리에서 배경색 샘플링
- 유클리드 거리 계산으로 색상 매칭
- 알파 블렌딩으로 부드러운 가장자리 처리
- 밝기 필터링 (임계값 > 180)으로 흰 배경 제거

### HEIC 변환 (`iphone-heic/assets/app.js`)
- 클라이언트 측 처리 + 선택적 API 폴백
- PWA 지원 (manifest, 아이콘 32px~512px)

## Cloudflare Workers

`_worker.js` 파일이 `/api/*` 요청을 백엔드 서버로 라우팅. Cloudflare Pages에 정적 파일과 함께 배포됨.

## 주의사항

- **자동화된 테스트 없음** - 모든 테스트는 브라우저에서 수동 수행
- **린터/포매터 없음** - 코드는 수동 포맷팅
- **CI/CD 파이프라인 없음** - 배포는 수동
- **중복 코드 주의**: `stamp/`과 `stamp/stamp-pages/`는 동일한 코드 (수정 시 양쪽 동기화 필수)
- **중복 코드 주의**: `web-projects/iphone-heic/`와 `applications/iphoneheic/`도 동일
- **프라이버시 우선** - 대부분의 도구가 파일을 클라이언트에서만 처리
- `review_report.md`에서 광고 배치 준수 여부 및 QA 항목 추적
- 저작권: © 2005–2026 LeDeuxions

## 자주 하는 작업

### 새 웹 도구 추가
1. `web-projects/` 아래에 디렉토리 생성
2. 인라인 스타일 또는 외부 CSS가 포함된 `index.html` 추가
3. 기존 디자인 패턴 준수 (반응형, 이중 언어, 광고 배치)
4. `web-projects/index.html`에 링크 추가
5. `sitemap.xml`에 새 페이지 추가

### 도장 도구 수정
- `stamp/`과 `stamp/stamp-pages/` 양쪽 모두 수정하여 동기화 유지
- 핵심 로직: `js/stamp.js`, `js/pdf-stamp.js`
- 스타일: `css/style.css`

### 광고/분석 업데이트
- AdSense 클라이언트 ID: `ca-pub-2824624880449066`
- 광고 유닛은 HTML 페이지에 인라인으로 삽입
- 광고 배치 감사 상태: `review_report.md` 참조

---
> Source: [vis555540-ops/ledeuxions-site](https://github.com/vis555540-ops/ledeuxions-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
