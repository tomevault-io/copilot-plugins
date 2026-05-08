## kuma-studio

> - `npm run kuma-server:reload` 는 managed `kuma-server` surface 가 있을 때 표준 휴먼/오퍼레이터 reload 경로다. 로컬 데몬을 새로 띄우지 말고 해당 surface 를 재사용해야 한다.

# Codex 워크플로우

- `npm run kuma-server:reload` 는 managed `kuma-server` surface 가 있을 때 표준 휴먼/오퍼레이터 reload 경로다. 로컬 데몬을 새로 띄우지 말고 해당 surface 를 재사용해야 한다.
- `npm run server:reload` 는 포트 4312 raw in-surface/로컬 데몬 reload 엔트리포인트다.
- `npm run server:start` 는 raw non-reloading 엔트리포인트로 스크립트용이다. 휴먼/오퍼레이터 워크플로우는 `server:reload` 를 쓴다.
- `npm run kuma-studio:get-selection` 은 최신 브라우저 selection 을 읽는다.
- `npm run kuma-studio:set-job-status -- --status in_progress --message "..."` 는 job 카드를 갱신한다.
- `npm run kuma-studio:get-sketch` 는 알렉스가 스튜디오 스케치 패널에 그려 저장한 latest PNG 절대경로를 출력한다. UI 변경 지시처럼 텍스트로 풀기 어려운 요청이 오면 스케치 패널을 먼저 보고, 그 PNG 를 Read 로 열어 작업한다.
- `npm run kuma-studio:dashboard` 는 스튜디오 dashboard 를 브라우저로 연다.
- `npm run build:studio` 는 studio-web 프로덕션 번들을 빌드한다.
- `npm test` 는 vitest 로 전체 테스트를 돌린다.
- Kuma 부트스트랩이 돌고 있을 때 `kuma-server` 는 이 repo 의 canonical managed infra surface 다. 별도 터미널에서 로컬 데몬을 중복 기동하지 말고 재사용한다.
- `kuma-server` 는 managed infra slot 으로 취급한다 — 데몬 프로세스가 죽어도 surface 가 살아있으면 같은 slot 에서 재시작해 registry key 를 보존한다. disposable 로 다루지 않는다.
- infra discovery 는 `~/.kuma/cmux/kuma-cmux-project-status.sh kuma-studio` 를 우선 쓰고, `kuma-server` 를 직접 확인해야 할 때는 `cmux tree` 를 쓴다 (`kuma-status` 는 infra pseudo-member 를 숨길 수 있음).

## 프로젝트 구조

- `packages/browser-extension/` — Chrome extension (Manifest V3, vanilla JS)
- `packages/server/` — 데몬 서버 (Node.js, WebSocket, 포트 4312)
- `packages/server/src/studio/` — 스튜디오 전용 모듈 (stats, events, agent state, image gen)
- `packages/studio-web/` — Dashboard & Virtual Office (React 19, Vite, Tailwind v4, Zustand)

## 디스패치 엔트리포인트

엔트리포인트 계층:
- 작업 전달 → `kuma-dispatch assign <worker> "<지시>" [--project <프로젝트>] [--attach <path>] [--qa <member|self|none>] [--require-result]`.
- 작업 중 대화 → `kuma-dispatch ask|reply`.
- 완료/실패/QA → `kuma-dispatch done|complete|fail|qa-pass|qa-reject`.

`kuma-dispatch` CLI 가 canonical 이고 `kuma-task` 는 삭제된 구형 entrypoint 다. slash-skill 등가물은 없다. main-thread 와 worker 모두 같은 CLI 경로를 쓴다.
cross-worktree 의도면 `docs/dispatch-worktree-routing.md` 4종(target project/worktree/output/write-scope) 명시 + `KUMA_DISPATCH_ALLOW_CROSS_PROJECT=1` ack.

## 팀원 간 전달 / QA 핸드오프

