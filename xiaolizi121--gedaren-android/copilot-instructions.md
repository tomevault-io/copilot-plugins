## gedaren-android

> 用 GitHub Actions 云端把 [割草达人-安卓版](../割草达人-安卓版/) 的竖屏触控版网页游戏打包成安卓 APK，用户本机零 Android 环境。

# 割草达人 GitHub Actions 云打包

## 项目目的
用 GitHub Actions 云端把 [割草达人-安卓版](../割草达人-安卓版/) 的竖屏触控版网页游戏打包成安卓 APK，用户本机零 Android 环境。

## 关键文件
- `www/`：游戏源码（index.html + mobile.css + mobile.js），源件由 `../割草达人-安卓版/` 复制
- `capacitor.config.json`：appId=`com.gedaren.daren`、appName=`割草达人`、webDir=`www`
- `android/`：Capacitor 7.6.8 生成的原生工程（Gradle 8.11.1 / compileSdk 35 / JDK 21）
- `android/app/src/main/AndroidManifest.xml`：已加 `android:screenOrientation="portrait"` 和 `VIBRATE` 权限
- `@capacitor/app`：监听 Android 返回键，返回键第一次暂停、第二次退出并保存
- `.github/workflows/build-apk.yml`：push main / 手动触发 → npm ci → cap sync → gradlew assembleDebug → 上传 artifact `gedaren-apk`
- `README.md`：游戏介绍（仓库首页展示）；`BUILD.md`：给最终用户的中文操作说明

## 关键事实（2026-08 确认）
- Capacitor 7 要求 JDK 21，CI 用 `actions/setup-java` temurin 21。
- 输出 APK：`android/app/build/outputs/apk/debug/app-debug.apk`，Debug 签名，可直接侧载安装。
- 竖屏锁定：官方做法是在 AndroidManifest `<activity>` 加 `android:screenOrientation`。
- 手机端触控：按住屏幕任意位置出现临时摇杆；不再有冲刺按钮/暂停按钮。
- 手机端视野：`MOBILE_VIEW_ZOOM = 0.88`，按一加 Ace 2 Pro 逻辑尺寸 412×915 优化。
- 进度存档：返回键暂停时和退出前写入 `gd4_save_v1`，菜单显示“继续上次进度”。
- 音效：Web Audio 合成音已增强（压缩器 + 噪声层 + 多层振荡器），手机版仍由首次点按解锁。
- 静音：左上角 🔊/🔇 写入 `gd4_muted`，暂停/切后台会挂起 AudioContext，恢复前台且未静音时才继续。

## 使用流程
用户建 GitHub 仓库 → 上传本目录全部文件（用 GitHub Desktop 最稳，避免漏掉 `.github`/隐藏文件）→ Actions 自动构建 → 下载 Artifact。

## 当前状态（2026-08-23 已跑通）
- GitHub 仓库：`xiaolizi121/gedaren-android`（public，默认分支 main）
- 本地已 `git init -b main`，已推送到 origin/main；本地提示需 `safe.directory` 指向本项目
- 首次云端构建成功：run `32619782941`，输出 `dist/app-debug.apk`（~4.1MB，debug 签名）
- 本机已安装 GitHub CLI（`C:\Program Files\GitHub CLI\gh.exe`），远程推送凭据通过 `-c credential.helper='!"C:/Program Files/GitHub CLI/gh.exe" auth git-credential'` + `GH_TOKEN` 临时令牌完成
- 注意：`dist/` 已加入 .gitignore，不要把 APK 提交进仓库

## 注意
- 本目录是独立工程，不属于父目录 Git 仓库。
- 改 `www/` 下的文件后，需要 `npx cap sync android`（本地）或重新 push（CI 自动 sync）。
- 图标暂用 Capacitor 默认，后续可用 `@capacitor/assets` 生成。
- GitHub 令牌是一次性临时使用，用完应由用户在 GitHub 后台删除；下次推送可用 `gh auth login` 正式登录。
- 工作流 `paths` 已限定为 www/android/配置类文件，README/BUILD 等文档变更不会重复触发构建。

---
> Source: [xiaolizi121/gedaren-android](https://github.com/xiaolizi121/gedaren-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
