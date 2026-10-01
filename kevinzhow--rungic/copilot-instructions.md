## rungic

> 用户明确要求 SSH 自动开启。升级和排障时保留启用状态，不得根据旧文档的“默认关闭”擅自关闭 SSH 或增加关闭策略。`ssh.socket` 的开机启用与监听即是自动接入入口，`ssh.service` 可按连接触发启动。

# 用户要求与工程约定

## SSH 自动开启（用户于 2026-09-29 明确要求）

用户明确要求 SSH 自动开启。升级和排障时保留启用状态，不得根据旧文档的“默认关闭”擅自关闭 SSH 或增加关闭策略。`ssh.socket` 的开机启用与监听即是自动接入入口，`ssh.service` 可按连接触发启动。

## 项目镜像制作 Skill

用户于 2026-09-28 要求将三段式制作经验固化为项目通用 skill，2026-09-30 调整为“CI1 设备底座/GKI → CI2 独立 RungicOS 镜像 → CI3 单独安装/升级 Rungic”，不再以 Android 与 Rungic 整包刷入为默认目标。接入新机型/固件、构建和安装排障时，使用 `.agents/skills/rungic-three-stage-image/SKILL.md`（可显式调用 `$rungic-three-stage-image`）。已有兼容底座可复用；Rungic 更新不默认重刷 Android 或清数据。完整 Android 整包工具保留给明确指定的历史/恢复工作。新独立首装安装器尚待实现，不能把现有 APT 部署或旧 product 首启种子称为已完成的新入口。契约见 docs/75。按需读取 Skill 参考；设备差异进入 spec/适配器，产物与缓存放 `.work/`。

## 适配前先调研（用户于 2026-09-23 明确要求）

每一项 Android / Linux / Plasma 适配开始实现之前，先广泛调查上游、类似项目和同类设备的已有工作，寻找最适合本机的可复用方案。

- 查实际源码、近期版本、已知问题及其修复状态，不能仅凭 README 的功能声明或旧教程决定。
- 比较直接使用、少量修改和自行实现的成本，优先保留成熟组件。已有实现质量不好、架构不适合或维护成本更高时，可以重写必要部分。
- 结合本机 Android 16、ARM64、Alpine musl、LXC、原生 Wayland、Mesa/KGSL 和 SELinux 状态核对兼容性。
- 本地记录来源/版本、许可证信息、选用或放弃原因、剩余问题和实机验收办法。研究结果与已验证可用的功能要明确区分。
- 一次初步检索不代表该项已经完成选型；在实际适配前继续完成对应源码和接口核验。没有搜到适配方案，只能记为本轮未找到，不能宣称不存在。
- 这项要求是工作顺序与质量要求，不增加逐项请求用户确认的流程。

当前设备能力审计见 `docs/research/28-capability-audit.md`；首轮复用研究见 `docs/research/29-reuse-research.md`。

已部署能力与验收范围见 `docs/research/30-feature-adaptation.md`；桌面与 Android 后端的实际连接、启动/挂载、接口契约、研究方法及扩展入口见 `docs/research/31-backend-integration.md`。后续适配先对照当前架构，保留源码/补丁与实机证据，并同步更新这两篇的相应内容。

## 操作前先查项目知识库（用户于 2026-09-27 明确要求）

在安装、刷机、构建、部署或排障之前，先检索 `docs/`、相关工具的注释/测试和已有实机日志，读完与本次设备、固件、组件和操作路径直接相关的记录，再制定命令与回退步骤。不能只看计划文档；要核对历史的失败原因、修正版本、实际验收边界和当前源码实现，避免重复已知错误。旧记录适用于别的机型或版本时，只复用方法，重新核对本机身份、槽位、镜像哈希、接口和运行状态。

