## esp32-arduino-builder

> 本仓库是 ESP32 Arduino 全芯片静态库自动编译平台，基于 PlatformIO espressif32 平台（Jason2866 架构）二次开发。

# AGENTS.md

本仓库是 ESP32 Arduino 全芯片静态库自动编译平台，基于 PlatformIO espressif32 平台（Jason2866 架构）二次开发。
通过 GitHub Actions 自动编译所有 ESP32 芯片（esp32 / s2 / s3 / c2 / c3 / c5 / c6 / c61 / h2 / p4）的 Arduino 静态库，并打包为完整的 Framework 包发布到 GitHub Release 供 PlatformIO 直接使用。

## 项目架构

- **平台层**：`platform.py`（`Espressif32Platform` 类）、`platform.json`（包/工具链定义）
- **构建脚本**：`builder/frameworks/arduino.py`（加载 framework 包内的 `tools/pioarduino-build.py`）
- **CI/CD**：`.github/workflows/build-libs.yml`（GitHub Actions，每日 02:00 轮询 + 手动触发）
- **板子定义**：`boards/*.json`（保留 Espressif、M5Stack、Seeed Studio 等主流板子定义）
- **编译产物**：打包为 `framework-arduinoespressif32.tar.xz` 与 `esp32-arduino-libs.zip` 发布到 **GitHub Release**（不提交进 git 仓库）
- **打包/校验工具**：`tools/` 下的 `package_framework.py`、`normalize_pio_specs.py`、`validate_pio_package.py` 等

详细架构说明见 [README.md](README.md)。

## 关键约定与机制

### Framework 整包分发与新鲜度检测

1. **整包架构**：CI 将 arduino-esp32 源码 + 补丁后的 `pioarduino-build.py` + 全芯片静态库（`tools/esp32-arduino-libs/<chip>/`）组装为单一 `framework-arduinoespressif32.tar.xz`，发布到 GitHub Release（固定 tag `framework-latest`）。
2. **固定 URL**：`platform.json` 中的 `framework-arduinoespressif32.version` 始终指向固定的 Release 下载链接，不再频繁产生 git 提交。
3. **强缓存绕过（指纹机制）**：PlatformIO 默认对相同 URL 的包只比对 URI 并跳过重新下载。为确保新构建能推送到用户端，CI 同时上传 `framework-fingerprint.json`。`platform.py` 的 `_ensure_framework_fresh()` 在每次构建前检查该指纹，发生变化时自动清除 PlatformIO 下载缓存并强制重新安装 framework 包。

> 注意：预编译库**不能**提交进 git 仓库——其中包含超长路径文件
> （如 Matter/connectedhomeip 头文件路径 >260 字符），会导致 Windows 上
> `git clone` 平台仓库时 checkout 失败（`Filename too long`）。

### 工具链选择

- Xtensa 芯片（esp32/s2/s3）→ `toolchain-xtensa-esp-elf` (15.2.0)
- RISC-V 芯片（c2/c3/c5/c6/c61/h2/p4）→ `toolchain-riscv32-esp` (15.2.0)

## CI/CD 注意事项

### GitHub Actions workflow

- 镜像：`espressif/esp32-arduino-lib-builder:release-v5.5`（Docker Hub 官方镜像）
- 仓库：`AnSiwei/esp32-arduino-builder`（GitHub）
- 密钥：`GITHUB_TOKEN`（GitHub 自动注入，workflow 声明 `permissions: contents: write`）

### 已知坑点与补丁清单

1. **GitHub Actions 注入的 GITHUB_TOKEN 只能访问当前仓库**，无法访问
   `espressif/arduino-esp32`。`config.sh` 里 `AR_REPO_URL` 的构造逻辑是：
   `GITHUB_TOKEN` 非空时拼成 `https://<token>@github.com/espressif/arduino-esp32.git`，
   空时回退到匿名 URL。若不处理，`install-arduino.sh` 的
   `git clone $AR_REPO_URL` 会因 token 无权限而 **403 Authentication failed**。
   **必须 `unset GITHUB_TOKEN`**（回退到匿名 URL，公开仓库可匿名 clone）。

