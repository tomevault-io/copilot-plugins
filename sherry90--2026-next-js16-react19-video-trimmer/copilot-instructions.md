## 2026-next-js16-react19-video-trimmer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A local-first video trimmer. The browser streams local files over loopback to the bundled Node/FFmpeg process; URL sources use Streamlink or yt-dlp before the same frame-accurate native trim stage.

## Commands

### Development

```bash
npm run dev           # Start dev server (localhost:3000, Turbopack)
npm run build         # Production build
npm start             # Start production server
npm run lint          # Run ESLint
npm run type-check    # TypeScript type checking
```

### Testing

```bash
npm test              # Run Vitest unit tests
npm run test:ui       # Vitest UI
npm run test:coverage # Coverage report
npm run test:e2e      # Playwright E2E tests (most skipped)
npm run test:e2e:ui   # Playwright UI
```

## Quick Reference

### 키보드 단축키 (편집 화면)

| 키                 | 기능                              |
| ------------------ | --------------------------------- |
| **Space**          | 재생/일시정지                     |
| **I** / **O**      | In Point / Out Point 설정         |
| **←** / **→**      | 1프레임 이동 (1/30초)             |
| **Shift + ←/→**    | 1초 이동                          |
| **Home** / **End** | In Point / Out Point로 점프       |
| **A**              | 미리보기 (선택 구간 재생)         |
| **휠**             | 타임라인 줌 (1x ~ 10x, 커서 기준) |
| **Shift + 휠**     | 타임라인 가로 패닝 (줌 > 1일 때)  |

단축키는 입력 필드 포커스 시 비활성화됩니다.

### 핵심 아키텍처 (요약)

- **State**: Zustand 단일 스토어 (`src/stores/useStore.ts`). 읽기는 `src/stores/hooks/`(reactive) / `snapshot.ts`(이벤트용 비반응) 분리 — 컴포넌트가 `useStore` 직접 호출 금지.
- **Phase**: idle → uploading → editing → processing → completed | error (순차 진행)
- **Layers**: `app → widgets → features → shared/stores/lib` (FSD 계열 단방향)
- **Processing**: 로컬은 loopback raw upload → 네이티브 FFmpeg, URL은 chzzk→streamlink / 그 외→yt-dlp → 공통 네이티브 FFmpeg
- **Custom server**: `server.ts`가 다운로드/SSE/프록시를 Next 우회(raw bypass)

> 아키텍처·설계 상세(레이어, selector 분리, URL 파이프라인, player-timeline sync, 커스텀 서버)는 **[.docs/01_OVERVIEW.md](.docs/01_OVERVIEW.md)** 참조.

## Critical Patterns (주의점만)

작업 시 반드시 지킬 것. 배경 설명은 [.docs/01_OVERVIEW.md](.docs/01_OVERVIEW.md) 참조.

1. **Player-Timeline sync**: `timeupdate` 핸들러는 `isScrubbing || player.seeking()` 동안 스토어를 갱신하지 말 것(플레이헤드 튐 방지). 이벤트 핸들러는 `snapshot.ts` getter로 최신값을 읽어 stale closure를 피한다. (`playerStoreSync.ts`, `Playhead.tsx`, `usePreviewPlayback.ts`)
2. **Phase 전환**: 항상 순차 진행. 전환 시 이전 phase 정리 필수(blob URL revoke, 서버 임시파일 `DELETE /api/download/[jobId]`).
3. **Memory cleanup**: object URL revoke, video.js `player.dispose()`, wavesurfer `destroy()`, useEffect cleanup. 모듈은 `registerCleanup()`으로 등록 → `reset()`이 `runAllCleanups()`. **URL 소스는 `videoFile.url`이 proxy URL이라 `reset()`의 revokeObjectURL이 no-op** (createObjectURL은 파일 소스·export blob에만 사용).
4. **Player 접근**: `useVideoPlayerContext()`로 얻고 항상 존재 확인 후 호출.
   ```typescript
   const { player } = useVideoPlayerContext();
   if (!player) return;
   player.currentTime(time);
   ```

## Development Workflow

### Adding Features

1. Add state to Zustand store with validation (`constraintUtils.ts` 검증 패턴 준수)
2. Create feature folder with components/hooks/utils
3. Use context only if needed (avoid prop drilling)
4. Add unit tests for logic, E2E tests for workflows
5. Run `npm run type-check` before committing

## Important Constraints

### Supported Formats

MP4, WebM, OGG, MOV, M4V, AVI, WMV, MKV, FLV, TS, 3GP, 3G2, MPEG, MPG (14 formats)

### File Size Limits

- **Recommended**: 500MB (safe)
- **Warning**: 1GB (caution)
- **Soft max**: 2GB (memory check)
- **Hard max**: 5GB (absolute limit)

### Trim Accuracy

정확 트리밍 경로에서 `-c copy`를 사용하지 않는다. 입력 seek 후 선택 구간을 디코딩·재인코딩하며 허용 오차는 원본 1프레임이다. HLS raw segment 입력은 짧은 pre-roll을 처음부터 디코딩한 뒤 output seek한다.

## Git Commit Guidelines

### Commit Process

**ALWAYS use interactive staging**:

```bash
git add -p  # or git add --patch
```

### Commit Message Format

**Language**: Write commit messages in Korean (한국어)

**DO NOT include**:

- AI authorship mentions
- Co-Authored-By lines
- References to Claude or AI assistance

**Format**:

```
<type>: <subject in Korean>

<body in Korean - optional>
```

**Types**:

- `feat`: 새 기능
- `fix`: 버그 수정
- `refactor`: 리팩토링
- `docs`: 문서 변경
- `test`: 테스트 추가/수정
- `chore`: 기타 변경

**Example**:

```
feat: 타임라인 줌 기능 추가

Ctrl+휠로 타임라인 줌 조절 가능
- 줌 범위: 1x ~ 10x
- Waveform과 연동
```

## Documentation

**For detailed information, always refer to:**

- `**.docs/00_INDEX.md**` - Documentation index
- `**.docs/01_OVERVIEW.md**` - 아키텍처 & 설계 (레이어, 상태관리, URL 파이프라인, 커스텀 서버)
- `**.docs/02_API.md**` - API endpoint specs
- `**.docs/03_DEPENDENCIES.md**` - Dependency bundling (ffmpeg/yt-dlp/streamlink)
- `**.docs/04_DEVELOPER_GUIDE.md**` - Patterns, testing, implementation details
- `**scripts/README.md**` - Scripts documentation

This file (CLAUDE.md) is a quick reference guide for Claude Code. For comprehensive technical details, patterns, and implementation specifics, see `.docs/`.

---
> Source: [Sherry90/2026-Next.js16-React19-Video_trimmer](https://github.com/Sherry90/2026-Next.js16-React19-Video_trimmer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
