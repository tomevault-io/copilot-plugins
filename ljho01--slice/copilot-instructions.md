## slice

> **Slice**는 Splice에서 다운로드한 오디오 샘플을 로컬에서 탐색·미리듣기하는 데스크톱 앱이다.

# Slice — 프로젝트 규칙 및 문서

## 프로젝트 개요

**Slice**는 Splice에서 다운로드한 오디오 샘플을 로컬에서 탐색·미리듣기하는 데스크톱 앱이다.
macOS Splice 앱의 SQLite DB(`~/Library/Application Support/com.splice.Splice/...`)에서 팩/샘플 메타데이터를 읽어
`~/Slice/` 디렉토리에 오디오 파일을 복사하고, 자체 SQLite DB(`~/Slice/slice.db`)로 관리한다.

**모노레포 구조:**
- `apps/desktop`: Tauri v2 기반 데스크톱 앱
- `apps/landing`: 웹 랜딩페이지 (React + Vite)

## 기술 스택

| 레이어 | 기술 |
|--------|------|
| 프레임워크 | **Tauri v2** (Rust 백엔드 + WebView 프론트엔드) |
| 프론트엔드 | **React 18** + **TypeScript** + **Vite 6** |
| 스타일링 | **Tailwind CSS v4** (`@tailwindcss/vite` 플러그인) |
| UI 컴포넌트 | **Radix UI** + **shadcn/ui** 패턴 (`class-variance-authority`, `clsx`, `tailwind-merge`) |
| 아이콘 | **lucide-react** |
| 가상 스크롤 | **@tanstack/react-virtual** |
| 백엔드 DB | **rusqlite** (SQLite, bundled) |
| 오디오 디코딩 | **Symphonia** (wav, pcm, mp3, flac, ogg, vorbis, aiff) |
| 패키지 매니저 | **Bun** (npm/yarn/pnpm 사용 금지) |

## 디렉토리 구조

```
slice/
├── apps/
│   ├── desktop/                  # Tauri 데스크톱 앱
│   │   ├── src/                  # 프론트엔드 (React + TypeScript)
│   │   │   ├── main.tsx          # 엔트리포인트
│   │   │   ├── App.tsx           # 메인 앱 컴포넌트
│   │   │   ├── types.ts          # 공유 타입 정의
│   │   │   ├── index.css         # Tailwind 설정
│   │   │   ├── lib/
│   │   │   │   └── utils.ts      # cn() 유틸
│   │   │   ├── routes/           # 라우트 페이지
│   │   │   └── components/       # UI 컴포넌트
│   │   ├── src-tauri/            # Rust 백엔드
│   │   │   ├── Cargo.toml
│   │   │   ├── src/
│   │   │   │   ├── main.rs
│   │   │   │   └── lib.rs
│   │   │   ├── capabilities/
│   │   │   └── icons/
│   │   ├── package.json
│   │   ├── vite.config.ts
│   │   └── tsconfig.json
│   └── landing/                  # 웹 랜딩페이지
│       ├── src/
│       │   ├── main.tsx
│       │   ├── App.tsx
│       │   └── index.css
│       ├── public/
│       │   └── icon.png
│       ├── package.json
│       ├── vite.config.ts
│       └── tsconfig.json
├── package.json                  # 모노레포 루트 (workspaces)
└── bun.lock
```

## 핵심 아키텍처

### 앱 Phase 흐름

```
loading → check_library_status →┬─ has_data: true  → ready (메인 UI)
                                └─ has_data: false → import (ImportScreen)
                                                        ↓
                                                  import_from_splice
                                                        ↓
                                                      ready
```

### Tauri 커맨드 (Rust → JS IPC)

| 커맨드 | 설명 |
|--------|------|
| `check_library_status` | DB에 데이터가 있는지, Splice가 설치되어 있는지 확인 |
| `import_from_splice` | Splice DB 읽기 → `~/Slice/`로 파일 복사 → 자체 DB에 메타 저장. `import-progress` 이벤트 발행 |
| `scan_library` | 팩 목록 반환 (JOIN으로 샘플 수 포함) |
| `get_all_samples` | 전체 샘플 목록 (filename 정렬) |
| `get_pack_samples` | 특정 팩의 샘플 목록 |
| `get_waveform` | 파형 피크 데이터 반환 (128 포인트). DB 캐시 사용, 없으면 Symphonia로 실시간 디코딩 |

### DB 스키마

**packs 테이블:**
- `id` INTEGER PK, `uuid` TEXT UNIQUE, `name`, `description`, `cover_url`, `genre`, `permalink`