- G100 / `portov_cn` 镜像工作先查 `docs/75-image-build-separation.md`、`docs/77-g100-three-ci-assessment.md`、`docs/78-g100-firmware-inventory.md`；Magisk 与刷写另查 `docs/05-magisk-root.md`、`docs/11-stock-install.md`、`docs/12-offline-magisk.md`、`docs/13-offline-magisk-user-app.md`。其中 G100 S / `mumba_cn` 的镜像、哈希和刷机命令不能直接用于 G100。
- X70 Air Pro / `vantage_cn` / `W2WV36.55-75-15` 先读 `profiles/devices/motorola/vantage_cn/W2WV36.55-75-15-knowledge.md`，再按症状读 83/85 篇。同目录保存执行 spec、精确 fastboot adapter 和供提取器实际使用的 stock identity；知识笔记不改写已发行 spec 的哈希绑定。通用 ABI、构建隔离与清理经验已进入三段式 skill 的对应 references。
- 已知坑：Motorola bootloader 拒绝重新封装的 `super.img` 时，参照 11 篇核验 fastbootd 的分区刷写路径；`oem fb_mode_set` 后进入 fastbootd 前要清除标志。Magisk 仅修补 `init_boot` 后的首次运行可能提示修复环境，完整离线首启机制与“不能把 Magisk 作为系统应用”的教训见 12、13 篇。Magisk 31.0 的 SQL NULL 崩溃见 39 篇。
- 多个 ADB server 或多台手机同时在线时，先用 `adb devices -l`、端口和设备序列号核对连接归属；后续每条设备命令指定精确序列号。2026-09-27 曾同时运行 5037/5038，USB G100 被 5037 接管，5038 只显示 Wi-Fi G100 S，不能把单一端口未列出设备判定为手机启动失败。
- G100 首次刷入 Magisk 修补的 `init_boot` 后，管理器可能提示“修复运行环境”并重启；`magiskd` 已在运行不代表 Shell 已获授权。2026-09-27 实测需在 Magisk 的“超级用户”页启用 Shell，之后 `su -c id` 才得到 uid 0。核验时同时检查 Magisk 版本、普通应用身份与 SELinux，不把一次 `su` 拒绝误判为内核启动失败。
- G100 2026-09-28 清数据首启问题见 79 篇：原厂、简单 Magisk 和仅离线种子引导的 `init_boot` 曾在已有数据状态下启动；完整包清数据后进入 Recovery。将部署触发改为 Magisk `service.d` 的 v4 整包复刷后仍进入 Recovery，故不能把 `sys.boot_completed` 触发器认定为已证实根因或把此改动记为修复成功。G100 S 11 篇的简单 Magisk 方案做过清数据首启，13 篇的离线种子方案保留了其他用户数据，两者验收边界不同。后续应固定其余镜像和数据状态、逐项替换启动组件定位，避免同时改镜像与清数据后作因果判断。
- ADB 多命令 root 调试不要写成 `adb shell su -c '命令一; 命令二'`：本机 ADB 的 shell 转义可让只有第一条命令以 root 运行。改为 `printf '%s\n' '命令一' '命令二' | adb -P 5037 -s ZY32M9MRVP shell su -c sh`，逐条确认身份与输出。Magisk root 上下文的 `pm install`、`pm grant`、`appops set` 曾出现 Binder `Failed transaction (2147483646)`；需在 Android shell 上下文安装或改为镜像预装。手机 toybox `flock -n 9` 对继承 fd 报 `Bad file descriptor`，首启锁改用 Magisk BusyBox 的 `flock -n 文件 命令`。
- G100 rootfs 首装时 Android toybox `dd --help` 虽列出 `conv=sparse`，实际会报 `bad conv=sparse`；对 16 GiB 稀疏镜像使用已验证的 ARM64 稀疏写入器并核对整镜像 SHA，避免占满 `/data`。LXC 的早期初始化日志目录须在镜像内预建；toybox loop 的 autoclear 会在容器退出后留下失效的 dm 映射，重启前须由 `rootfs-image attach` 检查并重建映射。相关实机结果记录在 79 篇。
- 2026-09-27 G100 的整包试刷中，第一个原厂 `super.img_sparsechunk.0` 已写入，第二个分片的 fastboot USB 传输没有返回，主机复位后手机出现 USB `error -71` 且暂不能枚举；停止重试并先恢复设备连接。旧机型的 super 分片刷入经验不能当作本机已通过的路径。过程、后续恢复和验收边界见 79 篇。
- 新遇到的失败、修复和实机证据及时写入对应 `docs/`，并在下一次相关操作前重新查阅；研究结论、离线校验和实机验收必须分别标注。
- 本轮 G100 完整镜像的经验汇总见 `docs/80-g100-image-installation-retrospective.md`，逐次证据见 79 篇。`.5` 清数据刷入后用户已确认正常进入 Plasma；后续先复用安全阶段初始化、真实 loading 和账户准备门槛，不能将旧候选的失败或待验收状态当作最终状态，也不能把本机结果推广到其他机型。
- 将普通 APK 改为 product/app 预装时，须同时核验其原生库安装方式：ZIP 中压缩的 ARM64 JNI 库要放入对应应用的 `lib/arm64`，不能仅复制 APK。12 篇已有相关经验；79 篇的 G100 Rungic 因遗漏 `libc++_shared.so` 在启动时崩溃。`pm path` 和默认权限通过不足以验收应用，必须实际启动；用 `pm install -r` 临时修好也不能代替只读镜像预装验收。

## 优先修复共享系统能力，避免逐个应用重复适配

用户于2026-09-23明确要求：不要 case by case 地修复各个 App；尽量利用 Linux 与桌面系统已有的标准接口、服务及 Pipeline 扩展机制，在共享层解决问题，避免不同 App 反复遇到同类故障。

