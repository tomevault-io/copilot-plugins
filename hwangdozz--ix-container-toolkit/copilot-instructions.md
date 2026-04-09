## ix-container-toolkit

> **问题背景**：天数（Iluvatar）GPU 没有类似 NVIDIA Container Toolkit 的成熟容器支持。Kubernetes 中即使通过官方 Device Plugin 成功分配了 GPU 设备节点，普通镜像里仍缺少驱动库、工具链路径和动态链接器配置，导致 GPU 应用无法直接运行。

# ix-toolkit — 项目概述与开发记录

> 更新日期：2026-04-02

## 开发目的

**问题背景**：天数（Iluvatar）GPU 没有类似 NVIDIA Container Toolkit 的成熟容器支持。Kubernetes 中即使通过官方 Device Plugin 成功分配了 GPU 设备节点，普通镜像里仍缺少驱动库、工具链路径和动态链接器配置，导致 GPU 应用无法直接运行。

**目标**：为天数 GPU 实现一套对标 NVIDIA Container Toolkit 的工具链，把**宿主机驱动层能力**自动注入到容器中，让申请 GPU 的 Pod 尽量无需为了驱动适配而修改镜像。

## 当前结论

- 项目主线方案已经明确：`runtime shim + OCI prestart hook + installer DaemonSet`
- 真实节点验证已经完成，代码中的大部分默认路径假设成立
- 关键背景已更新：天数 Device Plugin 实际注入的环境变量是 `ILUVATAR_COREX_VISIBLE_DEVICES`，值是 GPU UUID，不是数字 index
- 当前实现已经补上 UUID 解析、`ixsmi` 调用、驱动路径解析、`ldconfig` 刷新、RuntimeClass 清单和节点自动打标等能力

## 项目信息

- **模块名**：`github.com/ix-toolkit/ix-toolkit`
- **语言**：Go 1.26.1+
- **目标平台**：Linux/amd64
- **主要组件**：
  - `ix-container-hook`：OCI prestart hook
  - `ix-container-runtime`：containerd runtime shim
  - `ix-installer`：节点安装器

## 已验证的真实节点环境

基于 `docs/project-status.md`、`docs/pod-analysis.md`、`docs/physical-node-validation.md`，当前已确认：

- **GPU 节点**：Iluvatar BI-V150 × 8
- **设备节点**：`/dev/iluvatar0` 到 `/dev/iluvatar7`
- **设备格式**：字符设备，major `508`，没有 `/dev/iluvatarctl`
- **驱动根目录**：`/usr/local/corex`
- **实际安装目录**：`/usr/local/corex-4.3.0`
- **符号链接关系**：`/usr/local/corex -> corex-4.3.0/`
- **驱动库路径**：`/usr/local/corex/lib64`、`/usr/local/corex/lib`
- **驱动工具路径**：`/usr/local/corex/bin`
- **OCI runtime**：`runc 1.3.3`
- **containerd 配置文件**：`/etc/containerd/config.toml`

结论：设备节点命名、驱动路径、containerd 配置路径这些基础假设都已经在物理机上确认过。

## Device Plugin 的实际行为

当前必须基于下面这个真实背景开发，而不是旧假设：

- Device Plugin 申请资源名时会给 Pod 注入 GPU 设备节点
- 注入的环境变量名是 `ILUVATAR_COREX_VISIBLE_DEVICES`
- 变量值是 UUID 列表，例如 `GPU-c22a...,GPU-3767...`
- 不是旧文档里写的 `ILUVATAR_VISIBLE_DEVICES=0,1`
- Device Plugin **不会** 自动挂载 `/usr/local/corex`
- Device Plugin **不会** 自动设置 `LD_LIBRARY_PATH`
- Device Plugin **不会** 自动写入 `ld.so.conf.d`
- Device Plugin **不会** 自动注入 OCI hook

这意味着 ix-toolkit 需要解决两件事：

1. 把 UUID 解析成实际的 `/dev/iluvatarN`
2. 把宿主机驱动库和工具挂进容器，并补齐动态链接器配置

## 当前架构

### 1. `ix-container-hook`

OCI prestart hook，由 `runc` 在容器启动前调用。

当前职责：

1. 从 stdin 读取 OCI state JSON，拿到 bundle
2. 读取 `bundle/config.json`
3. 查找 `ILUVATAR_COREX_VISIBLE_DEVICES`
4. 支持 `all`、`none`、数字 index、UUID 列表几种格式
5. 调用 `pkg/device` 枚举 `/dev/iluvatar*`
6. 用 `ixsmi --query-gpu=index,uuid --format=csv` 做 UUID 到 index 映射
7. 若 `ixsmi` 不可用，降级为位置映射并输出 Warn
8. 将设备节点 bind-mount 到容器 rootfs
9. 将驱动库和工具 bind-mount 到容器内的 `/usr/local/corex/...`
10. 写入 `/etc/ld.so.conf.d/ix-toolkit.conf`
11. 在容器 rootfs 内运行 `ldconfig` 刷新 `ld.so.cache`

### 2. `ix-container-runtime`

作为 containerd 使用的 runtime shim，透明包装底层 `runc`。