**samples 테이블:**
- `id` INTEGER PK, `local_path`, `filename`, `audio_key`, `bpm`, `chord_type`, `duration` (ms)
- `file_hash` UNIQUE, `genre`, `sample_type` ("oneshot"/"loop"), `tags` (콤마 구분)
- `pack_uuid` FK → packs.uuid, `waveform_peaks` (JSON 캐시)

### 오디오 재생 방식

- HTML5 `<Audio>` 엘리먼트 사용
- `convertFileSrc()` (Tauri asset protocol)로 로컬 파일 URL 생성
- **Transpose**: `audio.playbackRate = Math.pow(2, semitones / 12)` + `preservesPitch = false`
  - 속도와 피치가 함께 변함 (길이 유지 안 함)
  - 범위: -12 ~ +12 반음
- `requestAnimationFrame` 루프로 실시간 재생 progress 추적

### SampleBrowser 필터 시스템

다중 필터 조합 가능:
- **텍스트 검색** (filename, pack_name, tags)
- **Genre** (체크박스 다중 선택)
- **Instrument** (태그 기반, 사전 정의 키워드 매칭)
- **BPM 범위** (듀얼 레인지 슬라이더)
- **Key** (음정 필터)
- **Type** (all / oneshot / loop)
- **태그 include/exclude** (클릭: 포함, `-` 버튼: 제외)
- 키보드 화살표(↑↓)로 샘플 간 이동 + 자동 재생

### 가상 스크롤 + 미니 파형

- `@tanstack/react-virtual`로 대량 샘플 목록 가상화 (ROW_HEIGHT = 48px, overscan = 15)
- **MiniWaveform**: 1초 디바운스 후 Tauri `get_waveform` 호출, 메모리 캐시, 동시 요청 3개 제한

## 코딩 컨벤션

### TypeScript / React

- 함수형 컴포넌트 + hooks 사용 (클래스 컴포넌트 사용 금지)
- `useCallback`, `useMemo`로 불필요한 리렌더링 방지
- 경로 alias: `@/` → `./src/` (import 시 항상 `@/` 사용)
- 타입은 `src/types.ts`에 중앙 관리, Rust 백엔드와 동일한 구조 유지
- UI 텍스트는 한국어 (버튼, 레이블, 안내 메시지 등)
- `export default function ComponentName()` 형식 사용

### 스타일링

- Tailwind CSS v4 유틸리티 클래스 사용 (인라인 style 최소화)
- 다크 모드 기본 (`<html class="dark">`)
- `cn()` 헬퍼로 조건부 클래스 합성
- shadcn/ui 컴포넌트는 `src/components/ui/`에 위치
- 색상은 CSS 변수 기반 (`--background`, `--foreground`, `--border` 등)
- zinc 계열 중심의 모노톤 다크 테마

### Rust

- 모든 로직은 `src-tauri/src/lib.rs` 단일 파일
- `AppState { db: Mutex<Connection> }`으로 DB 접근 (Tauri managed state)
- 에러는 `Result<T, String>` 반환 (한국어 에러 메시지)
- Tauri 이벤트(`app.emit`)로 프론트엔드에 진행 상황 전달

## 개발 환경

```bash
# 데스크톱 앱 개발 서버 (Vite + Tauri)
bun run dev:desktop    # localhost:1420

# 랜딩페이지 개발 서버
bun run dev:landing    # localhost:3000

# 빌드
bun run build:desktop  # 데스크톱 앱
bun run build:landing  # 랜딩페이지

# 패키지 설치 (루트에서)
bun install

# 특정 앱에 패키지 추가
bun --cwd apps/desktop add <package>
bun --cwd apps/landing add <package>
```

> **필수:** 패키지 매니저는 반드시 **Bun**을 사용한다. `npm`, `yarn`, `pnpm` 명령어 사용 금지.
> 의존성 설치는 `bun install`, 패키지 추가는 `bun add`, 스크립트 실행은 `bun run`을 사용한다.
> 모노레포 구조에서 특정 앱의 의존성을 추가할 때는 `bun --cwd apps/<app-name> add`를 사용한다.

## 주의사항

- `sounds/` 디렉토리는 테스트용 WAV 파일이므로 커밋하지 않는다
- Splice DB는 읽기 전용(`SQLITE_OPEN_READ_ONLY`)으로 접근 — 원본 변경 금지
- `~/Slice/slice.db`가 앱의 자체 데이터베이스
- `waveform_peaks`는 samples 테이블에 JSON 문자열로 캐시됨
- 모든 응답과 UI 텍스트는 한국어로 작성

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ljho01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
