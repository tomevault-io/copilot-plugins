## claude-usage

> - 앱은 정상 실행됨: `pgrep`로 프로세스 살아있고, `lsappinfo`에서 `type="UIElement"` + 올바른 bundleID.

# CLAUDE.md — claude-usage (macOS 메뉴바 앱)

## ⚠️ 메뉴바 상태 아이템이 "안 보일 때" — 번들 id 스턱 함정 (이 저장소 최대 함정)

### 증상
- 앱은 정상 실행됨: `pgrep`로 프로세스 살아있고, `lsappinfo`에서 `type="UIElement"` + 올바른 bundleID.
- `NSStatusItem`도 정상 생성됨: `button` 존재, 이미지+타이틀 세팅됨, `window.level == 25`(statusBar), `window.isVisible == true`.
- 그런데도 **메뉴바 어디에도 아이콘이 안 그려진다.**
- 상태 아이템 창 프레임을 찍어보면 우측 클러스터가 아니라 화면 밖/빈 중앙(예: `x ≈ -(메인디스플레이폭)`)에 박혀 있음 — 시스템 메뉴바가 슬롯을 안 잡은 "유령" 상태.
- `NSStatusItem.isVisible`는 이때도 `true`라 코드로는 숨김 여부를 감지 못 함(거짓말함).

### 근본 원인
macOS(WindowServer/SkyLight)는 메뉴바 상태 아이템의 화면·위치를 **CFBundleIdentifier(번들 id) 단위로 영속 기억**한다. 멀티 디스플레이/디스플레이 재배치 환경에서 이 위치가 한 번 화면 밖으로 묶이면, **같은 번들 id의 모든 빌드가 계속 그 죽은 위치로 떨어져 영구히 안 보인다.**

### ✅ 해결책: 번들 id를 새 값으로 바꾼다
스턱 상태는 번들 id에 묶여 있으므로 `CFBundleIdentifier`만 신선한 값으로 바꾸면 슬롯이 새로 잡힌다.
- 이 저장소: `com.andy.claudeusage` → **`com.andy.menubar.claudeusage`** 로 변경(`build.sh`의 `BIN_ID`).
- 추가로 `statusItem.autosaveName`을 안정적인 문자열로 설정 → 위치가 앱 **자신의** defaults 도메인에 기록되어, 이후 다시 스턱되면 `defaults delete <bundleid>`로 초기화 가능.

### ❌ 효과 없는 시도 (전부 실험으로 검증함 — 다시 시도하지 말 것)
- **실행파일/타깃 이름 변경**(ClaudeUsage→ClaudeBar 등): 무관. 위치는 바이너리 이름이 아니라 번들 id로 기억된다. (과거 커밋의 ClaudeBar 리네임은 오진이었음.)
- `defaults delete com.apple.systemuiserver` / 앱 도메인 defaults 삭제
- `lsregister -u`/`-f` 재등록
- 같은 번들 id에서 `autosaveName`만 신선한 값으로 변경
- `killall SystemUIServer ControlCenter Dock`, 재부팅
- 시작 시 `NSApp.activate(ignoringOtherApps:)` 호출
- 부트스트랩 방식(`MainActor.assumeIsolated`), `LSUIElement` 유무, SwiftPM vs swiftc, 배포 타깃(.v13) — 전부 무관. 아이템은 항상 정상 생성됨(순수 배치 문제).

### 진단 방법
1. `statusItem.button?.window`의 `frame`/`screen`/`level`/`isVisible`를 **파일로** 덤프(`open`으로 띄우면 NSLog stderr가 분리되니 `FileHandle`로 /tmp에 기록). 프레임 x가 음수/클러스터 밖이거나 예상 밖 스크린이면 스턱.
2. 가장 확실한 가름: **신선한 번들 id의 최소 상태바 앱**(`NSStatusBar.system.statusItem` + `button?.title` 20줄)을 따로 띄운다. 그게 뜨면 코드가 아니라 번들 id 스턱이 원인.

## 멀티 디스플레이 메모
정상 상태 아이템은 일반 앱처럼 **활성 디스플레이의 메뉴바**를 따라 표시된다(특정 화면 고정 아님). 한 화면 밖 좌표에 묶여 안 보이는 건 위 "스턱"일 때뿐이다. 빌트인 노치/외장 모니터 자체는 원인이 아니었다.

## 빌드/실행 (개발 시 함정만)
설치·실행 기본 절차는 README.md "설치" 참고. 개발 시 주의점:
- `swift build --show-bin-path` **단독은 재컴파일을 보장하지 않음** — 항상 `swift build`를 먼저 실행.
- 베어 바이너리 직접 실행 금지 — 번들 Info.plist(LSUIElement) 미적용으로 동작이 달라진다. `open ClaudeUsage.app` 사용.

---
> Source: [oungo/claude-usage](https://github.com/oungo/claude-usage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
