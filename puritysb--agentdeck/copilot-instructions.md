## agentdeck

> 이 프로젝트(AgentDeck)에서 작업할 때 모든 AI 에이전트(Antigravity 포함)는 다음 지침을 **반드시** 따르십시오.

# AGENTS.md

## 프로젝트 지침 (Antigravity & AI Agents)

이 프로젝트(AgentDeck)에서 작업할 때 모든 AI 에이전트(Antigravity 포함)는 다음 지침을 **반드시** 따르십시오.

### 1. 컨텍스트 및 아키텍처 파악 (필수)
- 모든 작업(특히 새로운 기능 추가, 구조 변경, 디버깅) 시작 전에 **반드시 `CLAUDE.md`와 `DEVELOPMENT_LOG.md`를 먼저 읽으십시오.**
- `CLAUDE.md`는 프로젝트 아키텍처, 브릿지-플러그인 통신 규약, 포트 설정, UI/UX 비전(특히 Android E-ink 최적화 규칙)을 담고 있는 **단일 진실 공급원(SSOT)**입니다.
- 정보를 여러 문서로 분산시키지 마십시오. 만약 프로젝트의 핵심 구조적 변경이 발생하면, 이 정보들을 `CLAUDE.md`나 `DEVELOPMENT_LOG.md`에 업데이트하여 최신 상태로 유지해야 합니다.

### 2. 워크플로우 자동화 활용
- 빌드, 환경 설정 등의 반복 작업은 직접 스크립트 명령어를 유추해서 실행하지 말고, `.agents/workflows/` 디렉토리에 정의된 워크플로우를 사용하십시오.
  - 예: 안드로이드 APK 빌드 (`build-android.md`), 터미널 환경 세션 시작 등

### 3. 주요 개발 원칙 요약
- **Monorepo**: 프로젝트는 `pnpm workspaces` 기반으로 구성되어 있습니다. 항상 적절한 패키지(`bridge`, `plugin`, `shared`, `android` 등) 디렉토리를 확인하고 작업하세요.
- **Android / E-ink UX**: 안드로이드 환경(Jetpack Compose) 수정 시 E-ink 디스플레이(Crema/Onyx) 특성을 매우 엄격하게 고려해야 합니다. (그레이스케일 디더링, 하드웨어 부분 새로고침 등 `CLAUDE.md`에 명시된 규칙 준수)
- **명령어 안전성**: 데몬 스크립트나 시스템 환경에 영향을 주는 코드를 테스트할 때는 신중히 접근하십시오.

### 4. App Store 심사 invariants (필수)
macOS 앱은 App Store 를 통해 배포되며 Apple Review Guidelines 2.5.2 (번들된 인터프리터 금지) / 4.2 (self-contained minimum functionality) / 4.2.3 (App Store 외부 설치/실행 유도 금지) 를 모든 변경에서 유지해야 합니다.

- **`AGENTDECK_APP_STORE` 컴파일 플래그 경계 존중**: `Process()`, `/bin/sh`, `osascript`, `.command` 스크립트 생성, 외부 CLI(`security`/`sqlite3`/`adb`/`openclaw`/`whisper-cli`) 호출은 전부 `#if !AGENTDECK_APP_STORE` 뒤에만 둔다. 새 서브프로세스 경로를 추가해야 한다면 이 조건부 뒤에 두고, `apple/scripts/verify-appstore-archive.sh` 의 금지 문자열 목록도 같이 업데이트한다.
- **App Store UI 문구는 companion executable 설치/기동을 유도하지 않는다**: "Install the AgentDeck CLI", "Run `agentdeck daemon install`", "Open Terminal and ..." 류 카피는 금지. Setup card, Settings, menubar 알림은 외부 도구 존재 여부와 무관하게 동일한 문구를 노출한다.
- **기능 분류 문서 선행**: 기능을 추가하거나 이동할 때 `docs/appstore-feature-matrix.md` 의 표에 먼저 행을 추가한 뒤 구현을 시작한다. 이 표는 App Store tier / terminal-only tier 분리를 지키는 계약이다.
- **심사 제출 전 검증**: `xcodebuild ... -configuration Release` 뒤에 `bash apple/scripts/verify-appstore-archive.sh <.app>` 를 로컬에서 돌려 통과시킨 뒤 커밋한다. Debug 빌드는 Swift debug dylib 때문에 verifier 가 실패하는 것이 정상이며, Release 통과가 기준.
- **자세한 규칙**은 `CLAUDE.md` 의 "App Store build invariants" 섹션과 `apple/APP_REVIEW_NOTES.md` 를 참조.

---
> Source: [puritysb/AgentDeck](https://github.com/puritysb/AgentDeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
