## my-harness

> 이 레포는 **하네스 팩토리**(도메인 한 문장 → 에이전트 팀 + 스킬 생성 메타 스킬)다. Claude Code와 Codex 양쪽에서 동작하도록 듀얼 런타임으로 구성돼 있다. 이 파일은 **Codex CLI**용 진입점이다(Claude Code는 `CLAUDE.md` + `.claude-plugin/` 사용).

# AGENTS.md — Codex 런타임 진입점

이 레포는 **하네스 팩토리**(도메인 한 문장 → 에이전트 팀 + 스킬 생성 메타 스킬)다. Claude Code와 Codex 양쪽에서 동작하도록 듀얼 런타임으로 구성돼 있다. 이 파일은 **Codex CLI**용 진입점이다(Claude Code는 `CLAUDE.md` + `.claude-plugin/` 사용).

## 하네스 팩토리 사용
- 새 도메인/프로젝트용 하네스를 만들거나 확장·점검하려면 **`skills/myharness/SKILL.md`를 읽고 그 워크플로우(Phase 0~7)를 따른다.**
- Codex 스킬 자동 활성화를 쓰려면 `install.sh`로 `.agents/skills/myharness`를 설치한다(그 후 "하네스 만들어줘" 등에 description 기반 활성화 + `/skills`·`$myharness` 명시 호출).

## Codex 런타임 어댑터 (중요)
팩토리 워크플로우는 기본이 Claude Code 팀 도구(`TeamCreate`/`SendMessage`)다. **Codex에선 다음으로 매핑한다:**
- 멀티 에이전트 → Codex 네이티브 subagents(내장 `default`/`worker`/`explorer` 또는 `.codex/agents/*.toml`), `/agent` 전환. 완전 독립 병렬은 `codex exec` subprocess.
- 팀원 통신/태스크 → `_workspace/` 파일 기반 전달(다음 단계가 Read).
- 에이전트 정의 → Claude는 `.claude/agents/*.md`, Codex는 `.codex/agents/*.toml`.
- 스킬 → Claude `.claude/skills/`, Codex `.agents/skills/` (SKILL.md 포맷 동일).
- 상세: `skills/myharness/references/runtime-adapters.md`, 오케스트레이션은 `references/orchestrator-template.md` 템플릿 D.

## 외부 리뷰 게이트
`skills/myharness/references/external-review-loop.md` — codex/gemini 독립 검증. 양쪽 런타임 동일하게 subprocess로 작동. `skills/myharness/scripts/check-review-tools.sh`로 도구 연동 점검(없으면 게이트 생략).

## codex exec 베스트 프랙티스 (자동화 시)
기본 `--sandbox read-only` / 쓰기만 `workspace-write` / 스크립트 소비 `--json` / 최종 메시지만 `-o` / 격리 `--ignore-user-config` / stdin은 `< /dev/null`.

## 변경 이력
| 날짜 | 변경 내용 | 사유 |
|------|----------|------|
| 2026-06-10 | Codex 듀얼 런타임 진입점 신설 | Claude Code + Codex 양쪽 설치 지원 |

---
> Source: [cookyman74/my_harness](https://github.com/cookyman74/my_harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
