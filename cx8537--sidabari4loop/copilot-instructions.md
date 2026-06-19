## sidabari4loop

> 이 문서는 Sidabari4Loop 프로젝트에서 작업할 때 **반드시** 따라야 할 규칙과 지침을 정의합니다.

# CLAUDE.md

이 문서는 Sidabari4Loop 프로젝트에서 작업할 때 **반드시** 따라야 할 규칙과 지침을 정의합니다.
모든 코드 작성·수정·디버깅 전에 이 문서를 읽고 시작하세요.

---

## 0. 프로젝트 정체성

- **프로젝트명**: Sidabari4Loop
- **표시명 (사용자 대면)**: Sidabari4Loop
- **식별자**: `kr.co.nullnull.Sidabari4Loop` (Tauri identifier · crate `sidabari4loop`)
- **본질**: Claude Code를 로컬 PTY에서 실행하고, Claude Code의 **Hook 이벤트**를 수신해 정해진 동작을 수행하는 1인용 데스크톱 도구. 원본 **Sidabari**(EC2 배포·진단 자동화)에서 SSH/SFTP/빌드/배포/진단을 모두 제거하고 분리한 파생 프로그램이다.
- **저작**: 코드·문서 100% Claude Code가 작성·유지보수한다. 인간 협업자는 요구사항·사양·검수만 담당한다 — **이 문서를 읽는 너(Claude Code)가 이 프로젝트의 저자이자 유지보수자**다.
- **기술 스택**: Tauri 2.x + Vite + React 19 + TypeScript + Rust
- **상세 사양**: `SIDABARI4LOOP_SPEC.md` 참조

---

## 1. 절대 원칙 (NEVER 규칙)

### 1.1 코드를 추측하지 말 것

> **추측은 모든 버그의 시작이다.**

- ❌ 절대 금지:
  - 함수가 어떻게 동작할 거라고 **상상해서** 수정하기
  - 파일을 읽지 않고 "아마 이럴 것이다"라고 가정한 채 코드 작성
  - 에러 메시지 일부만 보고 원인을 단정한 뒤 즉흥적으로 패치
  - 라이브러리 API를 기억에 의존해 호출 (특히 Rust crate, Tauri API, xterm.js API)
- ✅ 반드시 할 것:
  - **수정 전에 관련 코드를 먼저 읽는다.** 함수 본체, 호출부, 인터페이스, 타입 정의 모두.
  - 외부 라이브러리 사용 시 **공식 문서 또는 실제 소스를 확인**한다.
  - 에러 발생 시 **스택 트레이스 전체와 `Caused by` 체인을 끝까지 추적**한다.
  - 가설을 세웠으면 코드를 봐서 **검증**한 뒤에야 수정한다.
  - 모르는 코드는 "모른다"고 명시하고, 확인 후 진행한다.

### 1.2 보안을 최우선으로

> Sidabari4Loop은 로컬 PTY에서 임의 프로그램(claude·셸)을 실행하고, Claude Code 훅을 사용자 프로젝트의 `.claude/settings.local.json`에 설치한다. 자격증명을 다루지는 않지만, **명령 실행·파일 쓰기·IPC 경계**가 위협 영역이다.

#### 1.2.1 명령 실행 / PTY

- **사용자 입력·외부 텍스트를 셸 명령 문자열로 직접 조합하지 않는다.**
  - ❌ 금지: `format!("{} {}", program, args)` 후 셸에 통째 전달
  - ✅ 권장: `portable-pty`의 `CommandBuilder`로 **프로그램과 인자를 개별 전달**. Windows npm shim 래핑(`cmd.exe /c <abs.cmd>`)도 인자를 분리해 넘긴다 (`pty.rs`).
- **Hook payload·Claude 출력 같은 외부 텍스트를 절대 명령으로 실행하지 않는다.** `append-event.js`/`gate.js`는 파일 append와 게이트 응답만 한다.

#### 1.2.2 파일 시스템 / 경로 검증

