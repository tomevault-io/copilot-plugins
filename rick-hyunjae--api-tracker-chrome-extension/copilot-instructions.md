## api-tracker-chrome-extension

> 작업 전 반드시 `.claude/rules/karpathy-guideline.md`를 따른다

# CLAUDE.md

## Behavioral Guidelines

작업 전 반드시 `.claude/rules/karpathy-guideline.md`를 따른다

## Script Command

이 저장소는 **Vite + Vitest** 기반 **Chrome MV3 확장 프로그램**이다.

```bash
npm run dev            # Vite 개발 서버
npm run build          # tsc --noEmit && vite build → dist/ 산출
npm test               # Vitest watch
npm run test:run       # Vitest 1회 실행
npm run test:coverage  # 커버리지 포함 실행
npx vitest run src/path/to/file.test.ts   # 단일 파일
npx tsc --noEmit       # 타입체크 (build에 포함)
```

**확장 로드**: 빌드 후 Chrome `chrome://extensions` → 개발자 모드 → "압축해제된 확장 프로그램 로드"로 `dist/`를 지정한다.

## Architecture

MV3 확장. 세 실행 컨텍스트(서비스 워커 / 콘텐츠 스크립트 / UI)로 나뉜다.

```
src/
├─ background/   # 서비스 워커: 메시지 라우터(handleMessage), session-manager(appendCall), sender
├─ content/      # 콘텐츠 스크립트: injected-capture(fetch·XHR 패치), content-bridge, widget-host
├─ shared/       # types·messages·storage(chrome.storage.local)·domain-match 등 공유 모듈
├─ ui/
│  ├─ sidepanel/ # 사이드패널: Rail + List/Detail/Settings 뷰, view-utils, icons
│  ├─ widget/    # 페이지 위 플로팅 위젯 (shadow DOM, dock-position)
│  ├─ theme/     # 디자인 토큰(tokens.css)·폰트(fonts.css)·패널 스타일(components.css)
│  └─ consent/   # 동의 배너 (현재 dormant — MVP에서 게이트 제거됨)
└─ test-setup.ts # Vitest 전역 chrome API mock
```

루트에 `manifest.json`(MV3), `vite.config.ts`.

**캡처 파이프라인**: `content/injected-capture`(페이지 메인월드에서 fetch·XHR 패치, 캡처 시점 URL 절대화) → `content/content-bridge`(postMessage→runtime 전달) → `background/index`(handleMessage) → `background/session-manager`(appendCall, 캡처 필터) → `shared/storage`(chrome.storage.local) → 사이드패널·위젯이 `onStorageChanged`로 구독.

**전송 파이프라인(체리픽)**: 사이드패널 List(수집 탭)에서 체크박스로 전송 대상 선택 → 푸터의 전송 버튼이 곧바로 전송 → `SEND_CURRENT_SESSION { callIds }` (세션 이름은 UI가 싣지 않고, 백그라운드가 전송 시점에 `settings.sessionName`을 읽어 채운다) → `session-manager.splitAndArchive`(선택분만 `pending` 아카이브, 미선택분은 새 세션에 잔류) → `sender.sendSession`(페이로드에 `name` 포함) → 성공 시 `sent`+`mcpList` 병합 / 실패 시 `failed`. 기존 `SEND_SESSION`은 아카이브 세션 전용(향후 히스토리 뷰 재전송용).

## Key Info

- 산출물 `dist/`를 Chrome에 압축해제 로드 (개발 서버 상주 포트는 Vite 기본값)
- 타입체크 `npx tsc --noEmit`가 사실상의 게이트. **ESLint 설정 파일이 없으므로** lint는 tsc로 갈음한다(`.eslintrc` 없음)
- 설정은 `shared/storage`의 헬퍼로 직접 영속 → 서비스 워커가 캡처 시 읽어 필터 적용(별도 메시지 없음)
- **상태 변경 불변식**: `settings`를 제외한 모든 `currentSession`/`sessions` 변경(캡처·전송·개별/전체 삭제)은 반드시 백그라운드 메시지(write-lock 큐) 경유. 패널에서 `patchStorage`로 직접 쓰면 진행 중 전송의 늦은 쓰기와 lost-update 레이스가 생긴다 (`CLEAR_SESSION` 도입 배경)
- 사용자용 사용 가이드는 `docs/handoff/2026-07-28-extension-usage.md` (스크린샷 포함), 빌드·자동 테스트·E2E 검증 절차는 `docs/test/e2e_test.md`
- **UI에 없는 설정**: `blacklistedDomains`는 스키마와 `widget-host`에만 살아 있고 SettingsView에 노출되지 않는다. 화이트리스트는 캡처만 막고(위젯은 뜸), 블랙리스트는 위젯 마운트·스크립트 주입 자체를 막는다

## Superpowers

Superpowers 산출물 경로 규칙: `.claude/rules/superpowers.md`

## Code Graph Tools (Serena × code-review-graph)

코드 탐색·리뷰·리팩터링은 두 MCP 도구(**Serena** = LSP 실시간 / **code-review-graph** = 영속 그래프)를 라우팅 규칙에 따라 분업한다. Grep/Read보다 두 도구를 우선 사용한다.

**한 줄 원칙**: CRG로 시작해 Serena로 끝낸다. 같은 질문에 두 도구를 동시에 부르지 않는다.

상세 라우팅·워크플로·메모리 관리 규칙: `.claude/rules/code-graph.md`

## Workflow & Documents

작업은 다음 단계를 따라 진행되며, 각 단계의 산출물은 지정된 경로에 누적된다.

| 단계 | Skill / Agent | 산출물 경로 |
|---|---|---|
| 0. Compound Read | `compound-solutions` (read) | (조회만, 파일 생성 없음) |
| 1. Interview | `deep-interview` | `docs/interviews/YYYY-MM-DD-<topic>.md` |
| 2. Brainstorming | `superpowers:brainstorming` | `docs/specs/YYYY-MM-DD-<topic>-design.md` |
| 3. Writing Plans | `superpowers:writing-plans` | `docs/plans/YYYY-MM-DD-<feature>.md` |
| 4. Review | `plan-*` / `code-*` review | (파일 누적 없음 — 단발성 리뷰) |
| 5. Compound | `compound-solutions` (write) | `docs/solutions/<category>/<file>.md` |

**단계 0 필수**: Brainstorming 시작 전 `compound-solutions` read 모드로 `docs/solutions/`의 선행 지식을 확인한다. 문서는 카테고리 디렉터리로 나뉘고 각각 `module`·`tags`·`problem_type`·`symptoms` frontmatter를 가지므로, 그 필드를 grep하면 빠르게 좁힐 수 있다(인덱스는 `docs/solutions/README.md`). 관련 문서가 없으면 결과 없음을 확인하고 바로 진행한다.

**IMPORTANT**: 각 단계가 마무리되면 **반드시** 다음 단계 또는 적절한 후속 skill을 사용자에게 제안한다. 예: interview 종료 → brainstorming 제안, plan 작성 완료 → execute 제안, 작업 종료 → compound write 제안.

상세 운영 규칙:
- Compound 자동 트리거 표: `.claude/rules/compound-cluster.md`
- Superpowers 산출물 경로 정의: `.claude/rules/superpowers.md`

---
> Source: [Rick-HyunJae/api-tracker-chrome_extension](https://github.com/Rick-HyunJae/api-tracker-chrome_extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
