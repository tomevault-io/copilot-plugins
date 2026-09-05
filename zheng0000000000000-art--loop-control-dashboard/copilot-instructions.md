## loop-control-dashboard

> 먼저 [AGENT-GUIDE.md](AGENT-GUIDE.md)를 읽는다 — API·작업 수명주기·금지선·기여 방법이 거기 있다.

# 이 저장소에서 작업하는 에이전트의 규칙

먼저 [AGENT-GUIDE.md](AGENT-GUIDE.md)를 읽는다 — API·작업 수명주기·금지선·기여 방법이 거기 있다.

## 품질 게이트 (필수)
- 코드·데이터 수정 후 커밋 전에 반드시 측정을 실행한다:
  `dotnet run --project server -- measure dev-pack` (아래 CLI, 서버 기동 불필요)
- 위반(deviations)이 0이 될 때까지 수정 후 재측정한다.
- 끝내 통과하지 못하면 남은 위반 목록을 작업 보고에 그대로 포함한다 — 숨기지 않는다.

## 금지 사항
- blueprint·workflow-definition·측정 코드를 수정해서 게이트를 통과하는 것.
  기준 변경은 사람 결재 사항이다.
- approve/reject 계열 액션 호출. 결재는 사람 몫이다.
- Engine.cs·Storage.cs·Guardrails.cs에 도메인 지식(게임 용어, metricId, ollama 코드)을 넣는 것.

## 관례
- 주석은 한국어 기능 설명만 (파일 머리 1~2줄 + 함수 위 1줄). 함수 수정 시 주석 갱신.
- 커밋 전 git status로 bin/, obj/, history/ 미포함 확인.
- 코드 변경의 기본 경로는 dispatch/outbox다 — 사본에서 작업하고 diff를 제출하며, 반입은 사람이 한다.
  직접 수정 + 커밋은 예외이며 다음 경우에만 쓴다: ①관례·가이드 문서 자체(CLAUDE.md, AGENTS.md,
  AGENT-GUIDE.md, skills/, docs/) ②지시서에 "직접 경로"가 명시된 경우. 예외를 썼으면 작업 보고에
  사유를 남긴다.
- 지시서(#12부터)는 불변 제약을 인라인으로 싣지 않고 "이 지시서는 docs/directives/_header.md의
  불변 제약을 따른다" 1줄로 참조한다 — 전문은 docs/directives/_header.md에 있다(#11까지는 인라인).

## 스킬 라우팅
- /skills/common/ 은 모든 작업에서 읽는다.
- /skills/domains/ 는 이번 작업이 변경할 파일 경로가 스킬의 '트리거:'와 일치하는
  것만 읽는다. 일치하지 않는 도메인 폴더는 열지 않는다. 애매하면 읽지 않는다.
- 작업 보고(verification 문서)에 "참조한 스킬" 목록을 기록한다.

## 지시 게이트 (착수 전 자가 검사)
- 지시를 받으면 착수 전에 검사한다: ① 완료 기준이 검증 가능한가
  ② 대상 파일·범위가 특정되는가 ③ 기존 원칙·blueprint와 충돌하지 않는가.
- 부족하면 추측으로 진행하지 말고, 부족 항목을 **선택지 딸린 질문**으로
  되물은 뒤 대기한다. 질문 없이 추측 진행한 부분은 보고에 '추측 진행'으로 명시한다.
- 원칙과 충돌하면 "기준 변경입니까?"를 확인한다 — 기준 변경은 사람 결재 사항.

## 게이트 기록 형식
- 커밋 전 `dotnet run --project server -- measure dev-pack` 결과를 verification 문서에 JSON 한 줄로 기록한다.
- 형식: `{"gate":"dev-pack","violations":0,"attempt":1}`
- 위반이 남으면 `violations`에 실제 개수를 적고, 이어서 남은 위반 목록을 적는다.

---
> Source: [zheng0000000000000-art/loop-control-dashboard](https://github.com/zheng0000000000000-art/loop-control-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
