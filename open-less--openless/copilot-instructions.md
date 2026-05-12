## openless

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project

OpenLess is a menu-bar/tray voice-input layer. Hold or toggle a global hotkey, speak, and the dictated text is polished and inserted at the current cursor in any app. Product principles, state machine, and module list live in `docs/openless-development.md` and `docs/openless-overall-logic.md` — read those before changing product behavior.

The active codebase lives at `openless-all/app/` and is **Tauri 2 + Rust backend + React/TS frontend**, targeting macOS 12+ and Windows. The legacy Swift implementation (Sources/, Tests/, Package.swift, appcast.xml, Sparkle pipeline) was removed in commit `34d2823`; do not resurrect it.

UI must match `openless-all/design_handoff_openless/*.jsx` pixel-for-pixel; the JSX is reference-only, never imported.

## Build, Run, Test

### Tauri (current — start here)

```bash
cd "openless-all/app"
npm ci

# Dev: vite at :1420 + tauri shell
npm run tauri dev

# Build .app (+ DMG) — use this script, not `tauri build` directly,
# because it threads Apple signing env vars and validates Info.plist.
./scripts/build-mac.sh           # build, sign, install to /Applications, reset TCC
INSTALL=0 ./scripts/build-mac.sh # build only

# Frontend-only TS check
npm run build   # = tsc && vite build

# Rust type-check without full compile
cargo check --manifest-path src-tauri/Cargo.toml
```

### Windows (cross-check only — no macOS runner in CI)

```powershell
# Preflight: verify toolchain
.\scripts\windows-preflight.ps1

# Build (requires Windows host or cross-compile target)
.\scripts\windows-build-gnu.ps1
```

Generated artifacts:
- `openless-all/app/src-tauri/target/release/bundle/macos/OpenLess.app`
- `openless-all/app/src-tauri/target/release/bundle/dmg/OpenLess_<version>_aarch64.dmg`

Logs: `~/Library/Logs/OpenLess/openless.log` (macOS) / `%LOCALAPPDATA%\OpenLess\Logs\openless.log` (Windows).

There is no test runner wired in for the frontend. `src/lib/providerSetup.test.ts` is a hand-rolled assertion script — run with `npx tsx src/lib/providerSetup.test.ts` if you need it. Rust backend unit tests are run with `cargo test --manifest-path src-tauri/Cargo.toml --lib`; hardware / OS-integration behavior is still verified by running the app.

## Architecture

`coordinator::Coordinator` is the **single owner of session state**. Hotkey edges drive a small phase enum (`Idle → Starting → Listening → Processing`); recorder, ASR, polish, insertion, and history are wired here and nowhere else. Library/module code never calls across modules — they each depend only on shared types.

```
Rust (openless-all/app/src-tauri/src)        Purpose
──────────────────────────────────────        ────────────────────────────────
types.rs                                      Pure value types: DictationSession, PolishMode, HotkeyBinding, errors
hotkey.rs                                     Global hotkey monitor (modifier-key edges)
recorder.rs                                   Mic → 16 kHz mono Int16 PCM, RMS callback
asr/{mod,frame,volcengine,whisper}.rs         ASR providers: Volcengine streaming WebSocket + Whisper HTTP
polish.rs                                     OpenAI-compatible chat completions (Ark / DeepSeek / etc.)
insertion.rs                                  AX focused-element write → clipboard + Cmd+V → copy-only fallback
persistence.rs                                History/preferences/vocab JSON + platform credential vault
coordinator.rs + commands.rs + lib.rs         State machine, IPC surface, tray icon, window plumbing
permissions.rs                                TCC checks (Accessibility / Microphone)

Frontend (openless-all/app/src)
src/components/Capsule.tsx                    Capsule view + state enum
src/ (React)                                  Main window UI: Overview / History / Vocab / Style / Settings
src/i18n/                                     react-i18next init + zh-CN / en resources
src/pages/_atoms.tsx                          Recoil atoms — global frontend state
src/state/HotkeySettingsContext.tsx           HotkeySettings React context (capability + binding from backend)
```