2. **GitHub Actions 注入的 GITHUB_REPOSITORY_OWNER 是当前仓库 owner**（如 AnSiwei），
   导致 `config.sh` 里 `AR_USER=AnSiwei`、`AR_REPO=AnSiwei/arduino-esp32`，
   GitHub 上不存在该仓库，`git clone` 会 **404 Not Found**。
   **必须 `export GITHUB_REPOSITORY_OWNER=espressif`**。

> 两个坑点共同决定 `AR_REPO_URL` 的最终值，缺一不可：只有同时
> `unset GITHUB_TOKEN` + `export GITHUB_REPOSITORY_OWNER=espressif`，
> `AR_REPO_URL` 才会变成正确的匿名 URL
> `https://github.com/espressif/arduino-esp32.git`，从而成功 clone。

3. **`-A` 参数指定 tag 时 install-arduino.sh 会失败**：
   原始脚本用 `checkout -B "$AR_SOURCE_BRANCH" origin/"$AR_SOURCE_BRANCH"`，
   仅对分支有效。指定 tag（如 `3.3.11`）时 `origin/3.3.11` 不存在。
   **必须在构建前用 sed 补丁 install-arduino.sh**（fetch 加 `--tags`，直接 checkout 分支/tag）。

4. **TinyUSB master 会删除 vendor_host.c**：
   `update-components.sh` 从 master 拉取最新 TinyUSB，但上游可能删除文件
   （如 commit a57f857 删除了 `vendor_host.c`），而 lib-builder 的
   `arduino_tinyusb/CMakeLists.txt` 仍引用该文件，导致 CMake 报错。
   **必须在构建前从 CMakeLists.txt 中删掉对 vendor_host.c 的引用**。

5. **ESP-IDF v5.5 默认使用 newlib，导致 cross-endian 链接错误**：
   arduino-esp32 3.3.x 要求 picolibc multilib。若使用 newlib，GCC 会回退到通用大端库导致链接报错。
   **必须在 defconfig.common 追加 `CONFIG_LIBC_PICOLIBC=y` 与 `CONFIG_IDF_EXPERIMENTAL_FEATURES=y`**。

6. **cbor 组件在 picolibc 下缺少 open_memstream**：
   espressif/cbor 0.6.1~4 的 Linux open_memstream 依赖 glibc。
   **必须本地覆盖 cbor 并定义 `WITHOUT_OPEN_MEMSTREAM`**。

7. **编译耗时较长**（10 个芯片约 35~45 分钟），`timeout-minutes: 1440`。

### 验证组合

当前已验证的版本组合：
- arduino-esp32: `3.3.11`
- ESP-IDF: `v5.5.5`（分支 `release/v5.5`）
- lib-builder 镜像: `release-v5.5`
- 测试验证（2026-09-03）：esp32s3 / esp32c3 双环境在 Windows PlatformIO 6.1.19 编译通过

## git/上传中文编码问题（高频坑！）

- **症状**：GitHub release 的 name/body 中文字符全部变 `?`，即中文被损坏成问号。
- **根因**：Windows PowerShell 5.1 调用原生 exe（如 `curl.exe`）时，命令行参数按 ANSI(GBK) 编码传递，UTF-8 中文转 GBK 丢失变 `?`；GitHub 存的是损坏字符。
- **对策**：
  1. release 的 name/body 一律用**纯 ASCII**（如 `"ESP32 Arduino Framework (latest)"`），中文只放 YAML 注释/README。
  2. 必须用中文时：把 JSON 写成 UTF-8（无 BOM）临时文件，curl 用 `-d @file.json` 传文件内容，不经过命令行参数。
  3. git commit message 同理：Windows PowerShell 里 `git commit -m "中文"` 会乱码，建议 ASCII message 或 `-F 文件`。
- **注意**：CI 容器（Linux）内无此问题，只有 Windows 本地 PowerShell 操作 GitHub API 时中招。

## 常用操作与调试

```bash
# 手动触发构建（通过 GitHub API）
# POST /repos/AnSiwei/esp32-arduino-builder/actions/workflows/build-libs.yml/dispatches

# 下载并验证包脚本
python tools/validate_pio_package.py --libs-dir ... --framework-tar ...
```

---
> Source: [AnSiwei/esp32-arduino-builder](https://github.com/AnSiwei/esp32-arduino-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