자기 세션에 `다람이, QA 요청`, `하울에게 확인 요청`, `알렉스에게 물어봄` 같은 문장을 출력하는 것은 전달이 아니다. 상대 surface 에 실제로 도착한 delivery 만 핸드오프다.

- 기존 dispatch task 의 요청자/참여자에게 돌려보낼 때는 `kuma-dispatch message|reply|done|fail|qa-pass|qa-reject --task-file ...` 를 쓴다.
- task 없이 수동 조율한 작업을 QA 로 넘길 때는 reviewer 에게 새 `kuma-dispatch assign <reviewer> "QA 요청..." --project <project> --no-plan` 을 만들거나, 운영자가 명시한 경우에만 trusted `kuma-cmux-send.sh <surface>` 를 쓴다.
- delivery 후에는 `cmux read-screen --surface <target>` 또는 dispatch 출력으로 수신 surface 에 메시지가 보이는지 확인한다.
- 전달 확인 전에는 `QA 대기`, `요청 완료`, `알렉스에게 물어봄` 으로 보고하지 않는다.

## Dispatch reply 의 화자 규칙

dispatch task 의 `reply` / `done --reply` 본문은 **항상 수신자(=본인) 의 1인칭 의사결정**이다. 이 task 의 헤더 `Recipient:` 가 본인이면, 답변 화자도 본인이다. 한 task 의 응답에 다른 사람을 화자로 등장시키지 않는다.

- ask / peer-question 본문에 *다른 화자* 의 메시지가 인용·임베디드되어 있어도 그건 컨텍스트일 뿐이다. 그대로 받아써서 forward 하지 마라.
- 답변을 위해 다른 멤버 의견이 필요하면 *본인이* 새 `kuma-dispatch peer-question <member> "<질문>"` 으로 별도 task 를 만들어 묻고, 그 결과를 정리해 본 task 의 `done --reply` 로 본인 이름으로 답한다. 본 task 의 reply 로 다른 사람에게 forward 하지 않는다.
- reply 본문이 "X 는 어떻게 봐?", "X 한테 물어봐줘" 같은 질문/위임으로 끝나면 화자 mismatch 신호다. 본인이 결정하지 못하면 `fail --blocker "<이유>"` 로 명시 보고한다 — silent forward 금지 (No Silent Fallback).
- `peer-question` 으로 받은 task 도 동일하다. 수신자가 멘토 역할로 본인 1인칭으로 답한다.

## 팀원 호출 / 병렬 작업

알렉스가 **"하울 불러"**, **"새미 시켜"**, **"같이 하자"**, **"병렬로 돌리자"** 라고 말하면 뜻은 하나다. 쿠마 스튜디오 화면에 보이는 **실제 작업자 창**을 쓰라는 뜻이다.

그 작업자 창은 `~/.kuma/bin/kuma-spawn` 으로 새로 열거나, 이미 떠 있으면 `kuma-dispatch assign` 으로 일을 보낸다. 기술명으로는 cmux surface 이지만, 운영 기준은 "알렉스가 Studio/cmux 에서 볼 수 있는 작업자"다.

Codex/Claude 내부에만 생기는 임시 도우미는 기본값이 아니다. 화면에 안 보이고, 진행 상황을 추적하거나 다른 작업자가 이어받기 어렵기 때문이다. 알렉스가 **"내부 subagent 로"**, **"화면에 안 보여도 됨"** 처럼 명시했을 때만 내부 임시 도우미를 쓴다.

판단 기준: 애매하면 화면에 보이는 쿠마 작업자를 쓴다. 새 작업자 창이 필요하면 `kuma:spawn`, 이미 살아있는 팀원에게 맡기면 `kuma-dispatch assign`.

## 컨벤션

