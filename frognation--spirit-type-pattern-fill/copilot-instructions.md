## spirit-type-pattern-fill

> **GitHub:** https://github.com/frognation/Spirit-type-pattern-fill

# Spirit — Type Pattern Fill · CLAUDE.md

**GitHub:** https://github.com/frognation/Spirit-type-pattern-fill
**Dev:** `npm run dev` → http://localhost:3001
**Deploy:** Vercel (master 브랜치 자동 배포)

---

## 앱 개요

텍스트 또는 이미지를 다양한 패턴으로 채워 SVG 벡터로 익스포트하는 웹앱.
- Next.js 14 App Router + TypeScript + Tailwind CSS v4
- Canvas API 실시간 프리뷰 → SVG export (2× 해상도)
- UI 레퍼런스: https://can-editor-original.vercel.app/ (다크, monospace, border-based)

---

## 파일 구조

```
app/
  layout.tsx       루트 레이아웃 (monospace body, no extra fonts)
  page.tsx         <Editor /> 렌더
  globals.css      CSS 변수, range slider·color picker 커스텀 스타일

components/
  Editor.tsx       메인 레이아웃 + 전체 state 관리
                   · 텍스트 마스크 / 이미지 마스크 전환
                   · 글로벌 Ctrl+V paste 핸들러
                   · SVG export (2× scale)
  Sidebar.tsx      오른쪽 272px 사이드바
                   · 이미지 업로드 존 (drag-drop · click · Ctrl+V paste)
                   · Threshold 슬라이더, Invert 토글
                   · Size & Spacing (cellSize, gap, roundness, strokeDepth)
                   · Color mode (1/2/3/6+) + 색상 피커 + 패턴별 프리셋
                   · Typography (fontSize, letterSpacing, fontFamily, fontWeight)
                   · 로컬 폰트 업로드 (FontFace API, TTF·OTF·WOFF·WOFF2)
                   · Export SVG 버튼
  PatternBar.tsx   하단 중앙 floating bar, 9개 패턴 버튼

lib/
  types.ts         PatternType(9종), PatternOptions, TextConfig, PATTERN_DEFAULTS
  patternEngine.ts buildTextMask, generatePattern, renderToCanvas
                   computeDistField (BFS distance field, contour/outline 패턴용)
  svgExport.ts     exportSVG (색상별 <g> 그룹핑), downloadSVG
```

---

## 9가지 패턴

| ID | 바 라벨 | 설명 | 핵심 옵션 |
|----|---------|------|-----------|
| `pixelMosaic` | PIXEL | 소형 컬러 격자 사각형 | cellSize, gap, roundness, hexGrid, randomize |
| `circleArray` | CIRCLE | 원형 격자 fill | cellSize, gap, hexGrid |
| `checkerboard` | CHECKER | 대형 체커보드 | cellSize, 2색 |
| `stripeFill` | STRIPE | 가로 줄무늬 | cellSize(높이), gap |
| `contourChecker` | CONTOUR | 글자 획 곡선을 따라 흐르는 체커 | strokeDepth(gradient 스무딩) |
| `outlineCircles` | OUTLINE | 외곽선 strokeDepth 이내에만 원 배치 | strokeDepth(깊이) |
| `blockMosaic` | BLOCK | 큰 평면 색상 블록 (픽셀 일러스트) | cellSize, randomize |
| `ledMatrix` | LED | 소형 사각+원 혼합 도트 매트릭스 | cellSize, gap, roundness |
| `vertBars` | BARS | 수직 바 세그먼트 (CRT 스캔라인) | cellSize(폭) |

---

## 핵심 로직 요약

### 마스크 생성

```ts
// 텍스트 → 흰색 글자 / 검정 배경 ImageData
buildTextMask(textCfg, width, height): ImageData   // patternEngine.ts

// 이미지 → 이진 마스크  (Editor.tsx 내부 함수 buildImageMask)
// PNG/알파 있는 이미지: alpha > threshold → inside
// JPG/불투명:           luminance < (255-threshold) → inside
// invert 토글로 반전 가능
```

### 패턴 엔진 흐름

```
mask (ImageData)
  └─ generatePattern(mask, opts) → PatternElement[]
        └─ isInside(mask, x, y) : 각 요소 위치가 마스크 내부인지 체크
  └─ renderToCanvas(canvas, elements, bg)   // 프리뷰
  └─ exportSVG(elements, w, h, bg)          // SVG 문자열 생성
```

`contourChecker`, `outlineCircles` 는 `computeDistField(mask)` (BFS) 로
거리 필드를 먼저 계산한 뒤, 그래디언트로 방향 추출해 패턴에 활용.

### 로컬 폰트 업로드

```ts
const face = new FontFace(fontName, arrayBuffer);
await face.load();
document.fonts.add(face);   // 브라우저 메모리에만, 서버 미전송
```

### 이미지 붙여넣기

```ts
// Editor.tsx
document.addEventListener('paste', (e) => {
  // ClipboardData.items 에서 image/* → Blob → loadImageFromBlob(blob)
  // imgEl state에 저장 → render 시 buildImageMask 사용
});
```

---

## PatternOptions 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `type` | PatternType | 패턴 종류 |
| `cellSize` | 4–80 | 요소 크기 px |
| `gap` | 0–12 | 요소 간격 px |
| `colorMode` | mono/dual/tri/multi | 색상 개수 (1/2/3/6) |
| `colors` | string[] | hex 색상 배열 |
| `bg` | string | 배경색 hex |
| `roundness` | 0–1 | rect 모서리 둥글기 |
| `hexGrid` | boolean | 홀수 행 반 칸 오프셋 |
| `strokeDepth` | 0–1 | OUTLINE 깊이 / CONTOUR 스무딩 강도 |
| `randomize` | boolean | 색상 무작위 배정 |

---

## 앞으로 가능한 개선

- PNG/JPG export
- 패턴 미리보기 썸네일
- 랜덤 seed 슬라이더 (현재 패턴별 고정 seed)
- 글자별 개별 패턴·색상 적용
- JSON preset 저장/불러오기
- Undo / Redo
- 애니메이션 GIF export

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frognation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