当前职责：

- 只在 `create` 路径介入
- 读取 `bundle/config.json`
- 检查容器 env 中是否包含 `ILUVATAR_COREX_VISIBLE_DEVICES=`
- 若命中，则把 `ix-container-hook` 注入 `hooks.prestart[0]`
- 其余命令原样转发给真实 `runc`

### 3. `ix-installer`

以 DaemonSet init container 方式运行，在 GPU 节点上完成一次性安装。

当前职责：

- 复制 `ix-container-runtime` 和 `ix-container-hook` 到宿主机
- 生成 `/etc/ix-toolkit/config.json`
- patch `/etc/containerd/config.toml`，注册 `ix` runtime
- 可选重启 `containerd`
- 使用 in-cluster API 给当前节点打 `iluvatar.ai/gpu=present` 标签

## 当前默认配置

以下内容表示 **installer 在当前节点上实际生成的常用配置**，与真实部署环境对齐：

```json
{
  "underlyingRuntime": "runc",
  "hookPath": "/usr/local/bin/ix-container-hook",
  "hook": {
    "driverLibraryPaths": ["/usr/local/corex/lib64", "/usr/local/corex/lib"],
    "driverBinaryPaths": ["/usr/local/corex/bin"],
    "containerDriverRoot": "/usr/local/corex",
    "deviceListEnvvar": "ILUVATAR_COREX_VISIBLE_DEVICES",
    "libraryFilterMode": "so-only",
    "libraryExcludeDirs": ["python3", "cmake", "clang"]
  },
  "logLevel": "info"
}
```

补充说明：

- `libraryFilterMode=so-only` 的目的，是避免把潜在的大体积 SDK 内容整体挂进容器
- 当前验证节点的 `lib64/` 里只有少量驱动库文件，但保留这个过滤策略更稳妥
- `resolveDriverPaths()` 会解析 `/usr/local/corex` 这类符号链接，兼容版本化安装目录

## 完整工作流程

```text
用户 Pod 申请 GPU
  resources.limits.iluvatar.ai/gpu: "1"
          ↓
天数 Device Plugin 分配 /dev/iluvatarN
并注入 ILUVATAR_COREX_VISIBLE_DEVICES=GPU-...
          ↓
containerd 调用 ix-container-runtime create --bundle <path>
          ↓
ix-container-runtime 检测到 GPU env，向 config.json 注入 prestart hook
          ↓
runc 执行 prestart hook
          ↓
ix-container-hook:
  1. 解析 UUID -> GPU index
  2. 找到对应 /dev/iluvatarN
  3. 挂载设备节点
  4. 挂载 /usr/local/corex/lib64 / lib / bin
  5. 写入 ld.so.conf.d
  6. 执行 ldconfig
          ↓
容器进程启动，容器可访问 GPU 设备与宿主机驱动
```

## 项目现状

截至 2026-04-02，文档中能确认的状态如下：

- 核心链路已完成
- 单元测试已补齐，覆盖 config / device / hook / runtime 核心逻辑
- `ILUVATAR_COREX_VISIBLE_DEVICES=none` 的错误行为已修复
- `ixsmi` 不在 PATH、且依赖 `LD_LIBRARY_PATH` 的现实问题已处理
- RuntimeClass 资源已补齐到 `deployments/runtimeclass/`
- DaemonSet 主容器已从错误的 distroless shell keep-alive 方案调整为 `pause` 方案
- 真实节点上已做过 hook 注入和 bind-mount 手动验证

## 项目边界

ix-toolkit 的职责是注入**驱动层**，不是把整个天数 SDK 都搬进容器。

适合由 ix-toolkit 注入的内容：

- `/dev/iluvatarN`
- `libcuda.so`、`libixthunk.so`、`libixml.so`
- CUDA 兼容运行时动态库
- `ixsmi` 等基础 GPU 管理工具
- `ld.so.conf.d` / `ld.so.cache` 所需配置

不适合由 ix-toolkit 负责的内容：

- PyTorch、vLLM、DeepSpeed 等 Python AI 框架
- 大型编译工具链、头文件、示例、文档
- 用户业务镜像里的应用层依赖

这与 NVIDIA Container Toolkit 的职责边界一致：宿主机负责驱动层，镜像负责应用层。

## 剩余关注点

虽然背景信息已经基本补齐，但还有几件事要持续注意：

- 需要继续确认不同版本天数驱动目录结构是否始终兼容当前默认路径
- UUID 到设备映射当前依赖 `ixsmi`，已提供降级路径，但仍应关注跨版本 CLI 稳定性
- installer 的容器内 patch containerd、重启 containerd、打节点标签逻辑，还需要更多集群侧集成验证
- 是否要进一步引入 CDI，仍然是后续可选演进方向，不是当前主链路 blocker

## 相关文档

- `docs/project-status.md`：当前功能完成度与项目状态
- `docs/pod-analysis.md`：真实 Pod 行为对比和 Device Plugin 分析
- `docs/physical-node-validation.md`：物理节点上的设备、驱动路径、`ixsmi` 验证

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HwangDozz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HwangDozz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
