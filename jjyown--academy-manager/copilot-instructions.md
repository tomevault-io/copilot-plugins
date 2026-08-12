## workflow-doc-driven

> Always use docs workflow for continuity and quality


# Document-Driven Workflow

Before coding, read these files in order:
- `docs/plan.md`
- `docs/context.md`
- `docs/checklist.md`

## Mandatory Workflow

1. Restate the current task in one or two sentences.
2. Propose a short implementation plan aligned with `docs/plan.md`.
3. Implement in small steps and report changed files clearly.
4. Update document base date first (HARD REQUIREMENT):
   - 작업 시작 즉시 `python qa-artifacts/sync_doc_dates.py`를 실행해 날짜를 자동 동기화한다.
   - `docs/plan.md`, `docs/context.md`, `docs/checklist.md`의 `문서 기준일`을 **시스템 오늘 날짜**로 먼저 통일한다.
   - 날짜는 고정값이 아니라 매 작업일마다 갱신한다. 예: 오늘 2026-03-06 저장, 다음날 작업 시 2026-03-07로 재저장.
   - 날짜가 하나라도 다르면 구현/응답을 진행하지 않는다.
5. Update documents before final response:
   - `docs/plan.md`: status and next step
   - `docs/context.md`: key decisions and reasons
   - `docs/checklist.md`: checked and unchecked items
6. Run date consistency check before final response:
   - 세 문서의 `문서 기준일`이 동일한지 확인하고, 불일치 시 반드시 수정 후에만 종료한다.
7. If any checklist item is skipped, explain why.

## Date Logging Guard (강화 규칙)

- `문서 기준일`만 맞추는 것으로 끝내지 않는다.
- `docs/plan.md`, `docs/context.md`, `docs/checklist.md`에 **새로 추가하는 모든 작업 기록/로그/변경 이력의 날짜도 시스템 오늘 날짜와 일치**해야 한다.
- 과거 날짜를 복붙해서 신규 항목 날짜로 사용하면 안 된다.
- 신규 기록 날짜가 오늘이 아니면:
  1) 구현 작업 중단
  2) 날짜 교정
  3) 교정 확인 후에만 작업 재개
- 최종 응답 전 아래 2가지를 모두 만족해야 한다:
  1) 세 문서 `문서 기준일` 완전 일치 + 오늘 날짜
  2) 이번 작업 사이클에서 추가한 모든 로그/이력 날짜가 오늘 날짜

## Hard Stop Rule

- 다음 중 하나라도 위반 시 절대 최종 응답 금지:
  - 세 문서 `문서 기준일` 불일치
  - `문서 기준일`이 오늘 날짜가 아님
  - 이번 사이클에 추가한 작업 기록 날짜가 과거 날짜

## Quality Gate

- Run or describe relevant verification after edits.
- Explicitly mention risks, assumptions, and follow-up work.
- Avoid changing direction silently; record scope changes in `docs/context.md`.
- NEVER finalize while `문서 기준일` mismatch exists in docs.

---
> Source: [jjyown/academy_manager](https://github.com/jjyown/academy_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
