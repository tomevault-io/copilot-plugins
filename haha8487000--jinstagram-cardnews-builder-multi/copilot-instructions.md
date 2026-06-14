## jinstagram-cardnews-builder-multi

> > 메이TV 강의 워크숍용 노코드 카드뉴스 제작 웹앱. 정적 파일 + 클라이언트 사이드 PNG 캡처.

# 인스타 카드뉴스 빌더 — 개발 가이드

> 메이TV 강의 워크숍용 노코드 카드뉴스 제작 웹앱. 정적 파일 + 클라이언트 사이드 PNG 캡처.

## 🎯 프로젝트 목적

수강생이 코드 없이 1080×1350 카드뉴스 8장(cover + content×6 + cta)을 만들 수 있는 워크플로우 페이지.

## 🏗️ 아키텍처

```
[Browser]
   ├─ index.html         (DOM 구조 + 폼)
   ├─ styles.css         (UI 스타일)
   └─ app.js
       ├─ state          (전역 입력값 객체)
       ├─ renderCard()   (idx → cover/content/cta 분기)
       ├─ renderCover/Content/Cta()
       │     ├─ classic  레이아웃
       │     ├─ minimal  레이아웃
       │     └─ bold     레이아웃
       ├─ bindForm()     (input 이벤트 → state 갱신 → re-render)
       └─ downloadAll()  (8장을 html-to-image로 캡처 → JSZip)
```

서버 없음. 모든 로직이 브라우저에서 실행됨.

## 🔑 핵심 개념

### 1. State-driven 렌더링
모든 입력값은 `state` 객체에 저장. 입력 이벤트 → state 변경 → `updateAll()` → 현재 카드만 재렌더.

### 2. HTML 문자열 + 안전한 삽입
카드 디자인은 함수형 HTML 문자열로 생성. 모든 사용자 입력은 `escapeHtml()`로 escape 후 삽입. DOM 주입은 `Range.createContextualFragment` 기반 `setHtml()` 헬퍼 사용 (innerHTML 대체, 동일 효과 + XSS 안전).

### 3. 미리보기 vs 다운로드
- **미리보기**: `transform: scale(0.35)`로 1080×1350 카드를 380px 크기로 축소
- **다운로드**: 화면 밖 (top: -99999px) 임시 컨테이너에 1080×1350 원본 크기로 렌더 → html-to-image의 `pixelRatio: 2`로 2160×2700 retina PNG 캡처

### 4. 폰트 안정화
캡처 전 `await document.fonts.ready` + 200ms 대기. Pretendard Variable CDN 로딩 보장.

## 📐 카드 구성 (8장)

| 번호 | 타입 | 필드 |
|------|------|------|
| 1 | cover | badge, title, subtitle, highlight |
| 2~7 | content | number, subtitle, title, body |
| 8 | cta | title, subtitle, homepage, email, book |

## 🎨 레이아웃 (3종)

- **classic**: 원본 메이TV 톤. 그라데이션 + 큰 타이틀 + 골드 포인트
- **minimal**: 라인 + 여백 강조. 큰 숫자 액센트
- **bold**: 강한 대비. 컬러 블록 + 큰 숫자 배지

각 레이아웃은 `renderCover`, `renderContent`, `renderCta` 안에서 `layout` 파라미터로 분기.

## 🌈 컬러 시스템

- `colors.main` — 메인 (텍스트 기본)
- `colors.accent` — 포인트 (강조)
- `colors.bg` — 배경

프리셋 6종 + HEX 커스텀 입력. 변경 시 즉시 state 갱신.

## 🚧 알려진 한계

1. **html-to-image 폰트 임베딩**: CDN 폰트가 캡처 시 안 나타날 수 있음 — `document.fonts.ready` 대기로 해결. 그래도 깨지면 `pixelRatio: 1`로 낮추거나 `style` 변환 제거 시도.
2. **이모지 렌더링**: 시스템 이모지에 의존 → OS별 다를 수 있음. 통일하려면 Noto Color Emoji CDN 추가 필요.
3. **카드 수 고정**: 현재 8장 하드코딩 (`TOTAL_CARDS`). 가변으로 만들려면 본문 카드 추가/삭제 UI 필요.

## 🚀 개발 명령

```bash
# 로컬 미리보기 (정적 서버)
python3 -m http.server 8000
# → http://localhost:8000

# Vercel 배포
vercel --prod
```

## ✏️ 확장 아이디어 (v2)

- [ ] 본문 카드 가변 (4~10장)
- [ ] challenge / mission 카드 타입 추가 (원본 5종 템플릿)
- [ ] 저장/불러오기 (localStorage)
- [ ] 카드 JSON 내보내기/가져오기 (워크플로우 공유)
- [ ] 이미지 업로드 (로고, 인물 사진)
- [ ] AI 추천 — 주제만 입력하면 본문 자동 생성 (Claude API)

## 👤 운영자

김진수 (May 작가) · AICLab · 메이TV (@AICLab_TV)

---
> Source: [haha8487000/Jinstagram-cardnews-builder-multi](https://github.com/haha8487000/Jinstagram-cardnews-builder-multi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
