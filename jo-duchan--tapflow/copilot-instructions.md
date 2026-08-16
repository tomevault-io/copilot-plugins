## tapflow

> 로컬 전용 작업 로그 디렉토리. 로그는 git에 커밋되지 않고, 규약인 이 파일만 추적된다 — 루트

# .work — CLAUDE.md

로컬 전용 작업 로그 디렉토리. 로그는 git에 커밋되지 않고, 규약인 이 파일만 추적된다 — 루트
`AGENTS.md`와 `INDEX.md`가 여기를 링크하기 때문이다.

---

## 파일 명명 규칙

```
YYYY-MM-DD-{topic}-{type}.md
```

예시: `2026-05-08-agent-core-interface-plan.md`

---

## 메타데이터 (frontmatter)

모든 파일 상단에 아래 형식을 포함한다.

```yaml
---
created: YYYY-MM-DD
status: draft | in-progress | done | blocked
type: plan | review | compound | todo
phase: 1 | 2 | 3 | 4 | 5 | 5+
topic: 한 줄 요약
---
```

| 필드 | 설명 |
|------|------|
| `created` | 파일 생성일 |
| `status` | 현재 진행 상태 |
| `type` | 워크플로우 단계 (plan / review / compound / todo) |
| `phase` | tapflow 로드맵 Phase (1–5+). 해당 없으면 생략 |
| `topic` | 작업 주제 한 줄 요약 |

---

## type별 용도

- **plan** — 요구사항 + 테스트 케이스 정의. 작업 시작 전 작성.
- **review** — 엣지 케이스 검토 + 실제 데이터 검증 결과.
- **compound** — 재사용할 테스트·코드·프롬프트 템플릿 묶음.
- **todo** — 스코프 밖으로 분리된 항목 모음. 작업 시작 시 plan으로 전환한다.

---

## 프로그램 단위 계획 — 날짜 규칙의 예외

여러 PR에 걸치는 작업은 **날짜 접두사 없는 대문자 파일 하나**로 두고 갱신한다 — 현재
`WIRE-CONTRACT-PLAN.md`.

PR 단위로 쪼개면 리뷰 안전성은 확보되지만 **전체 그림을 놓치고 미시적 문제에 매몰된다.**
2026-08-08에 그게 일어났다: 입력 경로 6건을 하루에 고치면서 설계 계획이 **세 번 폐기됐고, 세 폐기가
같은 모양**이었는데(구조 문제에 국소 패치) 두 번째에서 알아채지 못했다. 컨텍스트는 리셋되고, PR
계획은 PR 단위, 리뷰 기록은 브랜치 단위여서 **프로그램 단위인 것이 없었다.**

그 파일이 반드시 담아야 하는 것:

- **현재 상태** — 층/단계별 진행, 맨 위에. 먼저 읽는 것
- **다음 행동** — 하나
- **결정 로그** — 제안됐다가 폐기된 것과 그 근거. 컨텍스트가 리셋되면 같은 걸 또 제안하므로 이게
  핵심이다. 틀린 것으로 판명된 전제도 같이 남긴다
- **이슈 매핑** — 새 이슈를 만드는 대신 기존 이슈를 층에 붙인다

층이 랜딩하면 **영구적 설계 근거는 해당 AGENTS.md로 옮긴다.** 이 파일은 진행 상태와 결정을 들고,
근거는 코드 옆에 산다.

## 아카이브

`status: done`이 된 파일은 `archive/` 서브디렉토리로 이동한다.
`.work/` 최상위에는 진행 중(`draft` · `in-progress` · `blocked`) 파일만 유지한다. `archive/`와
`reviews/`는 그 아래에 있으므로 이 규칙의 대상이 아니다.

---

## reviews/ — adversarial review 기록

`reviews/<브랜치명>.md` (슬래시는 `__`로 치환). PR 생성 전 독립 컨텍스트 리뷰의 발견사항과 처리 내역(수정 / 스킵+사유)을 기록하고, 리뷰한 HEAD 커밋 해시를 포함한다. `.claude/hooks/adversarial-review-gate.sh`가 `gh pr create` 시 이 기록과 HEAD 일치를 강제한다. 절차 정의: 루트 AGENTS.md "Adversarial Review".

**수명은 브랜치와 같다.** 브랜치가 머지되면 기록도 `reviews/archive/`로 옮긴다. 위 아카이브 규칙에
`reviews/`가 빠져 있어서 2026-08-15 기준 61건 중 59건이 이미 머지됐거나 사라진 브랜치의 것이었다.
비용은 디스크가 아니라 다음 세션이 이 디렉터리를 훑을 때의 신호 대 잡음이다.

**커밋되는 문서에서 이 디렉터리를 링크하지 않는다.** `.work/`는 gitignore라 기여자에게는 없는
경로이고, 링크는 그쪽에서 끊긴 채로만 보인다. 같은 날 기준 61건 중 레포 안에서 인용된 유일한 1건이
`packages/ios-agent/AGENTS.md`의 그런 링크였다. 남길 가치가 있는 근거는 링크가 아니라 승격으로
옮긴다 — 코드 옆 주석, 패키지 AGENTS.md, `contributing/`.

---

## 시작 방법

`/work-plan {topic}` 커맨드로 plan 문서를 생성한다.

---
> Source: [jo-duchan/tapflow](https://github.com/jo-duchan/tapflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