- 发现某个应用不能使用硬件或桌面功能时，先追踪它实际调用的接口与完整链路，区分共享后端缺失、标准接口未接入、能力协商/时序错误与应用自身缺陷。其他应用能用，不代表所有标准接口已经兼容；安装成功也不等于功能验收通过。
- 优先复用并完善系统机制，例如 libcamera Pipeline Handler、PipeWire、PulseAudio、GStreamer、Qt Multimedia、XDG Desktop Portal、Wayland 协议与桌面服务。适配应放在能让同类应用共同受益、职责正确的最低公共层；不要把硬件访问、权限处理、格式转换或时钟同步复制进多个应用。
- Android 摄像头、麦克风、编解码等能力继续共用已有后端。在 Linux 接口层补齐入口和能力协商，避免为每个 App 再造一条私有硬件通路。不能为了统一入口而破坏其他已工作的桌面或容器。
- 只有确认属于应用自身缺陷，或现有系统扩展机制不能合理解决时，才采用范围明确的应用补丁；记录证据、未采用共享层方案的原因、上游状态与维护/退出办法。功能设置和界面交互可以留在应用层，不能把它们与共享硬件适配混为一谈。
- 修改共享层后，除接口/协议级测试外，还应选取使用同一接口的多个独立应用交叉验收，并回归已有工作路径。媒体能力分别验证实际采集、编码文件、播放、时序、声音来源及停止后的资源释放；单个 App 出画面、生成文件或测试程序退出成功不足以代表整条能力可用。
- 本地文档记录“应用 → 标准接口/桌面服务 → 共享后端 → Android 硬件”的映射、已验证范围与缺口，后续适配先查此记录，避免重复研究和修复。

## 网络

用户于 2026-09-29 明确要求：每次涉及开发机、网络、构建或部署操作，先现场核验正在执行命令的机器身份、架构、网络路由与代理设置（例如 `hostnamectl` / `hostname`、`uname -m`、`ip route` 及对应系统代理查询）。远程命令在远端核验。不能根据旧对话、工作目录或文档把 K8、mibook 等名称当作当前开发机；下文机器条目只是历史记录或指定构建端点，不能替代本次检查。

用户于2026-09-27要求：开发环境默认使用所在宿主机的代理，任务先读取宿主机的系统代理再联网，不要假定直连。

- **Mac mini构建机**（`macmini.wire.net`，见docs/71）：用`scutil --proxy`读取macOS系统代理（当前为Surge，HTTP/HTTPS `127.0.0.1:6152`，SOCKS `6153`）。经ssh执行的命令和Docker容器都不会自动使用它：容器内以`host.docker.internal`代替本机地址，每条命令带上`http_proxy`/`https_proxy`，构建镜像时以`--build-arg`传入。`tools/build_on_device.py`的`MacMini`已按此实现，其他在Mac mini上的工具也要这样做。
- **手机**：下载走用户指定的`http://192.168.5.45:6152`（HTTP与HTTPS），优先于上级目录中的默认代理配置；容器内由`/etc/profile.d/proxy.sh`提供。
- **2026-09-28 的 K8 历史记录**：当时访问不到`192.168.5.45:6152`；用户要求测试不使用代理，SwiftWire 1080/8080 均超时，直连 AOSP 两个源码请求分别约 1.1/1.8 秒成功，当轮 X70 Air Pro 构建使用已授权直连。此记录不确定后续任务的执行主机与网络路径。
- **2026-09-29 的网络排障记录**：执行主机与地址的现场核验、G100 局域网 SSH 的 ARP 证据见 `docs/research/g100-ssh-connectivity-20260929.md`，不作为后续任务的固定开发机配置。
- 新增宿主机或工具时，先确认该机器的代理设置并写入本节。

## 独立 Plasma Mobile 环境的目标版本

用户于2026-09-23要求达到 **Plasma Mobile 6.5**，随后明确澄清 **可以采用更新稳定版**。当前要求是6.5或更新稳定版本，不再锁定6.5.x；此前6.3.6方案已撤回。

- 发行版按ARM64、glibc兼容、现成配套桌面依赖与维护成本选型，不因最初Debian13方案锁死底座。现已部署Ubuntu26.04LTS ARM64和官方Plasma Mobile6.6.5，定制KWin图形适配已显示桌面并通过GPU/触摸验收；不能把GPU单独探针通过当成桌面可用。选型见38篇，实施见40篇，Rime输入见41篇，运行补丁与验收见42篇。
- Plasma使用独立容器和Android入口。用户于2026-09-23后续明确停止维护Phosh，已移除本地Phosh专属实现；共享硬件能力保留在shared/。新桌面完成部署、硬件接入和实机验证前，不能以APK图标或rootfs引导成功代表可用。

## Magisk 31.0 数据库查询

