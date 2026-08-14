## noteboard

> **먼저 `PROJECT_SPEC.md`를 읽을 것.** 이 파일은 요약일 뿐, 지시서가 우선한다.

# CLAUDE.md — noteboard

**먼저 `PROJECT_SPEC.md`를 읽을 것.** 이 파일은 요약일 뿐, 지시서가 우선한다.

## 절대 규칙

- **GPL / AGPL 의존성 추가 금지. `@blocknote/xl-*` 사용 금지** (GPL 듀얼 라이선스).
- 의존성을 추가하면 반드시 `NOTICE` 갱신 + `npm run license:check` 통과 확인.
- BlockNote(MPL-2.0) 원본 파일을 fork/수정하지 말 것 — 래핑과 확장 API만.
- BlockNote는 0.x — **API를 추측으로 호출하지 말고** `node_modules/@blocknote/*/dist/*.d.ts`와
  blocknotejs.org로 확인 후 사용.

## 작업 방식

- 한 번에 하나의 마일스톤. 시작 전 계획 제시 → 사용자 확인 → 진행 → 완료 보고 후 정지.
- 커밋 메시지는 영어, push는 사용자가 명시적으로 요청할 때만.
- 한글 IME 입력 검증은 에디터 관련 변경마다 챙길 것 (사용자가 실기기에서 확인).

---
> Source: [fbdeme/noteboard](https://github.com/fbdeme/noteboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