- **훅 설치 디렉토리는 검증한다** (`hook_installer.rs`의 `validate_dir`: 비어있지 않음·절대경로·존재·디렉토리).
- 기존 `settings.local.json`은 **백업 후** 병합 쓰기한다. Sidabari4Loop 관리 영역(`_sidabari4loop` 마커)만 교체하고 사용자 설정은 보존한다.
- 임의 경로 쓰기 금지. 설정/데이터는 Tauri의 `app_config_dir`/`app_data_dir` 안에만 둔다.

#### 1.2.3 IPC (Tauri Command)

- Rust→JS로 노출하는 command 입력을 검증한다. command 권한 범위를 최소화한다 (한 command가 너무 많은 일을 하지 않도록).
- `permissions.deny > permissions.allow` 원칙 유지. capabilities(`src-tauri/capabilities/default.json`)는 필요한 플러그인 권한만 화이트리스트.

#### 1.2.4 감사 로그 / 권한

- Hook 이벤트는 SQLite(`audit.sqlite3`)에 적재한다. 파일 권한은 Unix에서 `0600`, 훅 디렉토리/스크립트는 `0700`.
- 로그/콘솔 출력에 민감 정보를 찍지 않는다.

#### 1.2.5 의존성 관리

- `cargo audit` / `npm audit`을 정기 실행. 새 의존성 추가 시 메인테이너·다운로드·최근 업데이트 확인.

### 1.3 자동화 안전 규칙

- **모든 자동 재시도 금지.** PTY 종료 시 OS 셸로 1회 폴백 외에는 자동 재spawn하지 않는다. 그 외 실패는 멈추고 사용자가 결정.
- **Claude 추천 자동 실행 금지.** PreToolUse 게이트는 항상 사람의 명시적 결정(허용/거부)을 거친다. 무응답은 deny(보수적).
- **이벤트 수신만으로 모달을 자동으로 띄우지 않는다** (설정/게이트 외).

---

## 2. 작업 절차

### 2.1 새 기능 구현 시

1. `SIDABARI4LOOP_SPEC.md` 해당 부분을 다시 읽는다.
2. 영향받는 기존 코드를 모두 찾아 읽는다 (`grep`, 읽기, 호출부 추적).
3. 데이터 흐름과 책임 경계를 파악한다 (특히 Hook 이벤트 흐름: 스크립트 → `events.jsonl` → `hooks_bus.rs` → `HookBridge`).
4. 보안 영향을 검토한다 (§1.2).
5. 작은 단위로 구현하고 각 단위를 검증한다.
6. 사양서와 일치하는지 확인한다.

### 2.2 버그 수정 시

> 이 절에서 추측 금지 원칙이 가장 강하게 적용된다.

1. 재현 조건을 명확히 한다 (어떤 입력·환경).
2. **스택 트레이스를 끝까지 읽는다.** `Caused by` 체인 끝까지 (Rust panic·JS 에러 동일).
3. 관련 코드를 직접 본다 (에러 함수 본체·호출부·인자 타입·라이브러리 실제 동작).
4. 가설을 "X가 원인일 가능성이 있다"로 표현하고, 코드로 검증한다.
5. 수정하고, 같은 패턴의 다른 곳도 점검한다.

### 2.3 의심스러울 때

- 코드를 보지 않은 함수를 호출하려 할 때 → 본다
- "아마 이렇게 동작할 것 같다" → 확인한다
- 에러 보고 즉시 패치가 떠오를 때 → 진짜 원인인지 의심한다
- 라이브러리 시그니처를 기억에 의존 → 문서/소스 확인
- 보안에 영향 줄 변경 → §1.2 다시 읽기

---

## 3. 코드 스타일

### 3.1 Rust
- `clippy` 경고를 무시하지 않는다. 끄려면 명시적 이유 주석.
- `unwrap()`/`expect()`는 정말 panic이 정상인 경우에만. 그 외엔 `Result`.
- 에러 메시지는 사용자에게 유의미하게 (어떤 작업이 왜 실패했는지).
- 스레드/watcher 실패 시 자동 재시도 X — stderr 로그 + 사용자 결정.

