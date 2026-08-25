## wedding-letter

> **목표:** 2025-2026 트렌드를 반영한 한국 모바일 단일 페이지 청첩장(index.html + images/)을 컨셉부터 QA까지 일관되게 제작한다. 이미지는 gpt-image2(글로벌 스킬)로 생성한다.

# Wedding Invitation Letter — 프로젝트 지침

## 하네스: 한국 모바일 웨딩 청첩장 제작

**목표:** 2025-2026 트렌드를 반영한 한국 모바일 단일 페이지 청첩장(index.html + images/)을 컨셉부터 QA까지 일관되게 제작한다. 이미지는 gpt-image2(글로벌 스킬)로 생성한다.

**트리거:** 청첩장 제작·수정·재실행 관련 모든 요청에 `wedding-invitation` 스킬을 사용하라. 단순 질문(예: "어떤 폰트가 좋아?")은 직접 응답 가능하지만, 산출물 생성/수정이 동반되면 반드시 오케스트레이터를 통해 진행한다.

**팀 구성:** 6명의 전문가 에이전트
- `concept-director` — 컨셉/톤앤매너/팔레트/타이포 결정
- `copy-writer` — 인사말, 부모 함자, 안내문 등 한국어 카피
- `image-artist` — gpt-image2로 히어로/플로럴/장식 이미지 생성
- `motion-designer` — 인터랙티브 요소 + 애니메이션 배경 명세 (Phase 2 병렬)
- `frontend-developer` — 모바일 우선 단일 HTML + 모션 명세 구현
- `invitation-qa` — 정합성/반응형/접근성/인터랙션/모션 구현도/reduced-motion 분기 검증

**주요 산출물:**
- `index.html` (메인 결과물)
- `images/` (이미지 자산)
- `_workspace/` (중간 산출물 — 부분 재실행 시 참고됨)

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-04-29 | 초기 구성 | 전체 (5 agents + 6 skills + CLAUDE.md) | 신규 하네스 구축 |
| 2026-04-29 | motion-designer 에이전트 + wedding-motion-design 스킬 추가, frontend/QA 모션 통합 | agents/motion-designer.md, skills/wedding-motion-design, skills/wedding-mobile-frontend, skills/wedding-invitation-qa, skills/wedding-invitation, agents/frontend-developer.md, agents/invitation-qa.md | 사용자 요청: 인터랙티브 요소 + 애니메이션 배경 추가 |

---
> Source: [revfactory/wedding-letter](https://github.com/revfactory/wedding-letter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
