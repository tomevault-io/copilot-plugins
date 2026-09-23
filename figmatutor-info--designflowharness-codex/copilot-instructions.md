## designflowharness-codex

> 이 저장소는 Codex에서 사용하는 Figma UI 생성 하네스다. 시작할 때 `npm run status`와

# Design Flow Harness — Codex

이 저장소는 Codex에서 사용하는 Figma UI 생성 하네스다. 시작할 때 `npm run status`와
`harness.config.json`을 읽는다. 이 저장소의 AGENTS.md, 프로젝트 스킬, JSON 계약만
활성 지침·명령·완료 판정으로 사용한다. 원본 프로젝트 경로에 쓰지 않는다.

## 흐름

요구사항·레퍼런스 → 최소 토큰 → 같은 대표 화면 시안 2~3개 → 사용자 방향 선택 →
선택 방향으로 컴포넌트 정리 → 전체 화면·상태 확장 → 구조·시각 검증.

- JSON 계약은 기계 판정의 원본이다. 필드는 `docs/contracts.md`, 게이트는 `scripts/lib/gates.mjs` 참조.
- `npm run status`는 다음 미완료 단계를 보여 준다. `npm run check -- --phase NAME`은 선행 단계도 검사한다.
- 방향 선택 전 Figma에 대표 시안을 그릴 수 있다. 전체 컴포넌트 라이브러리를 먼저 만들지 않는다.
- 최소 토큰은 출발점이다. 시안마다 primitiveOverrides로 차이를 기록하고 색만 바꾼 복제본 대신 구조·밀도·타이포의 차이를 비교한다.
- Figma 토큰 라이브러리 문서는 `docs/token-library-guide.md`의 지정 레퍼런스와 제작·검증 절차를 따른다. 전체 문서 정리는 방향 선택 이후에 한다.
- Figma 컴포넌트 라이브러리는 `docs/component-library-guide.md`의 지정 레퍼런스를 참고해 카테고리별로 정리하고 실제 마스터·상태·사용 예시를 검증한다. 방향 선택 이후에 적용한다.
- 사용자 선택을 임의로 만들지 않는다. 방향을 선택한 실제 발언이 있으면 `npm run select`로 기록한다. 이미 받은 승인을 다시 묻지 않는다.
- 방향 선택 외 단계에서 일률적으로 승인 대기를 추가하지 않는다. 누락 정보가 실제로 작업을 막는 경우만 질문한다. 외부 변경·비용은 세션의 승인 범위를 따른다.
- 선택 이후 입력이 달라지면 해시로 stale 판정한다. 영향 설명 후 실제 사용자 재선택/확인을 기록한다. 해시만 수동 교체해 우회하지 않는다.
- 이미지 생성 단계를 사용하지 않는다. 캐릭터 이미지에는 `design/characters`의 보유 원본만 FIT로 배치하고 남는 영역은 `color-character-bg` 파스텔 하늘색으로 채운다.
- 브랜드 로고가 표시되는 헤더에는 `design/characters/Logo.svg` 원본을 사용한다. 텍스트나 다른 글꼴로 재현하지 않고 원본 비율·색상을 유지한다. 일반 화면 제목·뒤로가기 헤더에 로고를 일률적으로 추가하지 않는다. SVG 로고에는 캐릭터용 하늘색 배경을 강제하지 않는다. 파일이 없으면 임의 대체하지 않고 누락을 알린다. 제작·검증은 `docs/figma-contract.md`의 헤더 로고 절차를 따른다.
- 규칙 위반은 캔버스/원본 JSON을 수정해서 해결한다. snapshot, 검사 결과, 시각 관찰을 만들어서 통과시키지 않는다.
- 임의의 폴더 존재, 체크 표시, 이전 PASS만으로 완료 판단하지 않는다.
- Figma MCP 호출은 `figma-worker` 전담 자식 에이전트에 위임한다. 메인 세션에서 Figma 읽기·쓰기·검색·캡처·추출 도구를 직접 호출하지 않는다. 원본 응답과 이미지를 메인 컨텍스트에 전달하지 않는다.
- 메인은 요구사항·승인·작업 범위·로컬 게이트를 관리한다. 전담 에이전트에는 전체 대화 대신 작업 계약과 필요한 파일 경로만 전달한다. 같은 파일의 Figma 작업은 순차 실행한다. 상세 절차는 `docs/figma-delegation.md`를 따른다.

## 프로젝트 스킬

| 요청 | 진입점 |
| --- | --- |
| 디자인 시작/이어서 진행 | `$ui-flow` |
| 레퍼런스 수집 | `$collect-references` |
| 레퍼런스 분석 | `$analyze-references` |
| 화면·상태·흐름 정리 | `$build-structure` |
| 최소 토큰·대표 시안·방향 선택 | `$generate-rules` |
| 컴포넌트·전체 화면 생성 | `$create-figma` |
| 검증/결함 수정 | `$audit-design` |
| Figma MCP 전담 실행 | `$figma-worker` (메인이 작업 계약으로 위임) |

역할명은 기존 프로젝트의 단계 명칭과 맞춘다. Codex 런타임 규약 때문에 위치는
`.agents/skills/<name>/SKILL.md`를 유지한다. 자동 발견이 아직 안 되면
해당 파일을 직접 읽거나 이 프로젝트를 새 Codex 세션으로 연다.

## Figma·이미지 도구

`docs/tool-adapters.md`를 관련 작업에서 읽는다. 세션에 실제 제공된 도구/설치된 스킬을
발견해 사용하며 MCP 이름이나 skill:// 경로를 고정하지 않는다. Figma 호출 전 해당
플러그인의 필수 스킬을 로드한다. Figma 메타데이터와 스냅샷 계약은 `docs/figma-contract.md`.

## 구현 변경 검증

Node.js 18+ / 외부 npm 의존성 없음. 코드 변경 후 `npm test`, `npm run doctor`, `npm run status` 실행.
새 복제본은 초안과 빈 시안 목록으로 시작하므로 `npm run check`가 실패하는 것이 정상이다.
완료하지 않은 디자인 단계를 테스트 통과를 위해 PASS로 만들지 않는다.

---
> Source: [figmatutor-info/designflowharness-codex](https://github.com/figmatutor-info/designflowharness-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
