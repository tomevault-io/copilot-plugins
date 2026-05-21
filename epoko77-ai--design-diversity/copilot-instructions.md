## design-diversity

> **목표:** Claude가 산출하는 PPT·웹사이트 디자인의 천편일률 문제를 해결한다. 공개된 디자인 시스템·비주얼 스타일을 수집·증류해, Claude Code에 복붙하면 그 스타일이 재현되는 '디자인 팩' 카탈로그(1차 출범 40팩 = PPT 20 + 웹 20)를 만들고, baseline 대조로 다양성을 검증한 뒤 GitHub 공개 레포 + 카탈로그 웹사이트로 발행한다.

# Design Diversity

## 하네스: Design Diversity

**목표:** Claude가 산출하는 PPT·웹사이트 디자인의 천편일률 문제를 해결한다. 공개된 디자인 시스템·비주얼 스타일을 수집·증류해, Claude Code에 복붙하면 그 스타일이 재현되는 '디자인 팩' 카탈로그(1차 출범 40팩 = PPT 20 + 웹 20)를 만들고, baseline 대조로 다양성을 검증한 뒤 GitHub 공개 레포 + 카탈로그 웹사이트로 발행한다.

**트리거:** 디자인 팩 카탈로그 제작·확장·검증·발행 작업 요청 시 `design-diversity` 스킬을 사용하라. "디자인 팩 추가", "특정 팩만 다시", "다양성 검증 다시", "스키마 변경", "카탈로그 사이트 재배포" 등 후속 요청도 동일 스킬. 단순 질문은 직접 응답 가능.

**산출물 형식 (확정):** 각 팩 = `design-packs/{slug}/` 안의 `prompt.md` + `tokens.json` + `preview.png` + `meta.yaml`. 규격 SSOT는 `design-pack-schema` 스킬.

**팀 구성:** 에이전트 팀(7인) — design-scout · pack-architect · ppt-pack-curator · web-pack-curator · sample-renderer · diversity-qa · catalog-publisher. 스킬 8개(오케스트레이터 `design-diversity` + 컴포넌트 7). 상세는 `.claude/agents/`·`.claude/skills/`가 SSOT.

**발행:**
- GitHub: https://github.com/epoko77-ai/design-diversity (공개)
- 사이트(production): https://design-diversity.vercel.app — Vercel 프로젝트 `site` (team epoko77-9135), Deployment Protection 해제됨
- 카탈로그: 80팩 (PPT 40 + 웹 40), 전부 status pass. PPT는 차트 + 다이어그램·컴포넌트 키트 포함

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-05-20 | 초기 구성 (7 에이전트 + 8 스킬) | 전체 | - |
| 2026-05-20 | 1차 풀빌드 — 40팩 + 레포 + 사이트 발행 | catalog | - |
| 2026-05-20 | ppt-design-idioms에 전문 차트 기준 추가 | skills/ppt-design-idioms | PPT 차트 허접 피드백 |
| 2026-05-20 | 프리미엄 브랜드 스타일 20팩 추가 (40→60), PPT 20팩 차트 재작업·재렌더 | catalog | 전문성·고급화 피드백 |
| 2026-05-21 | ppt-design-idioms에 다이어그램·컴포넌트 키트 기준 추가 | skills/ppt-design-idioms | PPT가 색·분위기만 있고 다이어그램 양식 부재 피드백 |
| 2026-05-21 | 신규 20팩 추가 (60→80), 기존 30 PPT 다이어그램 키트 보완·재렌더, 사이트 PPT/웹 구분 강화 + 사용법 안내 추가 | catalog, site | 다이어그램 보강·사용법 피드백 |

---
> Source: [epoko77-ai/design-diversity](https://github.com/epoko77-ai/design-diversity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