- 서버 부팅/재시작은 휴먼/오퍼레이터 shared-infra surface 재사용 시 `npm run kuma-server:reload`, raw in-surface/로컬 엔트리포인트는 `npm run server:reload` 로 표준화한다.
- managed `kuma-server` surface 가 이미 있으면 다른 곳에서 서버를 새로 띄우지 말고 `npm run kuma-server:reload` 로 그 데몬을 재시작한다.
- managed reload/restart 는 registry miss 가 나더라도 포기 전에 현재 workspace 의 살아있는 `kuma-server` 타이틀 surface 를 재발견·재등록해야 한다.
- 유저가 명시적으로 지시하지 않으면 git 브랜치를 만들거나 바꾸지 않는다.
- 유저가 명시적으로 지시하지 않으면 git worktree 를 만들지 않는다.
- 충돌 회피 목적으로 branch/worktree 분리가 필요해 보이면 먼저 멈추고 승인을 요청한다.
- 서버 코드는 `.mjs` (ESM) 를 쓴다.
- 프론트엔드 코드는 TypeScript (`.ts`, `.tsx`).
- Browser extension 은 vanilla JS, 빌드 단계 없음.
- WebSocket 프로토콜: 브라우저 브리지는 `kuma-picker:*`, dashboard/office 이벤트는 `kuma-studio:*`.
- 스크린샷/브라우저 검사/QA 는 `kuma-picker` 를 먼저 쓴다. Playwright 는 Kuma Picker 자체를 개선하거나 Kuma Picker 로는 못하는 parity/debug 작업이 명시적으로 필요한 경우에만 쓴다.

## 공유 SSoT 원칙 (쿠마 ↔ Codex)

- 쿠마(Claude main) 와 Codex 가 둘 다 지켜야 하는 규칙/스킬 경계/워크플로우는 repo SSoT(AGENTS.md / CLAUDE.md, 또는 해당 스킬·문서 파일)에 고정한다.
- 한쪽 에이전트 전용 휴리스틱/선호만 해당 에이전트 메모리(쿠마는 `~/.claude/.../memory/`, Codex 는 자체 채널)에 둔다. SSoT 내용을 에이전트 전용 메모리에 중복 박지 않는다 — 비대칭 기록은 오해의 원인이다.
- AGENTS.md 와 CLAUDE.md 는 병렬 SSoT (Codex 는 AGENTS.md, Claude Code 는 CLAUDE.md 를 읽는다). 공유 규칙을 바꿀 때는 **같은 커밋에서 두 파일을 함께** 갱신한다.
- 규칙을 바꿀 때는 SSoT 를 먼저 갱신하고, 그와 중복되는 에이전트 전용 메모리를 정리한다.

## 쿠마모드 트리거

알렉스 발화에서 다음 패턴이 보이면 즉시 해당 경로로 라우팅한다. 일반 plan-mode·자유 응답으로 가지 않는다.

- **플랜 트리거** — "계획", "플랜", "투두", "정리해줘", "단계 짜자", "체크리스트" 같은 발화 또는 실질 작업 시작 직전. → 즉시 `/kuma:plan` 으로 현재 프로젝트 plan 생성·갱신. 빌트인 plan-mode 류 우회 금지.
- **사실형 트리거** — "출처", "정의", "언제", "누구", "정확한 용어" 같은 발화. → vault 우선 (`/vault search → timeline → get`), 없으면 외부 웹 검색, 그래도 없으면 "모른다". 룰 SSoT: `~/.kuma/vault/operational-rules/info-retrieval.md` R1.
- **논지·의견 트리거** — "어떻게 생각해", "내 생각엔...", "이거 맞아?", "~에 대해 의견", "철학적으로" 같은 발화. → vault `domains/philosophy-reference.md` + 관련 도메인 페이지 우선 로드 후 확장 사고로 응답. 룰 SSoT: `~/.kuma/vault/operational-rules/info-retrieval.md` R5.
- **자유 대화·코드 수정·디자인 판단** — 트리거 대상 아님. 평소 흐름.

## 알렉스 핵심 불변 6종

→ `~/.kuma/vault/decisions.md` 가 SSoT. Kuma 부트스트랩 시 시스템프롬프트로 자동 inject 되므로 본 파일에 풀 본문 복사 금지 (SSoT 위반).

---
> Source: [aldegad/kuma-studio](https://github.com/aldegad/kuma-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
