## datamonitor-umja-3228

> 이 저장소는 "반도체 시료 생산주문관리 시스템" 개인과제의 **미션1 PoC** 중 하나다(데이터 모니터링 도구 검증).

# CLAUDE.md — DataMonitor PoC

이 저장소는 "반도체 시료 생산주문관리 시스템" 개인과제의 **미션1 PoC** 중 하나다(데이터 모니터링 도구 검증).

## 1. 모델/평가 제약 (최우선)
사용 모델: **Sonnet**, Effort: **Medium** 고정. 다른 모델 사용 이력이 있으면 부정행위로 간주된다.

## 2. 이 PoC의 목적과 범위
- 목적: `DataPersistence` PoC와 동일한 JSON 저장 포맷을 읽어, 현재 저장된 Sample/Order raw 데이터를 콘솔에 조회하는 읽기 전용 관리자 도구 구현 (`docs/FEATURES/08-data-monitor-tool.md`).
- 이 도구는 본 시스템의 "모니터링" 기능(상태별 집계/재고 상태 판정, 미션2)과 다르다 — raw 데이터 확인이 목적이다.
- 미션1이므로 TDD를 엄격히 강제하지 않는다(Vibe Coding 허용). 단, 실제 JSON 파일을 두고 정상 조회되는지 반드시 시연 검증한다.

## 3. 커밋 워크플로우 (반드시 준수)
작업이 끝나도 절대 바로 `git commit`을 실행하지 않는다.
1. 변경 사항을 정리해 커밋 메시지 초안을 사용자에게 제시한다.
2. 사용자가 "확인 내용 메시지"와 함께 승인하면, 아래 형식으로 커밋한다.

```
[Phase#] 변경 사항 제목

요구사항: <관련 docs/FEATURES/*.md 경로>
변경 사항:
- <핵심 변경 1>
- <핵심 변경 2>

테스트: <자동화 테스트가 있으면 몇 개 중 몇 개 통과했는지 수치로, 없으면 실제 검증한 시나리오를 구체적으로>

Reviewed-by: junyong.uhm (<사용자가 전달한 확인 내용 메시지>)
```

- 커밋 메시지 첫 줄(제목)은 항상 `[Phase#] 변경 사항 제목` 형식으로 작성한다.
- 본문은 `요구사항`/`변경 사항`/`테스트` 3개 섹션을 고정 순서로 포함한다. `테스트`는 "확인함" 같은 뭉뚱그린 표현 대신 무엇을 어떻게 검증했는지 구체적으로 적는다.
- 사용자의 명시적 승인 문구 없이 커밋하지 않는다.
- **주의**: 이 `[Phase#]`는 **이 저장소 자체의 진행 단계(1부터 시작)**다. 워크스페이스 루트 `PLAN.md`의 전역 로드맵 Phase 번호(저장소를 가로지르는 조율용 번호)와는 다르니 혼동하지 않는다.

## 4. Skill / Agent
- **`spec-reviewer`** 서브에이전트(`.claude/agents/spec-reviewer.md`): 구현이 `docs/FEATURES/08-data-monitor-tool.md`와 일치하는지 확인하는 read-only 감사 에이전트. **커밋 메시지 초안을 만들기 전에 반드시 호출**한다.
- `code-review`/`simplify`/`verify` 등 내장 스킬은 필요 시 자유롭게 사용한다.
- 이 저장소는 PoC이므로 `test-driven-development` 스킬은 등록하지 않는다(미션2 전용).

## 5. Phase 완료 체크리스트
1. 콘솔 실행으로 시연 (데이터 없음/있음 케이스 모두 확인)
2. `spec-reviewer` 호출 → PASS 확인
3. 커밋 메시지 초안 제시 → 사용자 승인 대기 → 승인 시 커밋

## 6. 문서
- `docs/FEATURES/08-data-monitor-tool.md` — 이 PoC의 요구사항
- `docs/FEATURES/07-data-persistence.md` — 참고할 저장 포맷 명세
- `docs/DOMAIN_MODEL.md` — 전체 과제의 공통 도메인 배경 (참고용)

---
> Source: [JunyongUhm/DataMonitor-UMJA-3228](https://github.com/JunyongUhm/DataMonitor-UMJA-3228) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
