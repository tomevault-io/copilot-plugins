## casa-golden

> 에어비앤비 숙소(Casa Golden, 홍대입구역 인근) 소개용 원페이지 웹사이트. React 19 + TypeScript + Vite. GitHub Pages(`joony.github.io/casa-golden/` 또는 `casagolden2026-afk.github.io/casa-golden/`)에 정적 배포.

# Casa Golden — 프로젝트 개요

에어비앤비 숙소(Casa Golden, 홍대입구역 인근) 소개용 원페이지 웹사이트. React 19 + TypeScript + Vite. GitHub Pages(`joony.github.io/casa-golden/` 또는 `casagolden2026-afk.github.io/casa-golden/`)에 정적 배포.

## 스택
- React 19, TypeScript, Vite 7
- `react-i18next` — 7개 언어 다국어 (ko, en, ja, zh-CN, zh-TW, id, es)
- `react-scroll` — 헤더 내비게이션 스무스 스크롤
- `yet-another-react-lightbox` — 방/시설 사진 갤러리 라이트박스
- `react-leaflet` + `leaflet` — 위치 안내 지도 모달 (OpenStreetMap 타일)

## 구조
- `src/App.tsx` — 전체 페이지 조립 (Header, HeroSection, Location 섹션, Facilities/Rooms 섹션, Footer, Lightbox). 스크롤 오프셋/지속시간 상수 상단에 정의.
- `src/components/` — HeroSection(자동 슬라이드+드래그 캐러셀), RoomGallery/GalleryImage(IntersectionObserver 기반 lazy loading), MapModal(Leaflet 지도+범례+구글맵 링크), VideoModal(언어별 "오시는 길" 영상), LanguageModal, LanguageButton, GuestGuideButton(언어별 PDF 가이드 오픈)
- `src/data/imageData.ts` — 히어로/각 방(room1, room2, living, kitchen, bathroom, utility, outdoor) 이미지 목록. 이미지 추가/순서 변경은 여기만 수정하면 됨.
- `src/data/guestGuidePdfMap.ts` — 언어 코드 → 게스트 가이드 PDF 파일명 매핑 (`public/guide/`).
- `src/locales/*.json` — 언어별 번역 텍스트. 새 텍스트 추가 시 7개 파일 모두 동기화 필요 (ko가 원본, 나머지는 번역).
- `src/i18n.ts` — 브라우저 언어 자동 감지 + `localStorage('userLanguageSelected')`로 사용자 선택 유지. 중국어는 zh-TW(대만/홍콩/마카오) vs zh-CN 지역 분기 로직 있음.
- `public/image/`, `public/video/`, `public/guide/` — 실제 정적 자산. `dist/`는 빌드 산출물(gitignore 대상 아님 — 배포 워크플로우가 `npm run build`로 재생성하므로 수동 편집 불필요).

## 배포
`.github/workflows/deploy.yml` — main 브랜치 push 시 자동으로 `npm ci && npm run build` 후 GitHub Pages에 배포. `vite.config.ts`의 `base: '/casa-golden/'`와 경로 일치 필요.

## 알아두면 좋은 점
- 이미지 경로는 절대 URL이 아니라 `image/xxx.webp` 형태 (Vite `base` 설정에 의해 자동으로 prefix됨). `import.meta.env.BASE_URL`을 쓰는 곳(guestGuidePdfMap, MapModal의 구글맵 아이콘)도 있음 — 새 정적 자산 참조 시 이 패턴을 따를 것.
- `MapModal.tsx`의 `LOCATIONS_CONST` 중 `busStop` 좌표에 "향후 수정 필요" 주석 있음 — 실제 위치 확인 후 업데이트 필요할 수 있음.
- Hero 자동 슬라이드 간격(`AUTO_SCROLL_INTERVAL`)은 과거 커밋에서 20s→10s→7s로 여러 번 조정된 이력 있음 (사용자가 UX 튜닝 중).
- 커밋 메시지 컨벤션: `[feat]`, `[fix]`, `[refactor]` 접두사 사용.
- 새 언어 텍스트/섹션 추가 시 `src/locales/*.json` 7개 파일 + `guestGuidePdfMap.ts` + `VideoModal.tsx`의 `videoUrlMap`까지 함께 확인.

---
> Source: [casagolden2026-afk/casa-golden](https://github.com/casagolden2026-afk/casa-golden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