2026-09-23 实际 `magisk --sqlite 'PRAGMA table_info(policies)'` 查询后 root 守护进程退出；源码与隔离复现定位到 SQL NULL 被直接转换为 `rust::Str`。见 `docs/39-magisk-daemon-crash.md`。

- 在此版本上，不向运行中的守护进程提交可能返回 SQL NULL 的查询，包括直接 `PRAGMA table_info(...)` 和未经检查的 `SELECT *`。
- 查询字段先查固定版本源码；确需数据库查询时选择明确非空列，或逐列使用 `COALESCE`，先在独立内存数据库核验结果。
- 不在实机复现该崩溃。不要将后续 `su` 的 SIGTRAP 当作 magiskd 最初退出的崩溃栈。

## 本地目录与同步边界

- 需要同步的源码、文档、基准数据和来源记录分别放在android/agent/desktop/system/shared/tools、docs、benchmarks、provenance等目录；当前只维护Plasma桌面。自有软件包定义在`packaging/`，发行清单在`release/`；`packages/`只管理上游配方与补丁。
- 下载、构建缓存、安装包、日志、截图、实机媒体、私钥与本机配置统一放在`.work/`，不得新增到源码目录。Python/Cargo开发可先`source tools/work-env.sh`；开发用Python依赖（PySide6、pytest）由`sh tools/dev-setup.sh`装入`.work/venv`，只用于原型和测试，进入生产的部分改用C++等重写（用户于2026-09-29明确）。
- 用户于2026-09-23明确要求同步开发用APK签名密钥：`signing/development/launcher-signing.p12`是上述规则的指定例外，随私有仓库跟踪，构建脚本默认使用它。此授权不包含其他密钥或`.work/`内容。
- 用户于2026-09-30明确同意：README 的产品展示图和动图放在`docs/images/readme/`，随仓库同步，是媒体只放`.work/`的第二个指定例外。只放经过挑选、缩小尺寸、确认不含隐私的成品；原始录像和截图仍留在`.work/readme/`。
- 目录说明见`docs/52-git-repository-scope.md`。不要恢复旧refs目录，也不要为了旧脚本重新引入Phosh；修复当前共享接口和路径。

## 上游源码与多机协作

- 所有修改过的上游组件按“固定上游＋补丁队列”维护（docs/71、docs/73）：只在`packages/<名称>/`（`recipe.json`固定来源与许可证，`debian/patches/rungic/`为DEP-3补丁），用`tools/pq.py prepare/export`修改补丁；共享和自有模块用配方的`overlay`放入源码树，不复制进补丁。不要恢复`vendor/`源码目录或`stage_vendor.py`。
- 2026-09-30已补齐原先直接维护的例外：Android原生宿主在`packages/android-host/`，Smithay/Winit分别在`packages/smithay/`、`packages/winit/`，Firefox移动配置在`packages/mobile-config-firefox/`。`native/plasma/`、`plasma/firefox-mobile/`和`vendor/`已删除。宿主自有模块在`android/host/`，`tools/prepare_android_host.py`在`.work/build/android-host/source/`组装三份固定源码；`android/build-native-core.sh`交叉编译Android库，Linux上游包继续用`tools/build_on_device.py`。`desktop/patches/qt-video-duration.patch`是未验收实验，未进入Qt补丁队列。
- 上游升级先核对新版本是否已包含我们的修复，已包含的删除；升级时更新配方版本、哈希和许可证记录。跨组件协议变更在同一组Git提交中同步；本机和K8用提交SHA协作，协作与核对结果见`docs/53-remote-system-development.md`。
- 本机生成的源码树、构建产物只能进入`.work`；审计仅按精确哈希豁免已核对的上游公开文件（`provenance/audit-exceptions.json`）。

## Rungic 首次进入与升级验收（2026-09-30 修订）

当前独立首装验证“兼容 Android 底座 → 全新 Rungic 安装 → 用户配置 → Plasma Mobile”，不要求清空 Android 数据；升级另外验证用户数据保留与失败恢复。2026-09-28 的“整包清数据刷入”要求仅适用于当时完整包或明确选择的旧流程，旧成功记录仍保留。普通重启、已有账户或临时补装不能替代相应首装验收；按本次用户指定范围测试，不把历史任务的摄像头排除扩大为永久限制。

- 首次安装必须显示 loading 和真实阶段；安装、权限、共享挂载与容器账户环境全部就绪后，才展示用户名密码表单。按就绪状态放行，不靠固定延时，也不把底层未就绪错误交给用户反复提交。
- APK 与安装器用带 release 的原子状态契约协作，根控制器独立核验完成标记；可复用的现有实现见 docs/79 和 docs/research/31。独立首装、升级、旧整包与受控 UI 验证分别记录，不能互相替代。

---
> Source: [kevinzhow/Rungic](https://github.com/kevinzhow/Rungic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
