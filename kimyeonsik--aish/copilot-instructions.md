## aish

> 셸과 AI를 통합한 Node.js 기반 인터랙티브 터미널 도구.

# CLAUDE.md — claude-shell (aish)

## 프로젝트 개요

셸과 AI를 통합한 Node.js 기반 인터랙티브 터미널 도구.
- **aish**: 대화형 REPL (일반 셸 명령 + AI 쿼리 동시 지원)
- **ai**: one-shot CLI (셸 진입 없이 AI에게 질문)
- 백그라운드 daemon이 Claude Agent SDK 세션을 관리

## 아키텍처

```
사용자
  │
  ├─ aish (대화형)  ─→  shell.ts (AishShell REPL)
  └─ ai "질문"      ─→  client.ts (one-shot)
                            │
                    Unix socket (IPC)
                            │
                       daemon.ts
                            │
                   @anthropic-ai/claude-agent-sdk
                   (continue: true — 서버사이드 세션)
```

daemon은 첫 쿼리 시 자동 시작. `~/.config/claude-shell/daemon.sock` 으로 통신.

## 파일 구조

```
src/
  client.ts       — CLI 진입점. 인자 없으면 shell.ts 로드, 있으면 one-shot
  shell.ts        — AishShell REPL 클래스 (명령 dispatch, 출력 압축, AI 연동)
  daemon.ts       — Claude Agent SDK 래퍼. 쿼리 직렬화, 메모리 추출
  connection.ts   — daemon 시작/연결/소켓 통신 유틸
  protocol.ts     — ClientMessage / DaemonMessage 타입 + NDJSON 직렬화
  i18n.ts         — 다국어 지원 (en/ko), ~/.config/claude-shell/lang 파일 기반
  types.ts        — 공유 타입, 경로 상수, DEFAULT_BUDGET

  context/
    manager.ts    — 컨텍스트 조합 (Memory + Topics + Window + Shell)
    memory.ts     — 장기 메모리 (project/conventions/decisions), JSON 파일 영속
    topic.ts      — 대화 요약 저장소 (최대 10개), 윈도우 eviction 시 자동 생성
    window.ts     — 단기 대화 윈도우 (토큰 기반 LRU)
    shell.ts      — ShellContext: cwd, 최근 명령, exitCode → 시스템 프롬프트용
    tokens.ts     — Claude BPE 근사 토큰 계산 (한글 1.5t, ASCII 4c=1t 등)

shell/
  ai.zsh          — zsh 통합. preexec/precmd hook으로 shell-state.json 갱신

scripts/
  test.sh         — 수동 테스트 스크립트
```

## 핵심 상수 (shell.ts)

```typescript
MAX_OUTPUT_ENTRIES  = 5        // 링 버퍼 크기 (최근 N개 명령 출력 보관)
MAX_OUTPUT_CHARS    = 20_000   // 명령 출력 저장 한도 (압축 후)
MAX_CONTEXT_CHARS   = 16_000   // AI 전송 컨텍스트 한도 (압축 후)
NOTIFY_THRESHOLD_MS = 10_000   // 이 시간 초과 시 OS 알림 발송
```

## 컨텍스트 예산 (types.ts)

```typescript
DEFAULT_BUDGET = { total: 3100, memory: 200, topics: 300, window: 2500, shell: 100 }
```

`commandContext`(최근 명령 출력)는 이 예산 **외부**에 있음.
`systemPrompt.append`에 ephemeral 주입 → 대화 히스토리에 누적되지 않음.

## 개발 워크플로우

```bash
npm run build          # TypeScript 컴파일 → dist/
npm install -g .       # 글로벌 설치 (nvm 경로: ~/.nvm/.../bin/aish)
./reinstall.sh         # uninstall(config 보존) → install (개발용 원클릭 재설치)
./install.sh           # 최초 설치 (빌드 + ~/.local/share 복사 + .zshrc 등록)
./uninstall.sh         # 완전 제거 (--yes 플래그로 config 보존 비대화형 가능)
```

빌드 후 배포까지 항상 두 단계: `npm run build` → `npm install -g .`

## 설계 결정 (변경 시 이유 먼저 확인)

### commandContext를 systemPrompt.append에 주입하는 이유
`continue: true` (서버사이드 세션) 구조에서 클라이언트가 대화 히스토리에
직접 데이터를 삽입할 방법이 없음. `append`가 유일한 ephemeral 주입 지점.
prompt에 붙이면 히스토리에 영구 누적 → 세션 토큰 예산 소진.

### compressOutput() 3단계 전략
에러/결과는 항상 출력 **끝**에 있음. 앞에서 자르면 핵심 정보 소실.
1. ANSI 이스케이프 제거 (무손실, 10~30% 절약)
2. 연속 중복 라인 압축 — 프로그레스 바, "....." (항상 적용)
3. head 30% + tail 70% 윈도우 — limit 초과 시만 적용

### shouldTryShell() — sh -c 실행 전 PATH 확인
`sh -c hello` 같은 명령 실행 시 `sh: hello: command not found`가 stderr에
출력된 후 handleCommandNotFound()가 호출되는 UX 문제.
PATH 또는 SHELL_BUILTINS에 없으면 sh -c를 건너뛰고 바로 not-found 처리.
looksLikeNaturalLanguage()도 dispatch()에서 선행 체크.

### editDistance() transposition 처리
표준 Levenshtein: `gti`→`git` = 거리 2. 3글자 maxDist=1 → 보정 실패.
단순 인접 전위(adjacent transposition)는 fast path로 거리 1 처리.

### OS 알림 (notify)
10초 이상 소요된 명령 완료 시 macOS(osascript) / Linux(notify-send) 알림.
`.unref()`로 이벤트 루프 비블로킹. 없는 환경에서도 크래시 없음.

## 주요 주의사항

- **daemon은 장시간 실행 프로세스**: 수정 후 `aish --stop` 또는 `aish --status`로 재시작 필요
- **두 가지 설치 경로**: `npm install -g`(개발) vs `install.sh`(배포). 글로벌 설치는 nvm 경로에 위치
- **`CLAUDECODE` 환경변수**: Claude Code 내부에서 daemon 실행 시 Agent SDK가 중첩 세션을 거부. `connection.ts`에서 env 삭제로 처리
- **ESM 전용**: `"type": "module"` — `require()` 불가, import 경로에 `.js` 확장자 필수
- **절대 auto-commit 하지 말 것**

## 테스트 방법

별도 테스트 프레임워크 없음. `node --input-type=module << 'EOF' ... EOF` 패턴으로 인라인 검증.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kimyeonsik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
