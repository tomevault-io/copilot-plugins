## bmad-2d-monitor

> 이 규칙은 `bmad-2d-monitor` 프로젝트를 작업하는 모든 AI 에이전트(Antigravity 등)가 모든 과정에서 **최우선으로 준수해야 하는 강제 사항(Rule)**입니다. 어떠한 경우에도 이 룰을 무시하거나 우회해서는 안 됩니다.

# AI 에이전트 핵심 행동 규칙 (Harness Constraints)

이 규칙은 `bmad-2d-monitor` 프로젝트를 작업하는 모든 AI 에이전트(Antigravity 등)가 모든 과정에서 **최우선으로 준수해야 하는 강제 사항(Rule)**입니다. 어떠한 경우에도 이 룰을 무시하거나 우회해서는 안 됩니다.

## 1. 깃(Git) 워크플로우 강제 룰 (Branching & PR)
1. **항상 동기화 (Fetch Latest):** 어떠한 코드 수정 작업이든 시작하기 전에는 가장 먼저 `git fetch origin`과 `git switch main`(혹은 기준 브랜치) 및 `git pull`을 실행하여 로컬 상태를 최신 원격 상태로 유지해야 합니다.
2. **새 브랜치 격리 작업:** `main` 브랜치에 직접 코드를 작성하거나 커밋(Commit)하는 행위를 절대 금지합니다. 반드시 작업 내용에 부합하는 목적(예: `feature/dashboard-ui`, `fix/canvas-bug`)을 담은 새로운 브랜치를 생성(`git checkout -b feature/...`)하여 작업해야 합니다.
3. **무조건 PR (Pull Request) 지향:** 작업 완료 후, 코드를 해당 브랜치로 커밋 및 리모트에 푸시(`git push origin 브랜치명`)하고, GitHub에 Pull Request(PR)를 작성하도록 사용자에게 제안하거나 스스로 CLI를 통해 PR을 생성해야 합니다.

## 2. 의사결정 및 개발 단계 (BMAD 방법론 기반)
- **문서(PRD/아키텍처) 기반:** 작업을 진행하기 전에는 반드시 `docs/PRD.md` 및 `docs/ARCHITECTURE.md`를 열람하여 기획 의도와 기술 스펙을 숙지합니다. 문서에 위배되는 방향으로 아키텍처를 임의 수정하지 않습니다.
- **선 계획 후 실행:** 중요한 기능이나 여러 파일이 수정되는 작업을 할 경우 즉각적인 코드 수정(Vibe Coding)을 지양하고, 먼저 아티팩트로 계획(Plan)을 세우고 사용자의 검토를 거칩니다.

## 3. 코드 품질 보증 및 클린 코드 (Clean Code)
- **Type Safety (TS First):** TypeScript를 엄격하게 적용하며, `any` 타입의 사용은 절대 금지합니다. 모든 DTO, API 응답, 상태 구조는 명확한 `interface`나 `type` 선언에 의존해야 합니다.
- **순수 함수와 단일 책임 (SRP):** 하나의 컴포넌트나 함수가 여러 책임을 갖지 않게 짧고 간결하게 분리합니다. React 등 UI 개발 시 비즈니스 상태 패칭(Hooks)과 렌더링 뷰(Presentation)를 물리적 혹은 논리적으로 분리합니다.
- **예측 가능한 에러 핸들링:** 빈 `catch` 블록이나 단순 `console.log` 오류 출력을 금지합니다. 에러가 발생한 경위와 사용자에게 보여질 명백한 알림 또는 폴백(Fallback) UI를 고려해야 합니다.

## 4. 자가 검증 (Self-Correction & Feedback Loop)
- **오류 발생 후 스스로 치유:** 테스트 통과, 린트 에러, 컴파일 에러 발생 시 사용자에게 바로 질문하기 전에 AI 본인이 에러 로그를 읽고 수정 시도를 먼저 수행해야 합니다.
- **자동 검증 우선:** 코드 뭉치 작성이 완료되면 관련되어 설정된 `.agent/workflows/ci-check.md`의 절차 혹은 `npm run lint`, `npm run test`를 스스로 실행하여 작업 안정성을 확보합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ldk-hub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