### Dictation pipeline

```
hotkey edge (1st)  →  beginSession:  Recorder.start → ASR.openSession → BufferingAudioConsumer.attach
hotkey edge (2nd)  →  endSession:    Recorder.stop → ASR.sendLastFrame → awaitFinal → Polish → Insert → History.save
.cancelled         →  ASR.cancel, Recorder.stop, capsule .cancelled
```

Invariants:
- **Polish/ASR fallbacks are silent.** Missing Ark creds → insert raw transcript. Missing Volcengine creds → mock pipeline copies a placeholder. The contract is *"the user's words don't get lost"* — don't add hard errors here.
- **`BufferingAudioConsumer`** queues PCM until the WebSocket is ready, then drains. Recorder always pushes to it; ASR is attached after `openSession` resolves.
- **Hotkey is toggle-only**, not press-and-hold. The monitor yields one edge per modifier-key keydown; the coordinator interprets odd/even.

### Permissions, credentials, on-disk state

- **Bundle ID `com.openless.app`** is hard-coded in `openless-all/app/src-tauri/tauri.conf.json` and `CredentialsVault.serviceName`. Changing it breaks system credential vault lookups *and* every existing TCC grant.
- **TCC**: Microphone + Accessibility + AppleEvents. `NSMicrophoneUsageDescription` / `NSAccessibilityUsageDescription` / `NSAppleEventsUsageDescription` live in `openless-all/app/src-tauri/Info.plist`. After a fresh build that resets TCC, the app must be **fully quit and relaunched** after granting Accessibility before the global hotkey tap installs.
- **Credentials** live in the OS credential vault (macOS Keychain, Windows Credential Manager, Linux keyring) under service `com.openless.app`. The legacy plaintext JSON (`~/.openless/credentials.json` on macOS/Linux, `%APPDATA%\OpenLess\credentials.json` on Windows) is only a migration source and is removed after a successful vault write. Never hard-code keys or include legacy credential files in logs, exports, build artifacts, or bug reports.
- **Per-user data**:
  - macOS: `~/Library/Application Support/OpenLess/{history.json, preferences.json, dictionary.json}` — capped at 200 history entries. **Do not rename `dictionary.json` to `vocab.json`** (drops user data).
  - Windows: `%APPDATA%\OpenLess\`
  - Linux: `$XDG_DATA_HOME/OpenLess`

### Release pipeline

Push a `v*-tauri` tag → `.github/workflows/release-tauri.yml` builds macOS arm64 `.dmg` and Windows x64 `.msi`. macOS Developer ID signing + notarization runs only when `APPLE_CERTIFICATE` / `APPLE_CERTIFICATE_PASSWORD` / `APPLE_ID` / `APPLE_PASSWORD` / `APPLE_TEAM_ID` secrets are set; otherwise it falls back to ad-hoc signing with a CI warning.

When bumping versions, update **all** version fields: `openless-all/app/package.json`, `openless-all/app/package-lock.json`, `openless-all/app/src-tauri/tauri.conf.json`, `openless-all/app/src-tauri/Cargo.toml`, `openless-all/app/src-tauri/Cargo.lock`. 漏一个就会 mismatch。

#### Windows CI 红线（不要踩同一颗雷两次）

Windows release 链路修过四颗雷，每一颗的 fix 都是不可合并的——"顺手统一" 一次就回归一次。改 `.github/workflows/release-tauri.yml` 的 Windows 段或 `windows-package-msvc.ps1` 之前必读：

1. **手动 light.exe 调用必须带 `-sice:ICE80`**
   `wix/openless-ime.wxs` 把 x64 + x86 OpenLessIme.dll 都装进 `INSTALLDIR\windows-ime\`。32-bit Component 落 64-bit Directory 触发 ICE80 (LGHT0204)，但 DLL 是绝对路径、不依赖 SysWOW64 重定向，按 Microsoft 文档是合法用法。Tauri 2 没暴露 light 透传参数，所以 *它自己* 的 light 调用必失败；CI workflow 的 "Repair Windows MSI" 步骤和 `windows-package-msvc.ps1::Repair-TauriMsiBundle` 用 `-sice:ICE80` 重链兜底。
   - ✗ 不要去 "修" wxs 让 x86 落到 32-bit Directory（要么改 install 路径破坏 IME 注册，要么拆独立 32-bit MSI 是架构变更）。
   - ✗ 不要从 Repair 调用里拿掉 `-sice:ICE80`。

2. **Windows `tauri build` 必须拆两轮 invoke，NSIS 先 / MSI 后**
   ```bash
   tauri build --bundles nsis ...   # Pass 1: 必成功（updater 硬依赖）
   tauri build --bundles msi ...    # Pass 2: 允许失败由 Repair 兜底
   ```
   Tauri 2 的 updater 签名 (`.exe.sig`) 是 *post-bundle 钩子*——单次 `tauri build` 内任何 bundler 失败，**所有** bundle 的 signature 都跳过。MSI 必踩 ICE80（见 #1），所以单 pass 拿不到 NSIS 的 `.exe.sig`，`write-updater-manifest.mjs` 必报 "Missing updater signature"。
   - ✗ 不要合并回 `--bundles nsis,msi` 单 pass。
   - ✗ 不要移除 NSIS pass 的 `if [ "$nsis_exit" -ne 0 ]` fail-fast。
   - ✗ 不要省略 `--bundles` 走默认 `targets: "all"`——Tauri 字母序 msi→nsis，MSI 一挂 NSIS 永不跑。

3. **Windows tauri build step 的 shell 必须 `bash`，不是 `pwsh`**
   `pwsh` 调外部命令会吃掉 `'{"bundle":...}'` 的内部双引号，tauri 收到 `{bundle:...}` 当作无效 JSON 拒绝执行、连 candle 都不会跑。1.2.15 翻过一次。
   - ✗ 不要因 "Windows 默认 pwsh 更顺" 而改回去。

4. **Repair 假设 candle 已跑出 wixobj**
   Repair 步骤兜的是 *light 阶段* 失败。如果 Pass 2 在 candle 之前就挂（比如 JSON 引号问题、wxs 语法错），Repair 会以 "Required WiX object missing" 死掉——别去 "加强" Repair，先去修上游为什么 candle 没跑。

#### 修 Windows CI 之前的固定动作

不看历史日志就盲改 workflow 是这一段坑反复刷新的根因。每次 Windows job 失败按这个顺序：

1. `gh run view <id> --json jobs -q '.jobs[]|select(.name|contains("windows"))|.databaseId'` 拿 job id。
2. `gh api repos/appergb/openless/actions/jobs/<job-id>/logs > /tmp/win.log` 抓全日志。
3. `grep -n "ICE\|light\|makensis\|Bundling\|Running\|Tauri\|Error\|exit code" /tmp/win.log` 找事件序列。
4. `git show v1.2.13-tauri:.github/workflows/release-tauri.yml` 对比最后一个 known-good 版本——v1.2.13 是 IME wxs 加入前最后一次成功的 Windows release。
5. 实质 diff 锁定后再动 workflow / wxs / 脚本。

#### 发版流程（保持现状，不要改）

修 Windows CI 时按这个流程迭代：

1. 改 workflow / wxs / 脚本，提交到 main。
2. bump 五处版本号（见上）。
3. `git tag v<X.Y.Z>-tauri && git push origin v<X.Y.Z>-tauri` → CI 跑 → action-gh-release 自动发版。

`release-tauri.yml` 触发条件只有 `tags: [v*-tauri]` + `workflow_dispatch`。release publish 步骤 gated on tag，所以 dispatch run 跑了不发版。
- ✗ 不要把流程改成 "push main 自动跑 CI 验证再 tag"——已经讨论过否决了，现状的 bump+tag 流程是用户偏好。
- ✗ 不要 `--amend` 已 push 的 tag 或 force-push。失败的 tag 留着、bump 一个新版本号继续。

## Repo conventions

- **Comments, log messages, user-facing strings, and most docs are in Simplified Chinese.** UI strings additionally route through `react-i18next` (`src/i18n/{zh-CN,en}.ts`) so we ship English alongside; `zh-CN.ts` is source of truth.
- **macOS hotkey monitor must use native `CGEventTap`, never `rdev`.** `rdev` synchronously calls `TSMGetInputSourceProperty` from non-main threads, which macOS 14+ aborts via `dispatch_assert_queue_fail` → SIGTRAP. macOS uses CGEventTap; `rdev` is only used on Linux/Windows.
- **Don't `NSApp.activate` on the dictation path** — it steals focus and breaks insertion. Only call `set_activation_policy(Regular)` + `activateIgnoringOtherApps` from `show_main_window` / mic-permission prompts, never from `start_dictation`.
- Rust modules wrap shared mutable state with `Arc<Mutex<...>>` (parking_lot). Keep that locking discipline when adding fields.
- Rust modules depend only on `types.rs`. New cross-module wiring goes in `coordinator.rs`, not in the leaf modules.

### Adding a new module

1. Add a `<name>.rs` (or directory) under `openless-all/app/src-tauri/src/`, importing only from `types`.
2. Register it in `lib.rs` (`mod <name>;`).
3. Wire it into `coordinator.rs` and expose any frontend-callable surface via `commands.rs` + `invoke_handler!`.
4. Add the matching TS wrapper in `openless-all/app/src/lib/ipc.ts` (with a mock branch for browser dev).

### Third-party service integrations & library / platform API research

When implementing features that depend on **anything outside this repo** — external HTTP APIs (ASR providers, polish endpoints, GitHub API), unfamiliar crates / npm packages, platform APIs (Apple Security framework, Win32, CoreFoundation), or any SDK whose surface shifts faster than your training cut-off — do not write integration code from memory. API surfaces drift; model training data is stale by definition. The same workflow below applies whether you are calling an HTTP endpoint, learning a new Rust crate, or wiring a system framework — substitute "endpoint URL" / "function signature" / "feature flag" as appropriate.

Follow this research-first workflow:

1. **Analyze before coding.** Identify every external call this feature needs: endpoint URL, HTTP method, authentication mechanism, request body schema, expected response schema, and error codes.
2. **Delegate web search to a sub-agent.** Spawn a read-only sub-agent whose sole job is to search for official documentation. The sub-agent runs in parallel — you continue other work instead of blocking on sequential web pages.
3. **Filter sub-agent results.** When the sub-agent returns, extract only the information directly relevant to the current implementation. Discard marketing pages, unrelated API versions, or tangential tutorials.
4. **Cross-verify one key finding.** Before writing code, validate at least one structural claim (endpoint URL, required header, auth format) with a direct `web_search` or `fetch_url` call. Sub-agents can hallucinate.
5. **Implement from verified documentation.** Only write integration code after the above steps. Never guess.

**Sub-agent search brief:**
- Focus each sub-agent on a single external service or protocol — one service, one sub-agent.
- Prioritize official documentation domains (e.g., `docs.volcengine.com`, `platform.openai.com/docs`), falling back to the project's GitHub README.
- The sub-agent must return **structured** findings: endpoint URL, HTTP method, required headers, request body JSON Schema, response body JSON Schema, and error code meanings.
- If the documentation covers multiple API versions, the sub-agent must note which version was referenced.

**Anti-patterns (do not do these):**
- ✗ Writing API integration code from memory without a documentation search.
- ✗ Pasting entire web pages into the main agent context — the sub-agent does the filtering.
- ✗ Mixing field names or endpoint paths from different API versions.
- ✗ Skipping error handling — every external call must degrade gracefully when the service is unavailable.

---
> Source: [Open-Less/openless](https://github.com/Open-Less/openless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
