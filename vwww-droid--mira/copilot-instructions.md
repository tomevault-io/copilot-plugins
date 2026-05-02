## mira

> 1. 真机构建不要用 `generic/platform=iOS` 兜底验证, 要用真实设备 `destination`.

# Mira repo notes

## iOS true device automation baseline

1. 真机构建不要用 `generic/platform=iOS` 兜底验证, 要用真实设备 `destination`.
2. 真机构建前要清掉宿主机库路径污染, 固定使用 `env -u LIBRARY_PATH -u SDKROOT xcodebuild ...`.
3. 当前已验证可成功的真机命令如下:

```bash
env -u LIBRARY_PATH -u SDKROOT \
xcodebuild \
  -project <path-to-mira-repo>/ios/Mira/Mira.xcodeproj \
  -scheme Mira \
  -configuration Debug \
  -sdk iphoneos \
  -destination 'id=<device-udid>' \
  -derivedDataPath <path-to-mira-repo>/build/ios-mira-device-native-relay-derived \
  -allowProvisioningUpdates \
  -allowProvisioningDeviceRegistration \
  ENABLE_DEBUG_DYLIB=NO \
  ENABLE_PREVIEWS=NO \
  build
```

4. `libbsm` `ldl` `lresolv` 这些链接项不要为了表面报错直接删掉, 它们很可能属于 iSH / Frida 兼容链路.
5. `idb` 已验证可以连上真机并完成安装启动, 推荐顺序是 `idb connect` -> `idb install --udid` -> `idb launch --udid`.
6. `idb launch` 传启动环境变量时, 直接使用 `IDB_` 前缀, 启动到 App 内后会去掉前缀.
7. 当前已验证可工作的自动启动示例:

```bash
idb connect <device-udid>
idb install --udid <device-udid> <path-to-mira-repo>/build/ios-mira-device-native-relay-derived/Build/Products/Debug-iphoneos/Mira.app
IDB_MIRA_RELAY_URL="http://<host-ip>:8765" \
IDB_MIRA_AUTO_CONNECT=1 \
idb launch --udid <device-udid> com.vwww.mira.ios
```

## iOS MCP and Frida baseline

1. iOS 的 MCP PTY 和 Frida 链路建立在 iSH 兼容层上, syscall 会经过翻译, 冷启动和脚本执行都比 Android 慢.
2. iOS 分析默认策略不是频繁开关 PTY, 而是优先保持一个长生命周期 session, 在同一个 session 中连续执行 `status -> list -> run -> rpc`.
3. 如果调用 Mira MCP 时没有显式传 `sessionId`, 应优先复用同一 `installId` 的活动 session, 不要每一步都 reopen.
4. iOS Frida Python 辅助逻辑不要依赖 `python3 - <<'EOF'` 这类 heredoc 注入方式, 更稳的是直接 `python3 -c` 并显式补齐 `frida-setup` 和 `PYTHONPATH`.
5. iOS Frida 等待逻辑不要依赖 `time.sleep`, 在 iSH 环境下它可能触发 `Errno 38`. 如果必须等待, 优先用 Frida 脚本即时 `send(...)`, 或由 server 内部使用更稳的轮询策略.
6. 当前已验证 iOS 可以稳定返回较大的枚举结果, 例如 `ObjC.enumerateLoadedClassesSync()` 级别的全量 class 枚举.
7. 当前已验证 iOS 可以稳定返回较大的 RPC 结果, 已测通约 256 KiB 级别字符串返回.
8. 如果 iOS 侧出现看似随机的掉线或崩溃, 先怀疑 session 生命周期和 reopen 频率, 再怀疑 Frida 脚本本身.

## Android automation baseline

1. Android 自动化入口固定使用 `<path-to-mira-repo>/mira-android`.
2. 默认链路是 build -> adb install -r -> am start.
3. Relay URL 通过 Activity extras 注入, 不走手工输入 UI.
4. 当前约定的 extras 是 `mira_relay_url` 和 `mira_auto_connect`.
5. 当前已验证可工作的自动化示例:

```bash
MIRA_ANDROID_RELAY_URL="http://<host-ip>:8765" \
<path-to-mira-repo>/mira-android
```

---
> Source: [vwww-droid/Mira](https://github.com/vwww-droid/Mira) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
