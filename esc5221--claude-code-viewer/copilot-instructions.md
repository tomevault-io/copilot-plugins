## claude-code-viewer

> 이 파일은 이 저장소에서 코드 작업을 할 때 Claude Code (claude.ai/code)에 대한 가이드를 제공합니다.

# CLAUDE.md

이 파일은 이 저장소에서 코드 작업을 할 때 Claude Code (claude.ai/code)에 대한 가이드를 제공합니다.

## 개발 명령어

```bash
# 개발 서버 시작 (핫 리로드 포함)
npm run dev

# 프로덕션 빌드
npm run build

# 프로덕션 빌드 미리보기
npm run preview

# 플랫폼별 배포 패키지 생성
npm run dist:mac    # macOS DMG 생성
npm run dist:win    # Windows 설치 프로그램 생성
npm run dist:linux  # Linux AppImage 생성
```

## 아키텍처 개요

이 프로젝트는 Claude Code CLI 세션을 위한 Electron 기반 데스크톱 뷰어입니다.

### 핵심 구조
- **메인 프로세스** (`electron/`): 파일 시스템 접근, 창 관리, CLI 통합
- **렌더러 프로세스** (`src/`): React 앱, UI 컴포넌트, 상태 관리
- **IPC 통신**: preload 스크립트를 통한 안전한 프로세스 간 통신

### 주요 기술 스택
- Electron 36 + React 19 + TypeScript 5
- Vite (electron-vite로 최적화된 빌드)
- Zustand로 상태 관리
- Tailwind CSS v4로 스타일링

### 중요 파일 경로
- `electron/main.ts` - Electron 메인 진입점, 딥 링킹 처리
- `electron/fileWatcher.ts` - 실시간 세션 업데이트를 위한 파일 감시
- `src/store/sessionStore.ts` - 프로젝트, 세션, 탭 상태 관리
- `src/components/SessionViewer/SessionViewer.tsx` - 세션 렌더링 로직
- `cli/claude-viewer-cli.js` - Claude CLI 통합 스크립트

### 세션 데이터 구조
- 세션은 `~/.claude/projects/*/sessions/*.jsonl` 파일로 저장됨
- 각 줄은 JSON 메시지 객체 (사용자, 어시스턴트, 도구 사용)
- 실시간 업데이트를 위해 chokidar로 파일 변경 감시

### macOS 보안 및 배포
- 하드런타임 활성화, 권한 설정 (`build/entitlements.mac.plist`)
- 자동 공증 스크립트 (`scripts/notarize.js`)
- 딥 링킹을 위한 `claude-viewer://` 프로토콜 핸들러

### 컴포넌트 아키텍처
- **Layout**: 사이드바, 탭 바, 메인 콘텐츠 영역
- **SessionList**: 프로젝트별 세션 목록, 호버 미리보기
- **SessionViewer**: 메시지 렌더링, 도구별 커스텀 디스플레이
- **Tools**: 각 도구 타입(File, Bash, Search 등)별 전용 렌더러

### 성능 최적화
- 세션 콘텐츠 지연 로딩
- 긴 세션을 위한 가상화 목록
- 효율적인 메시지 파싱 및 디핑
- 디바운스된 파일 감시

---
> Source: [esc5221/claude-code-viewer](https://github.com/esc5221/claude-code-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
