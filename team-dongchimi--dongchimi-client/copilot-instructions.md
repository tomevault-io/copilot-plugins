## dongchimi-client

> 이 문서는 agent용 허브입니다. 상세 규칙을 이 파일에 복붙하지 말고, 작업 종류에 맞는 source of truth 문서를 먼저 읽습니다.

# DONGCHIMI-CLIENT Agent Guide

이 문서는 agent용 허브입니다. 상세 규칙을 이 파일에 복붙하지 말고, 작업 종류에 맞는 source of truth 문서를 먼저 읽습니다.

## 저장소 정체

- Product: 동치미 클라이언트
- Jira root key: `DCMFE-*`
- Client web key: `DCMCL-*`
- Market owner web key: `DCMSM-*`
- Design system key: `DCMDS-*`
- Design system web key: `DCMDSW-*`
- 현재 app: `apps/client`, `apps/market-owner`
- 예정 app: `apps/design-system-web`, 선택적 `apps/admin`, `apps/mobile`
- 현재 package: `packages/design-system`, `packages/eslint-config`, `packages/typescript-config`
- 예정 package: `packages/shared`, `packages/tailwind-config`
- Package manager, app layout, build system은 실제 `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `docs/architecture/*`를 source of truth로 봅니다.

## 기본 순서

1. 요구사항 분석과 설계 -> Jira 이슈 작성 -> Jira 이슈 티켓 발급
2. Jira 진행중 상태 변경 -> turbo gen으로 파일 생성 -> Jira 이슈 내용 기반 page, components, hooks 등 spec 문서 작성
3. 실제 컴포넌트, 페이지, 기능 등 구현 시작
4. 구현 내용 확인 및 검토
5. 추가 작업
   - 컴포넌트: story 추가 및 동작 검증
   - hook: 필요하면 단위 테스트 및 동작 검증
   - page: 로컬 route 확인 및 검증
6. 구조 개선
7. 커밋 계획 수립 -> 기능/작업 단위 커밋 -> 푸시 및 PR 작성
8. Jira 이슈 리뷰중 상태 변경 -> 코드 리뷰
9. 머지 및 Jira 이슈 진행상황 완료 변경

Jira/Figma/사진 기반 작업은 항상 `jira-design-implementation-workflow`를 먼저 적용합니다. Jira 이슈 작성만 요청받은 경우에는 구현을 시작하지 않습니다.

## 먼저 볼 문서

비자명한 작업은 아래 문서를 먼저 확인합니다. 단순 오탈자나 한 줄 변경은 필요한 문서만 비례해서 봅니다.

- `README.md`, `docs/index.md`, `docs/code-quality/index.md`
- `docs/architecture/repo-structure.md`
- `docs/conventions/git.md`, `docs/conventions/package-management.md`
- `docs/workflows/jira-issue-authoring.md`, `docs/workflows/spec-writing.md`, `docs/workflows/testing.md`, `docs/workflows/turbo-generators.md`, `docs/workflows/sentry.md`, `docs/workflows/pr-checklist.md`, `docs/workflows/pull-request-writing.md`

## 작업 유형별 라우팅

| 작업 유형                                  | 먼저 볼 문서                                                                                                                                                                                              |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Jira 이슈 생성, 보강, parent/sub-task 분해 | `docs/workflows/jira-issue-authoring.md`, `.agents/skills/jira-design-implementation-workflow/SKILL.md`, `templates/jira-issue-template.md`                                                               |
| Jira/Figma/사진 기반 작성 또는 착수        | `.agents/skills/jira-design-implementation-workflow/SKILL.md`, `recipes/jira-design-to-implementation.md`                                                                                                 |
| 프론트엔드 작업 오케스트레이션             | `.agents/skills/frontend-task-orchestrator/SKILL.md`, 가까운 recipe/spec                                                                                                                                  |
| 페이지 또는 라우트                         | `.agents/skills/page-feature-workflow/SKILL.md`, `recipes/add-page.md`, `templates/page.spec.md`                                                                                                          |
| 앱 shared 컴포넌트                         | `.agents/skills/app-shared-component-workflow/SKILL.md`, `recipes/add-app-shared-component.md`, `templates/component.spec.md`                                                                             |
| 앱 로컬 컴포넌트                           | `recipes/add-component.md`, 필요 시 `.agents/skills/refactor-evaluator/SKILL.md`                                                                                                                          |
| 디자인시스템 컴포넌트                      | `.agents/skills/design-system-component-workflow/SKILL.md`, `recipes/add-design-system-component.md`                                                                                                      |
| hook 또는 상태 로직                        | `recipes/add-hook.md`, 필요 시 `.agents/skills/refactor-evaluator/SKILL.md`                                                                                                                               |
| API query 또는 mutation                    | `.agents/skills/api-integration-workflow/SKILL.md`, `recipes/add-api-query.md`                                                                                                                            |
| form 또는 validation 흐름                  | `.agents/skills/form-flow-workflow/SKILL.md`, `recipes/add-form.md`                                                                                                                                       |
| refactor 판단                              | `.agents/skills/refactor-evaluator/SKILL.md`, `recipes/refactor-component.md`                                                                                                                             |
| 성능 이슈                                  | `.agents/skills/performance-diagnosis/SKILL.md`                                                                                                                                                           |
| 코드 품질 리뷰                             | `.agents/skills/frontend-fundamentals-review/SKILL.md`, `docs/code-quality/index.md`, `docs/code-quality/frontend-fundamentals.md`, `recipes/frontend-fundamentals-review.md`                             |
| 구조 판단                                  | `.agents/skills/architecture-review/SKILL.md`, `recipes/repo-orientation.md`                                                                                                                              |
| Turbo generator 추가/수정                  | `.agents/skills/turbo-generator-workflow/SKILL.md`, `docs/workflows/turbo-generators.md`                                                                                                                  |
| Sentry 설정 또는 에러 알림                 | `docs/workflows/sentry.md`, `docs/conventions/package-management.md`, `docs/workflows/local-development.md`                                                                                               |
| skill 유지보수                             | `.agents/skills/manage-skills/SKILL.md`, `docs/agent/index.md`, `docs/agent/indexing.md`                                                                                                                  |
| 브라우저 PR 리뷰                           | `.agents/skills/browser-pr-review-workflow/SKILL.md`, `docs/workflows/pull-request-writing.md`, `docs/workflows/pr-checklist.md`                                                                          |
| 프로젝트 상태 모니터링                     | `.agents/skills/project-monitoring-workflow/SKILL.md`, 필요한 Jira/PR/Notion/check 근거                                                                                                                   |
| 구현 검증                                  | `.agents/skills/verify-implementation/SKILL.md`, `.agents/skills/frontend-quality-verification/SKILL.md`                                                                                                  |
| 커밋 분해 또는 PR 준비                     | `.agents/skills/commit-planning-workflow/SKILL.md`, `recipes/commit-plan.md`, `recipes/pr-prep.md`, `docs/conventions/git.md`, `docs/workflows/pr-checklist.md`, `docs/workflows/pull-request-writing.md` |

## Source Of Truth

- 문서 인덱스: `docs/index.md`
- agent harness와 skill indexing: `docs/agent/index.md`, `docs/agent/indexing.md`
- 코드 품질 기준: `docs/code-quality/index.md`, `docs/code-quality/frontend-fundamentals.md`
- 아키텍처와 workspace 경계: `docs/architecture/*`
- Git, package manager 규칙: `docs/conventions/*`
- 결정 로그 기준: `docs/decisions/index.md`
- Jira 이슈 작성과 분해 기준: `docs/workflows/jira-issue-authoring.md`
- 스펙 작성 기준: `docs/workflows/spec-writing.md`, `templates/*.spec.md`
- 반복 작업 절차: `recipes/*`
- turbo generator 기준: `docs/workflows/turbo-generators.md`
- Sentry 설정과 알림 운영 기준: `docs/workflows/sentry.md`
- 재사용 가능한 agent workflow: `.agents/skills/*/SKILL.md`

## 필수 작업 원칙

- 추측보다 로컬 파일, Jira, Figma, 첨부 이미지, 기존 코드 근거를 우선합니다.
- 검색은 우선 `rg` 또는 `rg --files`를 사용합니다.
- 수동 파일 수정은 `apply_patch`를 사용합니다.
- app/package 구조가 확정되기 전에는 새 추상화나 공유 패키지를 문서만으로 확정하지 않습니다.
- 요청 범위 밖 기능, 추상화, refactor를 추가하지 않습니다.
- 구현 전에 스펙 필요 여부를 `docs/workflows/spec-writing.md`로 판단합니다.
- Jira/Figma/사진 기반 작업은 Jira 작성, 진행중 전환, generator/scaffold, 스펙 작성, 구현, 개선, 검증, PR 준비 순서를 지킵니다.
- app/package 구조가 실제로 생기기 전에는 generator 명령과 workspace 검증을 문서상 절차로만 유지하고 실행 가능하다고 가정하지 않습니다.
- 커밋을 지시받으면 먼저 실제 diff를 기준으로 커밋 계획을 세웁니다. 커밋 수를 1~2개로 제한하지 말고, 독립적으로 리뷰/되돌리기/검증 가능한 기능 또는 작업 단위로 필요한 만큼 나눕니다.

## 검증

문서만 변경한 경우 최소 검증:

```bash
git diff --check
```

프론트엔드 앱 또는 package script가 생긴 뒤에는 `docs/workflows/local-development.md` 기준으로 format, lint, typecheck, build 검증을 추가합니다.

## 완료 후 확인

- Jira 또는 요청의 완료 기준을 충족했는지 확인합니다.
- 필요한 spec, docs, recipes, templates를 함께 갱신했는지 확인합니다.
- unused import, dead code, 임시 산출물이 남지 않았는지 확인합니다.
- 실행한 검증과 생략한 검증의 이유를 최종 요약에 남깁니다.

## 하지 말 것

- `AGENTS.md`에 긴 구현 규칙 원문, 현재 상태 기록, 과거 작업 로그를 넣지 않습니다.
- `docs/*` 내용을 그대로 복붙해서 중복 source of truth를 만들지 않습니다.
- 앱/package 구조가 확정되지 않았는데 문서에서 특정 프레임워크, package manager, monorepo 도구를 강제하지 않습니다.
- 중요한 불변량을 문서 안내만으로 끝내지 않습니다. 필요하면 lint, test, CI 강제를 별도 작업으로 분리합니다.

---
> Source: [TEAM-DONGCHIMI/DONGCHIMI-CLIENT](https://github.com/TEAM-DONGCHIMI/DONGCHIMI-CLIENT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