### 3.2 TypeScript
- `any` 금지. 필요하면 `unknown` 후 narrowing.
- 외부 입력(IPC 응답 포함)은 런타임 검증 (Zod — `lib/config.ts`).
- React 컴포넌트는 단일 책임. 패널 단위 모듈화.
- 상태는 Zustand(`useAppStore`)에, 컴포넌트 로컬로 충분한 건 로컬에.

### 3.3 공통
- 함수가 길어지면 분리(50줄 넘으면 의심). 매직 넘버/문자열은 상수화.
- 주석은 "왜"를 설명. "무엇"은 코드가 말하게.

---

## 4. 테스트 정책

- 새 로직에는 단위 테스트를 함께 작성.
- 보안 관련 함수(경로 검증, 설정 병합 등)는 반드시 테스트.
- 이벤트 분류(`classify_event`)·훅 병합(`merge_hooks`) 같은 로직은 다양한 입력으로 테스트.

---

## 5. 커뮤니케이션

- 변경 사항은 **무엇을·왜·어떻게** 바꿨는지 명확히.
- 사양서와 다른 결정을 했다면 명시하고 이유 설명.
- 모호하면 추측하지 말고 질문한다. "확실하지 않습니다"가 틀린 답보다 낫다.

---

## 6. 사양서와의 관계

- `SIDABARI4LOOP_SPEC.md`가 1차 참조 문서다.
- 사양서와 이 문서가 충돌하면 사양서 우선.
- 동작·사양·설정 스키마를 바꾸면 사양서도 같은 변경에 포함한다.

---

## 7. 핵심 요약 (매 작업 시작 전 다시 읽기)

> **1. 코드를 추측하지 마라. 보고 나서 고쳐라.**
> **2. 명령 실행·파일 쓰기·IPC 경계가 위협 영역이다. 외부 텍스트를 명령으로 실행하지 마라. §1.2.**
> **3. 자동 재시도/자동 실행 금지. 사람이 결정한다.**
> **4. 모르면 묻는다. 추측하지 않는다.**
> **5. 에러는 `Caused by` 끝까지 추적한다.**

---

## 부록 A: 자주 쓰는 라이브러리 참조

작업 전 공식 문서 확인 권장:

- **Tauri 2.x**: https://tauri.app/
- **portable-pty**: https://docs.rs/portable-pty/
- **xterm.js**: https://xtermjs.org/
- **react-resizable-panels**: https://github.com/bvaughn/react-resizable-panels
- **rusqlite**: https://docs.rs/rusqlite/
- **shadcn/ui**: https://ui.shadcn.com/
- **Zustand**: https://docs.pmnd.rs/zustand/
- **Claude Code Hooks**: https://docs.claude.com/en/docs/claude-code/hooks

## 부록 B: 보안 체크리스트 (변경 전 자가 점검)

- [ ] 사용자 입력/외부 텍스트가 셸 명령 문자열로 직접 조합되지 않는가? (CommandBuilder 개별 인자)
- [ ] Hook payload·Claude 출력을 명령으로 실행하지 않는가?
- [ ] 훅 설치 디렉토리 경로를 검증하는가? `settings.local.json`을 백업 후 병합하는가?
- [ ] Tauri command 입력이 검증되는가? capabilities가 최소 권한인가?
- [ ] 감사 로그/설정 파일 권한이 적절한가(Unix 0600/0700)?
- [ ] 자동 재시도/자동 실행이 없는가? 게이트가 사람의 결정을 거치는가?
- [ ] `cargo audit` / `npm audit` 통과하는가?

## 부록 C: UI 스타일 가이드

다크 테마 단일 모드. 신규 컴포넌트는 이 가이드를 따라 색·간격·구조 일관성을 유지한다. **색상은 항상 토큰을 사용하고 직접 hex를 박지 않는다.**

### C.1 색상 토큰 (App.css `.dark`)

