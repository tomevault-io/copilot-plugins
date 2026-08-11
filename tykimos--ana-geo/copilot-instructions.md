## ana-geo

> ANA(Agent-Native Agent, github.com/tykimos/agent-native-agent) 기반 Agent-Native GIS 앱 패밀리. 요구사항은 `PRD.md`, 앱 샘플은 `apps/*`에 배치 예정.

# ANA Geo

ANA(Agent-Native Agent, github.com/tykimos/agent-native-agent) 기반 Agent-Native GIS 앱 패밀리. 요구사항은 `PRD.md`, 앱 샘플은 `apps/*`에 배치 예정.

## 하네스: PRD 검수

**목표:** PRD.md가 구조적으로 완결되고, ANA 베이스 원칙과 정합하며, 명시된 GIS 스택으로 실현 가능한지 검수하여 `PRD-REVIEW.md`를 산출한다.

**트리거:** PRD 검수/리뷰/감사/재검수/보고서 갱신 요청 시 `prd-review` 스킬을 사용하라. PRD 내용에 대한 단순 질문은 직접 응답 가능. PRD 집필/수정 자체는 하네스 범위가 아니다.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-08-10 | 초기 구성 (검수자 3 + 리포터 1, 팀 모드) | 전체 | - |
| 2026-08-10 | finding 스키마 개편 (scope 래퍼, summary/crossref/handoff 필드), 인용·severity 규칙 명확화, 경로 우선순위 규칙 추가 | skills/prd-review-criteria, agents/* | 스모크 테스트 피드백 8건 반영 |
| 2026-08-10 | confidence 적용 대상(본체 vs 권고 세부)·handoff 미회신 마감 규칙 명문화 | skills/prd-review-criteria | 라운드 1 실행 중 structure 검수자의 정당한 지시 이탈 사례 일반화 |
| 2026-08-10 | 교차 대조표에 §16.3↔§17.4 카테고리 어휘 쌍 추가 | skills/prd-review-criteria/references/structure-checklist.md | 라운드 1에서 구조 축이 놓친 어휘 불일치(STR-038) — 대조표 공백이 원인 |
| 2026-08-11 | 개정 재검수용 "신설 절 양방향 접합 점검"(체크리스트 4-1) 추가 | skills/prd-review-criteria/references/structure-checklist.md | 라운드 2 신규 결함 대부분이 신설 절 미접합 유형 (structure 제안) |
| 2026-08-11 | 공통 규칙 5 "지적 대상은 정의 문장" 명문화 | skills/prd-review-criteria | 라운드 2에서 검수자 프레이밍 오류 2건이 교차 반증으로 철회된 사례 일반화 |
| 2026-08-11 | 체크리스트 4-1에 항목 3 "제거의 유효성"(제거 전 타 절 요구 확인) 추가 | skills/prd-review-criteria/references/structure-checklist.md | 라운드 3에서 v1.2의 §22 ✓ 제거가 §18/§33 요구와 충돌한 사례(STR-050) 일반화 |
| 2026-08-11 | "병합 정본 귀속 기본값"(선제안 우선, 선호 기반 반대 금지) 규칙 추가 | skills/prd-review-criteria | 라운드 3 STR-044↔ANA-014 정본 왕복 3회 사례 (structure 제안) |
| 2026-08-11 | 정본 규칙에 "라벨 이견은 선구현 우선" 보완, 공통 규칙 2에 "의도의 근거 검증" 보완 | skills/prd-review-criteria | ana(왕복 원인은 수락 교차)·gis("by design"의 §18.6 인용 불성립 사례) 제안 |

---
> Source: [tykimos/ana-geo](https://github.com/tykimos/ana-geo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
