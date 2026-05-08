## harness-strategy-report

> **목표:** GitHub 오픈소스 도구(특히 Claude Code 플러그인, 에이전트 프레임워크)에 대한 인터넷 반응을 종합 조사하고 리서치 보고서를 생성한다.

# Harness Research — Claude Code 하네스

## 하네스: GitHub 오픈소스 도구 종합 리서치

**목표:** GitHub 오픈소스 도구(특히 Claude Code 플러그인, 에이전트 프레임워크)에 대한 인터넷 반응을 종합 조사하고 리서치 보고서를 생성한다.

**트리거:** 특정 GitHub 저장소·오픈소스 도구·AI 에이전트 프레임워크에 대한 "조사", "분석", "리서치 보고서", "인터넷 반응", "도입 검토", "경쟁사 비교" 요청이 들어오면 `github-tool-research` 오케스트레이터 스킬을 사용하라. 단순 README 읽기나 1회 API 호출은 직접 응답 가능.

**실행 모드:** 에이전트 팀 (팬아웃/팬인 + 편집자 패턴, 5명)

---

## 하네스 2: OSS Growth & Branding Strategy

**목표:** GitHub 오픈소스 도구(특히 Claude Code 플러그인·에이전트 프레임워크)의 Stars J 커브 성장 + 브랜딩 전략을 수립한다. 리서치 결과를 입력으로 받아 성장 모델·포지셔닝·GTM·프로덕트 로드맵·파트너십을 종합한 실행 전략 보고서를 출력한다.

**트리거:** "star 늘리기", "J 커브 전략", "OSS 브랜딩", "성장 전략", "GTM 전략", "포지셔닝 전략", "실행 전략 보고서", "{저장소} 성장 전략" 요청이 들어오면 `oss-growth-strategy` 오케스트레이터 스킬을 사용하라. 단순 아이디어 브레인스토밍이나 단일 채널 질문은 직접 응답 가능.

**실행 모드:** 에이전트 팀 (팬아웃/팬인 + 편집자 패턴, 6명 — growth-strategist, positioning-strategist, gtm-channel-planner, product-roadmap-advisor, community-partnership-scout, strategy-editor)

**사전 조건:** 기존 리서치 하네스(`github-tool-research`)의 산출물이 있어야 의미 있는 전략이 나온다. 없으면 리서치부터.

---

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-04-18 | 초기 구성 — 5 에이전트 + 6 스킬 + 오케스트레이터 | 전체 | revfactory/harness 리서치 요구로 구축 |
| 2026-04-18 | 하네스 2 추가 — 6 전략 에이전트 + 2 전문 스킬 + 오케스트레이터(oss-growth-strategy) | agents/{growth-strategist, positioning-strategist, gtm-channel-planner, product-roadmap-advisor, community-partnership-scout, strategy-editor}.md, skills/{growth-modeling, positioning-messaging, oss-growth-strategy} | revfactory/harness J 커브 성장·브랜딩 전략 수립 요구 |

---
> Source: [revfactory/harness-strategy-report](https://github.com/revfactory/harness-strategy-report) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