| 변수 | 값 | 용도 |
|---|---|---|
| `--background` | `#191A1C` | 본문 배경 (터미널·콘솔) |
| `--card` | `#26282B` | 패널 타이틀바·모달 배경 |
| `--card-foreground` | `#CCCED3` | 카드 위 텍스트 |
| `--foreground` | `#CCCED3` | 일반 본문 텍스트 |
| `--muted-foreground` | `#B0B3B7` | placeholder·타임스탬프·비활성 |
| `--primary` | `#404348` | 기본 버튼 배경 |
| `--secondary` | `#25324D` | 포커스 받은 패널 타이틀바 |
| `--ring` | `#35538F` | focus ring |
| `--destructive` | oklch(0.704 0.191 22.216) | 에러·게이트 거부 |
| `--accent-gold` | `#B28B55` | Sidabari4Loop 브랜드·`[HOOK]` 콘솔 태그·모달 제목 강조 |
| `--action-green` | `#508956` | 활동 인디케이터 "thinking" 도트 |

신규 토큰 추가: `App.css` `.dark`에 `--name: hex;` + `@theme inline`에 `--color-name: var(--name);`.

### C.2 텍스트 위계

| 단계 | 토큰 | 사용처 |
|---|---|---|
| 강조 | `text-accent-gold` / `text-primary-foreground` | 브랜드·강조 라벨·모달 제목 |
| 일반 | `text-foreground` / `text-card-foreground` | 패널 타이틀·본문 |
| 흐림 | `text-muted-foreground` | placeholder·타임스탬프·비활성 |

### C.3 버튼 (`components/ui/button.tsx`)

- 직접 `<button>` 작성 금지. `<Button>` 사용.
- variant: 기본 `default`, 보조 액션 `ghost`.
- size: `sm`(h-7) / `xs`(h-6) / `icon-sm`(28²) / `icon-xs`(36², 아이콘 18px). 패널 내 아이콘 버튼은 `icon-xs`(예: ⚙ 설정, 🗑 콘솔 비우기 — `variant="ghost"`).
- 아이콘 의미색은 `className="[&_svg]:text-X"` (위험/에러 → `text-destructive`).

### C.4 레이아웃·패널

- **MainLayout**: 상/하 `react-resizable-panels` 수직 Group + `ResizeHandle`. 비율은 localStorage(`app-rows`).
- **패널 root**: `flex h-full flex-col bg-background` + `onMouseDown={onMouseDown}`.
- **패널 타이틀바**: `mx-0.5 mt-0.5 rounded-md transition-colors` + 포커스 분기 (`isFocused ? "bg-secondary" : "bg-card"`).
- **포커스 추적**: `usePanelFocus(panelId)`. 새 PanelId는 `store/useAppStore.ts`의 `PanelId` 타입에 추가 (현재 `"main-claude" | "console"`).

### C.5 모달/다이얼로그

- shadcn `Dialog`(`components/ui/dialog.tsx`) 사용. 배경 `bg-card`, 제목 `text-accent-gold`, 모서리 `rounded-lg`.
- 액션 버튼은 우측 하단. 주 액션 `default`, 취소 `ghost`.
- 폼 입력: `bg-background border border-input rounded-md px-3 py-2 text-sm placeholder:text-muted-foreground`.
- **이벤트 감지로 모달을 자동 표시하지 않는다.** 사람의 명시적 액션(⚙ 클릭) 또는 게이트 요청에만 띄운다.

### C.6 다크 테마 전제

- `index.html`의 `<html class="dark" style="color-scheme: dark">`로 항상 다크. 라이트 모드 미지원.

### C.7 신규 컴포넌트 체크리스트

- [ ] 색은 토큰 사용 (일회성 hex 금지, 필요 시 신규 토큰 추가).
- [ ] 패널이면 `usePanelFocus(panelId)` 적용.
- [ ] 버튼은 `<Button>`, 의미 아이콘은 `[&_svg]:text-X`.
- [ ] 텍스트 위계(강조/일반/흐림) 명확히.
- [ ] `npm run build` 통과.

---
> Source: [cx8537/sidabari4loop](https://github.com/cx8537/sidabari4loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
