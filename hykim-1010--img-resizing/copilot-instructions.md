## img-resizing

> 웹 운영 기획자를 위한 브라우저 기반 이미지 리사이징 툴.

# IMG Resizer — Project Rules

## 프로젝트 개요
웹 운영 기획자를 위한 브라우저 기반 이미지 리사이징 툴.
모든 이미지 처리는 Canvas API로 브라우저 내에서 처리한다. 서버 통신 없음.

## 기술 스택
- React + Vite
- Tailwind CSS
- Canvas API (이미지 처리)
- localStorage (즐겨찾기 저장)

## 디렉토리 구조
src/
├── components/
│   ├── Uploader.jsx       # 드래그 앤 드롭 업로드 영역
│   ├── ResizeSettings.jsx # 리사이징 모드 및 설정 UI
│   ├── Preview.jsx        # 결과 미리보기 + 후처리 버튼
│   ├── Downloader.jsx     # 다운로드 버튼
│   └── Favorites.jsx      # 즐겨찾기 목록 및 관리
├── hooks/
│   ├── useImageResize.js  # Canvas 기반 리사이징 로직
│   └── useFavorites.js    # 즐겨찾기 CRUD + localStorage
├── App.jsx
└── main.jsx

## 커밋 메시지 규칙
- 형식: `[Step N] type: 설명`
- N은 AGENTS.md 개발 진행 순서 기준으로 해당 작업이 속한 단계 번호
- 예시: `[Step 3] feat: useImageResize 훅 구현 (모드 ①②③)`

## 코딩 규칙
- 컴포넌트는 함수형으로 작성, Props는 명시적으로 구조분해
- 스타일은 Tailwind CSS 클래스만 사용, 인라인 스타일 금지
- Canvas 처리 로직은 useImageResize 훅에만 집중, 컴포넌트에 직접 작성 금지
- 즐겨찾기 데이터는 useFavorites 훅을 통해서만 읽고 씀
- 토스트 메시지는 전역 상태로 관리

## 즐겨찾기 데이터 구조
{
  id: string,           // crypto.randomUUID()
  label: string,        // 사용자 지정 별칭
  width: number,        // 가로 px
  height: number,       // 세로 px (모드 ①②에서는 0으로 저장)
  format: 'jpg'|'png',
  quality: number,      // 1~100
  mode: 1|2|3,          // 리사이징 모드

## 개발 진행 순서

### Step 1 — 프로젝트 초기화
- React + Vite + Tailwind CSS 세팅
- 프로젝트명: img-resizer
- AGENTS.md 기준 디렉토리 구조대로 빈 파일 생성
- App.jsx: Header + 전체 레이아웃 골격 (업로드 / 설정+미리보기 2단 / 즐겨찾기 순)
- 불필요한 Vite 기본 파일 정리

### Step 2 — 이미지 업로드 + 미리보기
- Uploader.jsx 구현
- 드래그 앤 드롭 + 클릭 업로드
- 드래그 상태별 UI (기본 / dragover / dragleave)
- 업로드 완료 시 원본 미리보기 + 원본 사이즈 표시
- 페이지 전체 드래그 진입 시 업로드 영역 하이라이트
- 미지원 형식 드롭 시 토스트 메시지
- 이미지 상태는 App.jsx에서 관리, setter를 props로 전달

### Step 3 — Canvas 리사이징 로직
- useImageResize.js 구현
- 모드 ① 가로 기준: 가로 고정, 세로 비율 자동 계산
- 모드 ② 세로 기준: 세로 고정, 가로 비율 자동 계산
- 모드 ③ 가로×세로 지정: 가로 기준 비율 리사이징 후 세로 캔버스 중앙정렬
  - 후처리 fitHeight: 세로 기준 재계산 → 가로 초과분 중앙 크롭
  - 후처리 fitWidth: 가로 기준 재계산 → 세로 초과분 중앙 크롭
  - hasMargin 반환 (후처리 버튼 노출 조건)
- 출력 형식 jpg/png, 압축률 quality 적용
- canvas.toBlob()으로 최종 파일 생성

### Step 4 — 설정 UI + 결과 미리보기 + 다운로드
- ResizeSettings.jsx: 모드 선택 / px 입력 / 파일형식 / 압축률
  - 모드별 입력 필드 활성화·비활성화 처리
  - 설정값 변경 시 useImageResize 호출 → 실시간 갱신
- Preview.jsx: 결과 미리보기 / 사이즈+용량 표시 / 후처리 버튼 / 크롭 오버레이
- Downloader.jsx: 다운로드 버튼 / 미업로드 시 비활성화

### Step 5 — 즐겨찾기 CRUD + localStorage
- useFavorites.js: addFavorite / deleteFavorite / reorderFavorites
- Favorites.jsx: 버튼 목록 / 툴팁 / 삭제 / 드래그 순서 변경
- [+ 추가] 버튼 → 등록 모달 (별칭/크기/형식/압축률/모드/후처리)
- 즐겨찾기 클릭 시: 설정 자동 적용 → 리사이징 → 즉시 다운로드

### Step 6 — UI 다듬기 및 반응형 처리
- 데스크탑(768px~): 설정+미리보기 좌우 2단
- 모바일(~768px): 단일 컬럼
- 흰 배경 + 파란 계열 포인트 컬러
- 토스트: 우하단 고정, 2초 후 자동 사라짐
- 빈 상태 안내 UX
- 파비콘 + 페이지 타이틀: "IMG Resizer"

---
> Source: [hykim-1010/img_resizing](https://github.com/hykim-1010/img_resizing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
