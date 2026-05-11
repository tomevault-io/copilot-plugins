## joyoseedit

> enableSR = (fisrEnable == 2 || fisrEnable == 4)  = true

# CLAUDE.md

## 简介

一个 **KernelSU WebUI 模块**，编辑 V3 版 Joyose 云控数据库
（`/data/user/0/com.xiaomi.joyose/databases/SmartP.db` 与 `teg_config.db`）。
UI 跑在 KernelSU 的 WebView 里；模块通过 `bin/joyose-edit.sh` 作为唯一的
特权入口（子命令白名单 — 永远不把用户输入拼进 shell）。

**重点支持的路径是 MIFISR（骁龙 8 Elite 2 / 小米 17 系列）**。其他路径
（15 系列 AFME/FRC/FSR、红米独显 Novatek）**项目维护者没有实机**，代码基于
样本 DB 形状 + Joyose 反编译推断保留，**不保证实际跑得通**。

---

## Joyose 的 boot 决策树（`com.xiaomi.joyose.enhance.a.c()`）

实机选哪条增强上下文由 MIUI device_features + vendor 系统属性决定，按顺序：

```text
d() 先看 FeatureParser.getString("support_dual_dpu")
   == "NT"                   → NovaTekEnhanceContext    (红米独显)
   == "CUSTOMIZE"             → k.b (CustomizeEnhanceContext)   ← 17 系列走这条
   否则 d() 返回 false

e() 再看 FeatureParser.getString("game_enhance_feature_name")
   == "game_enhance_fisr"    + game_turbo_api_version >= 1 → q.i (MIFISREnhanceContext)
   == "game_enhance_frc_sr"   → com.xiaomi.joyose.enhance.b (combiner)
   == "MIFI_SR"              + ro.vendor.display.hyperos.miDualDPU_gamebox_version==3 → v.b
   否则 e() 返回 false

兜底：按顺序查 ro.vendor.display.iris_x7.support / ro.vendor.xiaomi.sr.support /
ro.vendor.gpp.frc.support，第一个为 true 的决定走 o.a / v.d / m.a。
```

**17 Pro Max 和 17 Ultra 都声明 `support_dual_dpu=CUSTOMIZE`**，所以**都走 `k.b`**
（不是我们早前猜的 `q.i`）。"q.i" 路径是给声明 `game_enhance_fisr` feature 的机型
准备的，实机没见过。

---

## MIFISR 路径详解（17 系列，已实机验证）

### 三层阻塞结构

游戏助手悬浮窗能否显示"插帧/超分"勾选框、改 DB 能否生效，取决于三层条件都满足
（星铁 VK 还有一条额外的 Layer 4 守卫，见下方单独小节）：

**Layer 1：MIUI securitycenter UI 层能力判定**

`com.miui.securitycenter` 的 `GameBoxVisionEnhanceUtils.needInitService()` 会查：

- `ro.vendor.gpp.frc.support`（决定 `isDeviceSupportFRC`）
- `ro.vendor.xiaomi.sr.support`（决定 `isSupportResolution`）
- 某个 DPU 检查（决定 `isSupportDualDPU`；17 / 15 系列硬件无 DPU，永远 false）

**任何一个 vendor 属性不为 `"true"`，游戏助手本地直接拒绝渲染画质增强面板，Joyose
和 DB 再怎么配都白费。** MifisrView 顶部 banner 读这两个属性做红/黄/绿诊断。

17 PM 原厂两个 vendor 属性都没设；17 Ultra 两个都是 true。对 17 PM 用户：模块**不会
自动改** vendor 属性（那是系统级写操作，超出本模块职责），UI 给出 `resetprop` 命令
让用户自己决定翻不翻。

**Layer 2：Joyose `customize_game_params.*` bean 存在性**

`k.b.getPictureEnhanceSupportType(pkg)` 的第一行：

```java
if (e.k(ctx).e(pkg) == null) return new int[2];   // [0,0] → UI 勾选框全空
```

`k.e.e(pkg)` 查 `k.e.f3250c` HashMap，这个 map 由 `k.e.w()` 解析
`customize_game_params` 下面的 **5 个并列数组**填充，任一数组里出现
`"<pkg>_xxx"` 条目即可让 `e(pkg)` 返回非 null：

- `dp_fi_config` —— DMI 策略的 srcFps→targetFps 映射
- `dp_sr_config` —— DPQ 策略的超分配置
- `game_mifisr_config` —— MIFISR 策略的合体配置
- `gfrc_config` —— GFRC（骁龙老 FRC 兼容）
- `mfrc_config` —— MFRC（同上）

每条字符串格式 `<pkg>_<载荷>`（parser 强制 `split("_")` 长度 = 2，所以包名里不能有 `_`）。

**Layer 3：`fisr_config` 的 `feature` 声明 + `support_game_mode` 位图**

`k.e.u(pkg, featureName, true)`（被 `k.b.getPictureEnhanceSupportType` 和
`k.e.q` 复用）：

```java
for (q.b bVar : this.f3249b.h2()) {              // 所有 fisr_config 组
    if (bVar.f().contains(pkg) || bVar.f().contains("OTHER")) {
        boolean has = bVar.j().contains(featureName);   // feature 列表含 "FI"/"SR"/"FISR"
        String mode = bVar.k(featureName);              // 对应 feature 的 support_game_mode
        if (mode != null) {
            String[] parts = mode.split("#");            // "1#1" → ["1", "1"]
            // MGAME(均衡) → 取 [0]，TGAME(性能) → 取 [1]
            int idx = "MGAME".equals(currentMode) ? 0 : 1;
            return has && parts[idx] == "1";
        }
        return has;
    }
}
```

**关键**：`support_game_mode` 是 **2 位 bitmap**（`[MGAME位, TGAME位]`，不是模式 ID 列举）。
**`1` = 该模式启用本 feature，`0` = 禁用**。实机对照 Ultra 原厂下发 `"0#1"`：均衡读
`parts[0]=0` → 禁用，性能读 `parts[1]=1` → 启用，正好对应 `k.e.u` 里
`parts[idx] == 1 ? true : false` 的代码逻辑（用户实测 2026-04-22 确认：`"0#1"`
均衡没选项 / 性能有选项）。本模块 `mifisrStandard()` 默认 `"1#1"`（两模式都开）。

**模式来源：`h0.p(ctx).q()` 读的是 `IFeedbackControl.isEnableOptimizeGame()`**
（bound 到 `com.miui.powerkeeper.FeedbackControlService`），true → `"TGAME"` 即
性能档，false → `"MGAME"` 即均衡档，默认 true。所以没绑上 FeedbackControl 时
默认按 TGAME 读 `parts[1]`。

### policy 字段完整清单（2026-04-23 反编译核对）

所有 `fisr_config.enhance_config[i].enhance_policy_config[j]` 的字段，Joyose 的
`q.g.b()` parser 处理了哪些、哪些被实际消费：

| 字段                       | Joyose 读取点               | 作用                               | 缺失行为                                                                                          |
| -------------------------- | --------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------- |
| `feature`                  | `k.e.u` / `q.d.c`           | "FI"/"SR"/"FISR"/... 关键判定      | 必须                                                                                              |
| `strategy`                 | `k.e.i` / `k.e.h`           | 绑 strategy 实例名                 | 必须                                                                                              |
| `support_max_refresh`      | `q.d.f` → `l.i.r`           | `X#Y` 两档 **Hz 整数上限**（见下） | **默认 60**（两档都 clamp）；官方下发普遍 `60#120`                                                |
| `support_game_mode`        | `k.e.u`（带 `z2=true`）     | 2-bit mode 启用 bitmap             | 非 `X#Y` 格式 silently 忽略 mode 位                                                               |
| `disable_scene_list`       | `k.b.r` → `l.i.n`           | 场景 ID 黑名单                     | 空 list = 从不禁用                                                                                |
| `support_resolution_leave` | `q.b.l` → `k.e.m` → `l.i.t` | 渲染分辨率 level 白名单            | 空字段 = 允许所有分辨率；**非空但不命中当前 level 会打 `invalid render resolution` 并静默拒激活** |
| `switch_default_status`    | `q.d.b` → `enhance.a.u`     | UI 开关初始位 bitmap `X#Y`         | 默认 0，影响首次显示                                                                              |

所有带 `X#Y` 格式的字段都复用 `q.d.c` 的 `split("#") + parseInt(parts[idx])`，
按当前模式取 `parts[!("MGAME".equals(strQ)) ? 1 : 0]`（MGAME→[0]，其他→[1]）。
**但取到的整数语义各自不同，代码复用 ≠ 语义一致**：

- `support_game_mode` 当 bool bitmap：`parts[idx] == 1` 判定是否启用 feature。合法值 `0#0` / `0#1` / `1#0` / `1#1`。
- `support_max_refresh` 当 **Hz 数值**：`l.i.r` 的 `Math.min(gameFps*2, parts[idx])` 决定 FI 输出上限。官方下发 `60#120`（均衡档 cap 60Hz → FI 相当于禁用；性能档 cap 120Hz → FI 满血）。15 系列 AFME 和 17 系列 MIFISR 同款下发；项目 `mifisrStandard()` 默认也是 `60#120`。
- `switch_default_status` 当开关初始位整数。

### group 级字段（`enhance_config[i]`）

| 字段                      | 读取点                                                                                          | 作用                                             |
| ------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `game_list`               | 遍历匹配 pkg                                                                                    | 必须                                             |
| `support_vk`              | `q.d.x`                                                                                         | 见 Layer 4                                       |
| `joint_action_cmd`        | —                                                                                               | 红米独显侧使用                                   |
| `sr_with_fi`              | **死字段**：`q.b.f3660g` 被 parse 但无外部 xref，Joyose 当前版本不读                            | 无需关心                                         |
| `special_ui_message_type` | `GPUTunerService.getSpecialUIMessageType` → `GameBoxVisionEnhanceUtils.G(Resources)` 选 UI 文案 | 只改"智能插帧"副标题文案，0/1/2 三档，不影响激活 |

### config 顶层字段（`fisr_config` 本身）

| 字段                                   | 读取点  | 作用               |
| -------------------------------------- | ------- | ------------------ |
| `enhance_config`                       | 全局    | groups 数组        |
| `composite_scene_force_disable`        | `b0.o5` | 多人同屏强制禁用   |
| `orientation_change_temp_shutdown_frc` | `b0.S5` | 方向切换临时关 FRC |

### Layer 4（仅星铁）：`support_vk` 守卫

`k.b.isSupportEnhance(pkg)` 里多一段**星铁专属**守卫，反编译原文：

```java
if (!"com.miHoYo.hkrpg".equals(str)
    || !com.xiaomi.joyose.smartop.gamebooster.scenerecognize.g.a(ctx).b(str)   // VulkanModeRecognizer
    || this.f3243m.x(str)) {                                                    // q.d.x = group.support_vk
    return e.k(ctx).s(str) ? 1 : 0;          // 正常路径
}
y0.d.d(f3229n, "hkrpg in vk mode but not support vk");
return 0;                                     // 勾选框消失
```

`q.d.x(pkg)` 的 xref 全局只有两处：`k.b.isSupportEnhance` 和 `q.i.isSupportEnhance`，
两者都做 **`pkg == "com.miHoYo.hkrpg"`** 的精确字符串匹配。其它游戏根本走不到这条分支 ——
也就是说 **`fisr_config.enhance_config[].support_vk` 只对星铁起作用**，对别的包打开/关闭
都等效无害。

字段来源：`q.g.b()` 解析 `fisr_config.enhance_config` 时 `bVar.o(jsonObj.getBoolean("support_vk"))`
（`q.b.n()` 是 getter，`q.b.o(bool)` 是 setter）。Ultra 云端原厂**不下发** `support_vk: true`，
所以开箱即用的体验就是"星铁在 GL 菜单能看到勾选框，真正登录切 VK 后消失"。用户想让勾选
框在战斗场景也保留，必须自己把星铁所在 group 的 `support_vk` 翻成 `true`。

`VulkanModeRecognizer` 通过 `Pair<pid, isVulkan>` 的进程级缓存判定"当前 pkg 是不是 VK 模式"，
在 pid 变化（重启游戏）时失效 —— 这解释了"关闭重启游戏、登录前勾选框还在，登录后消失"：
pid 新建时 Pair 还没写入，VulkanModeRecognizer 返回 false，Joyose 走正常路径；VK 引擎
真正启动后 Pair 被填充，守卫被激活。

[src/parsers/fisr-config.ts](src/parsers/fisr-config.ts) 已定义 `FisrGroup.support_vk` 类型
和 `setPkgSupportVk()` helper；[src/ui/MifisrView.vue](src/ui/MifisrView.vue) 在 fisr_config
路由表下方加了开关，对星铁会高亮"★ 星铁必开"。

### Feature ↔ Status ↔ Strategy 的硬编码映射

**`q.d.i(status)` 是 status → feature 名的字符串映射函数**（复用在 `q.d.f` 和 `k.e.u` 里）：
`1→"FI"`、`2→"SR"`、`4→"FISR"`、其他 → `"NA"`。这是 `l.i.n()` 通过 `q.d.f(pkg, status)`
**精确查自己那条 policy** 的 `support_max_refresh`（而不 fallback 到其他 feature 的 policy）
的根因 —— 没有 `q.d.i` 这层字符串映射，就没法从 status 整数精确定位到同一 group 内的
目标 policy。

`q.e.d(pkg, status)` 里 feature 字符串**写死**：

```java
if (status == 1 && listJ.contains("FI"))   return true;   // 仅插帧
if (status == 2 && listJ.contains("SR"))   return true;   // 仅超分
if (status == 4 && listJ.contains("FISR")) return true;   // 两个都开（合体）
```

**status 码在 `com.xiaomi.joyose.enhance.b.setEnhanceStatus(pkg, status)` 里分派**：

| status | 含义         | FI 子策略状态 | SR 子策略状态 |
| ------ | ------------ | ------------- | ------------- |
| 0      | 全关         | 0（关）       | 0（关）       |
| 1      | 仅 FI        | 1（开）       | 0（关）       |
| 2      | 仅 SR        | 0（关）       | 2（开）       |
| 4      | FI + SR 合体 | 1（开）       | 2（开）       |

数字 `4` 不是 `3` 是有意的错位编码（FI 内部 status ∈ {0,1}，SR 内部 status ∈ {0,2}，
两者同时开加起来是 3，外部 API 把 3 重映射到 4 作为独立码）。

**Ultra 实机只会触发 status=0 / 2**（"全关" 或 "仅 SR"）—— 原厂 fisr_config 里
**每款游戏只下发一条 `feature:"SR"`**，没下发 `"FI"` 也没下发 `"FISR"`。
`q.e.d` 分派里 status=1（仅 FI）和 status=4（合体）都要求 feature list 包含对应
字符串，否则进不来。这是**官方设计选择**，不是 Bug。项目 `mifisrStandard()` 返回三条
（FI/SR/FISR）比 Ultra 官方更激进——允许用户在白名单游戏上强开 FI 或合体模式，
但**实际效果方没验证过**，画面可能异常，属于专家尝试。

**`q.d` 另有两个并列策略注册表**（给 `fisr_config.strategy` 字段用的**标签字符串**映射，
不是策略**实现**注册）：

- `q.d.r()` 注册 17 / 15 系列高通通路的 6 个合法 key：`AFME` / `FRC` / `FSR` / `XAISR` /
  `XFI` / `FSR3`（后三个在 5 台样本 DB 里暂未出现，可能是未来新机型的占位）。
- `q.d.s()` 注册红米独显（Novatek）专属的 3 个 key：`NT#FI` / `NT#SR` / `NT#FISR`。

这两套**标签字符串表**和下面 `k.e.n()` 注册的**策略实现类**（DMI / DPQ / MISR / MIFISR /
GFRC / MFRC）是两层独立的映射：`q.d.r/s` 决定"fisr_config JSON 里 `strategy` 字段能填哪些
值"，`k.e.n()` 决定"每个值对应哪个 `l.*` 实现类"。两层由 `q.g.b()` JSON parser + `k.e.i(pkg,
status)` 查询串起来。遇到样本 DB 出现不在下方 6 个实现类表里的 strategy 值（比如 `XAISR`），
说明 parser 读到了 `q.d.r()` 注册的合法 key 但还没有实现类 ——此类目前无实测，保持解析透传。

**Strategy 实现类**（`k.e.n()` 注册的 6 个）：

| 名字        | 类          | `a()` 返回                                          | `e()` 查的数组         | 实际工作                |
| ----------- | ----------- | --------------------------------------------------- | ---------------------- | ----------------------- |
| **DMI**     | `l.b`       | 固定 `1`                                            | `"dp_fi_config"`       | 纯 FI；读 srcFps→target |
| **DPQ**     | `l.c`       | 固定 `2`                                            | `"dp_sr_config"`       | 纯 SR（读配置）         |
| **MISR**    | `l.j`       | 固定 `2`                                            | `""`（不读）           | 纯 SR（无配置）         |
| **MIFISR**  | `l.i`       | **动态**（默认 4，被 `k(pkg,status)` 校准成 1/2/4） | `"game_mifisr_config"` | 通用，可扮演 FI/SR/FISR |
| GFRC / MFRC | `l.g / l.h` | ?                                                   | `gfrc/mfrc_config`     | 骁龙老通路（未验证）    |

**关键不变式**（已反编译核对、纠正过之前的错误表述）：

`setEnhanceStatus(pkg, status)` 的流程：

1. `e.i(pkg, status)` 从 `fisr_config` 里按 `feature` + `strategy` 名字查 strategy 实例
2. 查到后**立刻**调 `strategy.k(pkg, status)` 把实例绑定到 `(pkg, status)`
3. 激活策略 → 最终 `s.y(ctx).J(pkg, status)` 写 `"11 <pkg> <status>"` 到内核
   `/data/system/mcd/gameInfo`（eventId 11 是 MIFISR 专用通道）

DMI / MISR / DPQ 的 `k()` 是 `l.a` 接口的空默认实现；`a()` 固定常量。
MIFISR 的 `k()` 把 `f3316e[pkg] = status` 存进 map，`a()` 按 pkg 从 map 取值
（默认 4）—— 所以 MIFISR **同一个实例**能在不同 pkg 上扮演 status 1/2/4 任一种。

由此得出的合法绑定：

- `feature:"FI"` → 可绑 `DMI`（`a()=1`）或 `MIFISR`（`k(pkg,1)` 校准）
- `feature:"SR"` → 可绑 `MISR` / `DPQ`（`a()=2`）或 `MIFISR`（`k(pkg,2)` 校准）
- `feature:"FISR"` → 实际上只见过 `MIFISR`（默认 `a()=4`）

**17 Ultra 云控原厂只下发一条 `feature:SR + strategy:MIFISR`**（每款游戏一条）。
项目 `mifisrStandard()` 预设（[src/parsers/fisr-config.ts](src/parsers/fisr-config.ts)）
返回 FI / SR / FISR 三条 policy**全部绑 MIFISR**，对齐 Ultra 实际下发；
之前 `SR→MISR` / `FI→DMI` 那种历史说法是基于"`a()` 必须等于 status 才能分派"的
误读，实际 Joyose 不按这个规则走。

### FISR 激活完整路径（已反编译逐层核对，2026-04-23）

FISR（status=4）的 securitycenter UI → Joyose 内存 → 底层 hook 全链路：

```text
[securitycenter 端]
GameBoxVisionEnhanceUtils.u0(FRAME_INSERT|RESOLUTION)
  ├─ zB = 当前 FI 开?  zC = 当前 SR 开?
  ├─ Y() = f25816s = isSupportSuperResolutionWithFrameInsert
  │     - Y=true  → 允许同时开 → t0(true, true, 4) → o0(4)
  │     - Y=false → 强行互斥：勾 FI 则 zC=false；勾 SR 则 zB=false  ★ UI 闸门 1
  └─ IGPUTunerInterface.setFrameInsertingOrSuperResolution(pkg, 4)
      ↓
[Joyose binder 入口]
GPUTunerService 打 log "set function: 4"
  → enhance.a.T(pkg, 4) → this.f1013b.setEnhanceStatus(pkg, 4)
      ↓
[k.b (17 系列 CustomizeEnhanceContext)]
k.b.setEnhanceStatus(pkg, 4):
  1. f0.o(ctx, "customize_enhance_status_<pkg>", 4)     // 写 SP
  2. aVarI = k.e.i(pkg, 4)                              // 按 feature="FISR" 查 strategy
     ├─ 遍历 fisr_config groups 找 pkg 对应 policy（feature=FISR）
     ├─ 读 policy.strategy → 从 k.e.f3251d 注册表返 l.a 实例
     └─ 若无 FISR policy 或 strategy 名非法 → 返回 null
  3. 若 aVarI == null → "call error status: 4 , now is X" warn log，**不激活**  ★ UI 闸门 2
  4. aVarI.k(pkg, 4) → l.i.k: f3316e[pkg]=4; f3315d=pkg
  5. b.n(aVarE, aVarI, str) → b.o(aVar, str) → aVar.d().f(str)
      ↓
[l.i.f(pkg) → l.i.n(pkg, 4)]   FISR 走和 FI 同分支（i2 != 2）
  ├─ t(pkg, 4) 校验 support_resolution_leave 白名单    ← 不命中 "invalid render resolution"
  ├─ s(pkg, targetFps) 校验 dp_fi_config 帧率映射      ← 不命中 "invaild targetFps"（注意是原文 typo）
  ├─ iF = q.d.f(pkg, 4)：按 feature="FISR" 读那条 policy 的 support_max_refresh
  │       （不 fallback 到 FI policy；FI / FISR 各读各的；SR 完全不读）
  ├─ 若 iF > user_refresh_rate → "userRefreshRate is too low"，不激活
  ├─ this.f3314c = 4
  ├─ u(str, iF)：打 "refreshRate update, ..., bestRefreshRate: N"
  ├─ t.k(ctx).u(str); o.i(ctx).m(str)：停旧温控 / PID 监测
  └─ p(aVarE, str, 4)：
      - 打 "Current game: ..., running strategy is 4"（y0.d.a 级，需开 smartop.debug）
      - s.y(ctx).J(str, 4) → eventId 11 推送 "11 <pkg> 4" 到 /data/system/mcd/gameInfo
      ↓
[游戏进程内 libmivk / libmigl]
parseFISREnable → fisrEnable = 4
  每帧 vkQueuePresentKHR (VK) 或 PostFrameEnd (GL) hook:
     enableFI = (fisrEnable == 1 || fisrEnable == 4)  = true
     enableSR = (fisrEnable == 2 || fisrEnable == 4)  = true
  → FI 和 SR 两条 pipeline 同时激活（libmivk 星铁侧是 MiVkStarRailMIFIModule）
```

### FI/SR/FISR policy 组合对 UI 行为的完整映射

`k.b.isSupportEnhance` 只看 FI 或 SR feature 是否在 policy 列表里
（`k.e.s(pkg) = o(pkg,false) || p(pkg,false)`，**完全不查 FISR**）。
`isSupportSuperResolutionWithFrameInsert = !k.e.q(pkg) = !(FI && SR && !FISR)`。
组合效果：

| FI  | SR  | FISR | `isSupportEnhance` | `getPictureEnhanceSupportType` | UI 可合体     | UI 表现               |
| --- | --- | ---- | ------------------ | ------------------------------ | ------------- | --------------------- |
| ✅  | ✅  | ✅   | 1                  | `[1,2]`                        | 是            | 两勾选框 + 可升级合体 |
| ✅  | ✅  | ❌   | 1                  | `[1,2]`                        | 否（UI 互斥） | 两勾选框但合体不可达  |
| ✅  | ❌  | ✅   | 1                  | `[1,0]`                        | 是但无意义    | 只显示 FI             |
| ❌  | ✅  | ✅   | 1                  | `[0,2]`                        | 是但无意义    | 只显示 SR             |
| ❌  | ❌  | ✅   | **0**              | `[0,0]`                        | —             | **面板整个不显示**    |

**securitycenter UI 没有独立"FISR"按钮**
（反编译 `GameBoxVisionEnhanceUtils.s()` 固定返回两项：智能插帧 / 超级分辨率），
FISR 的唯一进入方式是"同时勾 FI + SR" + `isSupportSRWithFI=true`。
所以 FISR policy 必须和 FI + SR policy 同时存在才有意义。

### MifisrView UI 对这些闸门的防呆

[src/ui/MifisrView.vue](src/ui/MifisrView.vue) 的 fisr_config 路由面板下方有一组
banner，触发条件和 Joyose 侧闸门一一对应：

| banner                    | 级别      | 触发条件                                            | 一键修复                              |
| ------------------------- | --------- | --------------------------------------------------- | ------------------------------------- |
| `hasMissingMaxRefresh`    | warn      | 任一 FI / FISR policy 缺 `support_max_refresh`      | 一键填 `60#120`（144Hz 屏手动改 144） |
| `hasNoFiSr`               | **error** | policy 列表不空但既无 FI 也无 SR（典型：只留 FISR） | 一键加 SR policy                      |
| `hasUselessFisr`          | warn      | FISR 存在但 FI 或 SR 缺其一                         | 一键补缺的那条                        |
| `resolutionLeavePolicies` | info      | 任一 policy 带 `support_resolution_leave`           | 无自动修复（引导去 JsonEditorView）   |

这些 banner 的逻辑都在 MifisrView script 端作为 `computed` 实现，
banner 优先级用 `v-if / v-else-if` 控制：`hasNoFiSr` > `hasUselessFisr`。

---

## 诊断工具箱

### 开启 Joyose debug 日志

Joyose 的 `y0.d.a(tag, msg)` 是 Debug 级封装：

```java
static {
    // Build.TYPE == "user"（零售 ROM）且 persist.sys.smartop.debug != true → f4592a=false
    f4592a = !"user".equalsIgnoreCase(Build.TYPE)
             || b1.f.c("persist.sys.smartop.debug", false).booleanValue();
}

public static int a(String tag, String msg) {
    if (f4592a) return Log.d(tag, msg);
    return 0;      // 零售机默认不输出
}
```

`y0.d` 方法到 logcat 级别的映射：

| 方法               | 实现                     | logcat 级 | 默认是否输出      |
| ------------------ | ------------------------ | --------- | ----------------- |
| `y0.d.a(tag, msg)` | `Log.d(...)` if `f4592a` | D         | **零售机默认关**  |
| `y0.d.c(tag, msg)` | `Log.e(...)`             | E         | 总是              |
| `y0.d.d(tag, msg)` | `Log.i(...)`             | I         | 总是              |
| `y0.d.j(tag, msg)` | `Log.v(...)`             | V         | 总是              |
| `y0.d.k(tag, msg)` | `Log.w(...)`             | W         | 总是              |
| `y0.d.f(tag, msg)` | 写文件（不走 logcat）    | —         | Android 8+ 写磁盘 |
| `y0.d.g/h/i/...`   | 组合（含 `y0.d.f`）      | 视子方法  | —                 |

**开 debug log**：

```bash
adb shell setprop persist.sys.smartop.debug true
adb shell am force-stop com.xiaomi.joyose
```

之后 `Current game: ..., running strategy is N` / `getBastRefreshRate, ...` /
`doEnhance <pkg>` / `composite scene, ignore` 这些 `y0.d.a` 级的关键日志才可见。

### MIFISR 激活核心日志（过滤器）

```bash
adb logcat -v time -s SmartPhoneTag_i:V SmartPhoneTag_b:V GPUTunerService:V
```

关键行对应表：

| logcat 行                                                                                            | 含义                                     | 来自                                 |
| ---------------------------------------------------------------------------------------------------- | ---------------------------------------- | ------------------------------------ |
| `GPUTunerService: set function: N`                                                                   | UI 发送 status=N（1/2/4）                | `setFrameInsertingOrSuperResolution` |
| `GPUTunerService: set PictureEnhance: true/false`                                                    | UI 总开关                                | `setPictureEnhancement`              |
| `GPUTunerService: isSupportGameEnhancePkg: N`                                                        | 支持判定结果                             | `isSupportGameEnhancePkg`            |
| `GPUTunerService: getPictureEnhanceSupportType: [X, Y]`                                              | FI/SR 各自支持位                         | 同名方法                             |
| `GPUTunerService: isSupportSuperResolutionWithFrameInsert: bool`                                     | 合体允许与否（UI 闸门 1）                | 同名方法                             |
| `SmartPhoneTag_b: doEnhance <pkg>`                                                                   | k.b 激活流程开始                         | `k.b.o`                              |
| `SmartPhoneTag_b: stopEnhance <pkg>`                                                                 | k.b 停止流程                             | `k.b.v`                              |
| `SmartPhoneTag_b: call error status: N , now is M`                                                   | **k.e.i 查 strategy 失败（UI 闸门 2）**  | `k.b.setEnhanceStatus`               |
| `SmartPhoneTag_i: getBastRefreshRate, gameMode: ..., fiFps: X, supportMaxFps: Y, bestRefreshRate: Z` | FI 倍帧计算明细                          | `l.i.r`                              |
| `SmartPhoneTag_i: refreshRate update, ..., bestRefreshRate: N`                                       | 最终刷新率                               | `l.i.u`                              |
| `SmartPhoneTag_i: Current game: ..., running strategy is N`                                          | **激活成功关键标志**                     | `l.i.p`                              |
| `SmartPhoneTag_i: Current game: ..., stopping strategy`                                              | 停止成功标志                             | `l.i.d`                              |
| `SmartPhoneTag_i: invalid render resolution, ..., ignore`                                            | **support_resolution_leave 白名单 miss** | `l.i.n`                              |
| `SmartPhoneTag_i: invaild targetFps N`                                                               | dp_fi_config 帧率 miss（原文 typo）      | `l.i.s`                              |
| `SmartPhoneTag_i: composite scene, ..., ignore`                                                      | 多人同屏强制禁                           | `l.i.n`                              |
| `SmartPhoneTag_i: inDisableScene, ..., ignore`                                                       | 命中 disable_scene_list                  | `l.i.n`                              |
| `SmartPhoneTag_i: userRefreshRate is too low: X, target refresh rate is: Y`                          | 屏幕刷新率设置低于需求                   | `l.i.n`                              |
| `SmartPhoneTag_i: hkrpg in vk mode but not support vk`                                               | **星铁 Layer 4 守卫生效**                | `k.b.isSupportEnhance`               |
| `VulkanModeRecognizer: isVulkanMode, <pkg>-bool`                                                     | VK 场景识别器                            | `scenerecognize.g.b`                 |
| `SmartPhoneTag_b: fstbRule is null`                                                                  | FSTB（骁龙老通路）规则缺                 | 走 MIFISR 正常，**非错误**           |

### `/data/system/mcd/gameInfo` eventId 对照

`com.xiaomi.joyose.smartop.gamebooster.control.j.y(eventId, pkg)` 会把
`<eventId> <pkg> <args>` 写入这个文件供 libmigl / libmivk 备份读取。已知映射：

| eventId | 含义                                 | 来自                                          |
| ------- | ------------------------------------ | --------------------------------------------- |
| 1       | `parseGameSceneId`（场景 ID）        | `l.i.o` 间接                                  |
| 2       | `parseGamePictureInfo`               | —                                             |
| 3       | `parseGameVrsAptLevel`（6 个 level） | —                                             |
| 4       | `parseCommonVrs`                     | —                                             |
| 5       | `parseFilterConfig`                  | —                                             |
| 6       | game picture quality                 | 经常看到 `6 <pkg> 0 0 0`，跟画质档位相关      |
| 7       | `parseMiFiEnable`                    | —                                             |
| 10      | `parseALRLevel`                      | —                                             |
| **11**  | **`parseFISREnable`（MIFISR 通道）** | `s.y(ctx).J(pkg, status)`（status ∈ 0/1/2/4） |
| 12      | `parseCompositeSceneType`            | —                                             |

**FISR 激活与否最直接的物化证据是 `/data/system/mcd/gameInfo` 里的
`11 <pkg> <status>` 行**。没这行 = `l.i.p` 没被调 = 激活失败。

**注意**：Joyose 进游戏后 binder 主动推送给 libmivk/libmigl，文件只作为备份，
有时会被刷成 `<status> 0`（stopEnhance 时）或整个消失（进程清理）。
所以 `cat gameInfo` 应在游戏前台激活后立即执行。

### SharedPreferences 观察点

| 路径                                                                            | 关键 key                                                  | 含义                                                                        |
| ------------------------------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------- |
| `/data/user/0/com.xiaomi.joyose/shared_prefs/com.xiaomi.joyose_preferences.xml` | `customize_switch_<pkg>`                                  | Joyose 侧 FI/SR 总开关（`k.b.isEnhanceOn`）                                 |
| 同上                                                                            | `customize_enhance_status_<pkg>`                          | 当前类型 1/2/4（`k.b.getEnhanceStatus`）                                    |
| 同上                                                                            | `G_RENDER_RESOLUTION_<pkg>` / `RAW_PICTURE_QUALITY_<pkg>` | 当前游戏渲染分辨率 level（被 `l.i.t` 用于 `support_resolution_leave` 校验） |
| `/data/user/0/com.xiaomi.joyose/shared_prefs/teg_config_pref.xml`               | `pref_local_max_version`                                  | **teg 云控拉取的基准 version**（`teg-freeze` 刷的就是这个）                 |
| `/data/user/0/com.miui.securitycenter/shared_prefs/*.xml`                       | `<pkg>_ve_switch`                                         | securitycenter UI 侧开关状态（`GameBoxVisionEnhanceUtils.F/m0`）            |

### 一键诊断脚本

```bash
adb shell '
PKG=com.miHoYo.hkrpg
echo "=== vendor flags ==="
getprop | grep -E "gpp.frc.support|xiaomi.sr.support"
echo
echo "=== gameInfo events for $PKG ==="
grep "$PKG" /data/system/mcd/gameInfo 2>/dev/null || echo "(no entries)"
echo
echo "=== joyose SP state for $PKG ==="
grep -E "customize_switch_$PKG|customize_enhance_status_$PKG|G_RENDER_RESOLUTION_$PKG|RAW_PICTURE_QUALITY_$PKG" \
    /data/user/0/com.xiaomi.joyose/shared_prefs/*.xml 2>/dev/null
echo
echo "=== teg pref_local_max_version ==="
grep pref_local_max_version /data/user/0/com.xiaomi.joyose/shared_prefs/teg_config_pref.xml
echo
echo "=== pid / libmivk / libmigl ==="
PID=$(pidof "$PKG")
echo "pid=$PID"
[ -n "$PID" ] && cat /proc/$PID/maps | grep -E "libmivk|libmigl" | head -5
echo
echo "=== Joyose pid ==="
pidof com.xiaomi.joyose
'
```

### `customize_game_params.game_mifisr_config` 字符串格式

每条条目：

```text
<pkg>_<minFps>#<targetFps>#<srcFpsList>#<T1>#<T2>#<T3>#<T4>
```

- 包名和字段主体用**第一个 `_`** 分（包名禁止含 `_`）
- 字段主体**七段用 `#` 分**
- `minFps` / `targetFps`: `-1` 表示让驱动自动选
- `srcFpsList`: 逗号分隔多个源帧率，例如原神 `45,60` / 星铁 `60`
- `T1`..`T4`: 温控四档降档阈值（℃），T1 ≥ T2 ≥ T3 ≥ T4
- 实测 Ultra 默认 `47/45/44/42`

Parser 在 `src/parsers/mifisr-string.ts`。`parseMifisr` / `serializeMifisr` 必须
字节级 round-trip（harness 强制校验）。

### `dp_fi_config` 字符串格式（DMI 要用）

每条条目：

```text
<pkg>_<srcFps1>,<targetFps1>[;<srcFps2>,<targetFps2>...]
```

`l.b.q()` 根据当前游戏源帧率在 `;` 分段里找匹配 `<srcFps>`，返回对应 `<targetFps>`。
没有匹配条目 → DMI 启动时 `q()` 返回 -1 → 直接中止，FI 勾选框看着亮但不干活。

**MifisrView 默认不自动维护 `dp_fi_config`** —— 因为 `mifisrStandard()` 返回的
三条 policy 全部绑 `strategy: MIFISR`，而 MIFISR 不读 `dp_fi_config`。`syncDpFiConfig` /
`deriveDpFiValue` / `removeFromDpFiConfig` 函数仍保留在 [src/ui/MifisrView.vue](src/ui/MifisrView.vue)
里作为**专家模式工具**：只有用户在 JsonEditorView 手动把某条 policy 的 strategy 改成
`DMI` 时才需要维护对应的 `<pkg>_<srcFps,targetFps>` 条目，否则 DMI 激活时 `q()`
返回 -1 静默退出。删包流程（`removeCurrent`）仍会调 `removeFromDpFiConfig`
清理历史残留。

### MIVK `support_module` 里的 `mifi` / `misr`（独立渲染通道，非 MIFISR 依赖）

`mivk_settings.app_params[*].xrender_config.support_module` 是 `"name:level"` 数组：

```json
["mifi:31", "misr:7", "vrs:0", ...]
```

- `mifi` = Mi Frame Insertion（小米帧插值 module）
- `misr` = Mi Super Resolution（小米超分 module）
- level：`0` = 关；非 `0` = 开（`q0.s` 判断用的是 `!entry.endsWith("0")`，
  `7 = 0b111` 中档 / `31 = 0b11111` 全开 都被当作"开"，level 数值主要是给驱动
  内部分级用）

**关键修正 —— `l.i`（MIFISR 策略类）并不依赖 `mivk_settings` / `support_module`。**
反编译 `l.i.n()`（MIFISR 激活路径）和 `l.i.f()`（恢复路径）完整源码都没有任何一处
读 `mivk_settings`、`support_module`、mifi level、misr level。`l.i` 做的只是：
激活 strategy 状态 (`aVar.j(i2)` + `s.y().J()`)、设置 refresh rate
(`com.xiaomi.joyose.utils.k.q()`)、停掉旧温控 / PID 监测线程
(`s0.t.u()` / `s0.o.m()`，注意这俩是 `GameThermalMonitor` 和 `GamePidMonitor`，
**不是** MIVK 接触点，早前 CLAUDE.md 里把它们当作 MIVK hook 是错的)。

真正读 `support_module` 的是 `q0.s`（tag `SmartPhoneTag_MiGLConfig`）—— 这是
MiGL / MiVK **共用的 xrender 配置生成器**。它把每条 `"name:level"` 解析后按
level ≠ 0 置 `xrender_BaseInfo` 的 bitmap 位，再把 name 写入 `xrender_ModuleList`，
下发到渲染 hook（进程注入时由 hook 决定哪些 module 实际启用）：

```java
// q0.s
String name = P.get(i).split(":")[0].trim();
if (!P.get(i).endsWith("0")) {         // level != 0 就 enable
    xrenderModuleList.add(name);
    xrenderBaseInfo |= (bit << (i + 18));
}
```

所以 17 系列"画质增强"的完整链路其实是**两条并列通道**：

```text
Joyose 策略层 (l.i)              MiVK 渲染 hook 层 (q0.s → xrender)
───────────────────              ────────────────────────────────
customize_game_params        →   mivk_settings.support_module
fisr_config                  →   (name:level 数组)
l.i.n() 激活 strategy 状态    →   xrender_BaseInfo 位图
→ Joyose 告诉系统走 MIFISR 路由  → 渲染 hook 决定 mifi/misr 是否注入进程
```

- 只有策略层 → UI 勾选框能勾起、Joyose 状态切到 enhance，**但画面没插帧 / 超分效果**
  （渲染 hook 没启用对应 module）
- 只有渲染层 → hook 启用了但没有路由触发，不工作
- 两条都满足 → 实际的插帧 / 超分才看得见

实机对照表（反映的是"画面效果完整性"，不是"MIFISR 能否激活"）：

| 机型/游戏         | mifi | misr      | 备注                                      |
| ----------------- | ---- | --------- | ----------------------------------------- |
| 17 Ultra 原神     | `31` | `31`      | 两条通道都满 —— 画面效果完整              |
| 17 Ultra 星铁     | `31` | `7`       | 都 > 0；misr 走中档（避免细线糊）         |
| 17 Pro Max (原厂) | `0`  | `0` / `7` | 渲染层未启用 —— MIFISR 能激活但画面无效果 |

即使 level = 31，`mifi` / `misr` 各自的同目录参数块（`misr.manual_sr_size_config`
/ `mifi.screen_vu_type` 等）也**必须**配好才能让渲染驱动真跑起来。MivkView 管这部分；
MifisrView 顶部的 MIVK 诊断 banner 只做**信息性提示**（别把"红"banner 理解成
"MIFISR 不会激活"—— 它会激活，只是画面看不出效果）。

### 模块参数块（module_params）字段字典

`support_module` 下方每个模块（mifi / misr / drr / drr_static / gmem / vrs /
alr / mrp / aptssao / aptbloom / 等）还有**同级的参数块子对象**，完整路径是
`mivk_settings.app_params[*].<module>` 或 `migl_settings.game_params[*].<module>`。
下面的字段字典来自**五台样本 DB 全扫描 + libmivk 反编译 `getModuleData` 系列函数**
的交叉对照。

**Joyose Java 透传**：`com.xiaomi.joyose.utils.y.b()` 读 JSON → `r0.o` 的
`X()` / `W()` / `U()` / `T()` / `V()` 等 setter 存到字段 `K` / `O` / `Q` / `R` 等，
**不解释任何字段语义**。真正的 parser 在 libmivk / libmigl / libframeestimationGL3
的 per-game Module。

**通用编码约定**（所有模块复用，不再逐字段重复）：

- `<field>_by_temp_T` / `<field>_by_temp_M`：T = 电池温度（thermal），M = 机身
  温度（mobile surface）；格式 `"温度阈值:档位;温度阈值:档位;..."`
- `"WxH"` 字符串：libmivk 消费点统一走 `MiVkSettings::Split('x') + std::stoi`
- `"src->dst"` 格式：`split("->")` 拿到 src/dst 两个 "WxH" 再各自拆分
- bool 字符串（`"true"` / `"false"`）：消费端 lowercase 后用 4 字节指纹
  `0x1702195828`（小端 ASCII `"true"`）匹配，存 1 字节 bool

#### mivk 通道字段

**`misr`**：

| 字段 | 类型 | 语义 | 样本值 | libmivk 消费点 | 置信度 |
|---|---|---|---|---|---|
| `backbuffer_size` | `"WxH"` | SR 输入分辨率 | `"1920x883"` | `MiVkYuanshenMISRModuleData::getMiSRInfo` @ `0x588290` → width→offset+40、height→offset+44 | 高 |
| `manual_sr_size_config` | `string[]` | SR 升采样映射表，元素 `"Wsrc x Hsrc -> Wdst x Hdst"` | `["1920x882->2608x1200"]`；红米 `["774x1680->1200x2608"]`（竖屏方向） | 同上 → vector offset+48（16B struct per entry） | 高 |
| `force_on_mode_resolution_level` | int | 推测 UI 画质档位强锁值（样本全 `5`） | `5` | **三 so 字符串表都无命中**，疑似通过 `AFME_GL_Init` flags bits 传递 | 低 |
| `disable_scene_list` | array | 场景 ID 黑名单（空 = 不禁用） | `[]`（仅红米） | 未反编译到消费点 | 中 |

**`drr_static`**：

| 字段 | 类型 | 语义 | 样本值 | libmivk 消费点 |
|---|---|---|---|---|
| `config` | `"WrxHr->WpxHp"` | 渲染分辨率 → 呈现分辨率 | `"2160x994->1680x774"` | `MiVkStarRailDRRModuleData` 构造 @ `0x4f77bc`：render int offset+12/+16、present float offset+20/+24 |

`drr_static` 和温控版 `drr`（migl 通道下有）并列存在，后者多 `drr_by_temp_T/M`
温控档和 `params`。

**`gmem`**：

| 字段 | 类型 | 语义 | 样本值 | libmivk 消费点 |
|---|---|---|---|---|
| `image_dimension` | string | tile binning 配置，每分辨率最多 3 个 int 槽 | 17 系列 `"1680x774:37,43;2160x994:37"`；红米 `"1568x720:43,122"`（`:122` 独家） | `MiVkGmemModuleCommon::GetModuleData` @ `0x41ea88` nested `split(";") → split(":") → split("x") + stoi`，存入 vector `<{w, h, n0, n1, n2}>` |

n 值集合 `{37, 43, 122}` **不是** VkFormat（数字巧合）；下游消费点尚未追到，
可能是内部 bin 编号 / tile layout 索引 / pass ID 之类。

**`mifi`**：

| 字段 | 类型 | 语义 | 样本值 | libmivk 消费点 |
|---|---|---|---|---|
| `screen_vu_type` | int | VU 模式枚举（观测 `2`） | `2` | `MiVkYuanshenMIFIModuleData::getMifiMisrParams` @ `0x58546c` → offset+68 |
| `is_use_mask_image` | bool | 是否喂 mask 图像（VRS / foveated 线索，用途未完全定位） | `false` | 同上 → offset+80（指纹法） |
| `is_use_multi_sample` | bool | MSAA 启用 | `true` | 同上 → offset+81（指纹法） |
| `original_backbuffer_size` | `"WxH"` | 原始未改 backbuffer（用于检测尺寸变更） | `"1920x883"` | 同上 → offset+40/+44 |

**`vrs`**（仅 15/15 Pro/Redmi mivk 通道）：

| 字段 | 样本 |
|---|---|
| `vrs_by_temp_T` / `vrs_by_temp_M` | `"30:1;40:2"` |
| `params` | **per-game 方言**：hkrpg `"2160x970,1680x756;3;2,4,3,1;2;1030101"`（6 段）；yuanshen `"1920x862,1620x728‖960x431,810x364;0.018\|0.0008,0.027\|0.001,0.038\|0.0018;0.5,3.0;448,240,...;112,4,...;8388608,21120,1,0;"`（9 段带 `‖`） |

**`alr`**（仅 Redmi K90 Pro Max mivk，hkrpg 专属）：

| 字段 | 样本 |
|---|---|
| `alr_by_temp_T` / `alr_by_temp_M` | `"20:1;31:2;42:3"` |
| `smooth_interval` | `"15,30,1"` |
| `level_mask` | `"1,3,7"` |

#### migl 通道字段

**`drr`**（动态 DRR 含温控）：`drr_by_temp_T/M` + `params` 形如
`"2608x1200;0.9,0.8,0.7"`（分辨率 + 3 档降档系数）。

**`aptssao`**（Adaptive SSAO）：`aptssao_by_temp_T/M` + `res_params` **per-game 方言**
（王者荣耀 `"2608x1200,2352x1080;0.6,0.5,0.0;3;1;0.7"` 长格式 vs 崩铁
`"0.5,0.2,0.0"` 短格式）。

**`mifi`**（所有五台都有）：`fi_enable` (int，`1`=开) + `fi_config` 嵌套 object：

- **全机型共 5 字段**：`deferredDisplayMode` / `extraMode` / `singleFrameMode` /
  `needInputUI` / `enableDebugLog`
- **Xiaomi 15 扩展字段**：
  - `res_render_size_list` `"1848x830"`
  - `res_render_size_80p_list` `"1478x664"`
  - `res_backbuffer_size_list` `"1848x831;1920x863"`
  - `res_display_resolution_map` `"WQHD:2670x1200"`
  - `res_remode_map` `"-1:1800x810;1:1800x810;2:1920x863;3:2403x1080;4:2669x1200;5:2660x1200"`
    —— **mode 档位 1~5 到实际渲染分辨率的映射表，mode 5 = 2660x1200 最高档**
- **红米独家** `useAFME: 1` —— 字符串在 libmigl .rodata 存在但**无代码 xref，
  实测是死字段**（`AutoAFMECoreSR::init` 的 dlopen libframeestimationGL3 无条件
  执行）

**`vrs`**（仅 15/15 Pro migl，yuanshen/hkrpg）：字段最复杂，除温控外还有
`res_motion_params` / `rate_config_params` / `label_info`（**原神 shader 路径
正则分类**，5 类：hero/ground/wall/eff/mp）/ `label_level` / `mainPass_att`
（`"2160,970,1680,756"`）。

**`mrp`**（Multi-Resolution Pipeline）：`post_process_high` 多档 post-process
分辨率梯度（红米 6 档 `"2160x994;1680x774;1440x663;1440x662;1280x589;1080x497"`）
+ `max_resolution`（单值或分号列表）。

**`alr` / `aptbloom`**：`alr` 字段同 mivk.alr；`aptbloom` 仅 15/15 Pro sgame 独家
（`aptbloom_by_temp_T/M`）。

**`support_common_module`**：仅 15/15 Pro sgame 下发 `[]` 空数组占位，语义未定
（**不是** `xrender_config.supoport_common_module` 那个 Joyose 自带 typo 字段；拼写
不同）。

#### per-game 方言差异警示

下列字段**在不同游戏下编码格式完全不同**，写 parser 必须按 `entry.app` /
`entry.game` 分派，不能写通用 schema：

- `mivk.vrs.params` —— hkrpg 6 段分号 / yuanshen 9 段带 `||` 分隔符
- `migl.vrs.*` —— 原神有 `label_info` shader 分类层，崩铁有 `res_motion_params`
  的参数链
- `migl.aptssao.res_params` —— 王者长格式 / 崩铁短格式
- `mivk.gmem.image_dimension` —— n 值集合按机型变化（17 系列 `{37, 43}`，红米
  追加 `122`）

#### 暂无 so 层证据的字段

- `mivk.misr.force_on_mode_resolution_level` —— libmivk / libmigl /
  libframeestimationGL3 三 so 字符串表都无命中，最可能是历史残留 / 云控死字段，
  或通过 `AFME_GL_Init` flags bits 隐式传递
- `mivk.gmem.image_dimension` 的 `n0/n1/n2` 下游消费点 —— parser 存到 vector
  但读取点还未追到
- `migl.mifi.fi_config.useAFME` —— 字符串存在但无 xref，dlopen 无条件，死字段

#### 五台 DB 模块覆盖矩阵

| 机型 | mivk 通道模块 | migl 通道模块 |
|---|---|---|
| 17 Pro Max | misr×2 · mifi×1 · drr_static×1 · gmem×3 | drr×1 · aptssao×1 · mifi×1 |
| 17 Ultra | 同 17 PM | 同 17 PM |
| Xiaomi 15 | **只有 vrs×2**（无 misr / mifi / drr / gmem） | mifi×2 · vrs×2 · aptssao×1 · alr×1 · mrp×1 · aptbloom×1 · support_common_module×1 |
| Xiaomi 15 Pro | 同 15 | 同 15 |
| Redmi K90 Pro Max | misr×1 · drr_static×1 · gmem×2 · **vrs×2** · **alr×1** | drr×1 · aptssao×1 · mifi×1 · mrp×1 |

关键观察：17 PM / 17 Ultra mivk 完全一致（17 系列云控统一下发）；15/15 Pro mivk
**完全没有** misr/mifi/drr/gmem（骁龙老 AFME 通路不经 MIVK 侧 module_params）；
红米是混合形态（既有 17 同款 misr/drr/gmem，又有 15 同款 vrs，再加独有 alr）。

---

## 云控下发机制（已反编译验证，**2026-04 修订**）

之前版本把 `SmartP.db` 当作 Joyose 运行时主源、`teg_config.db` 当镜像——实测反编译
`com.xiaomi.teg.config` 包之后应该**反过来**：

```text
MIUI 云端
   │ HTTP POST /cloud/app/getData  body={version: pref_local_max_version}
   ▼
teg SDK (com.xiaomi.teg.config)   ← 每约 13 分钟 / update_miui_cloud_profile 广播 / 网络变化触发
   │
   │ if (cloud.data.maxVersion == sp_local_max) → no-op（整个不下发）
   │ else → 按 status=0/1 对 rule_id 执行 delete/insert/update 到 teg_config.db
   │        **没有 per-rule version 比较**
   ▼
teg_config.db.rules  ← 被无条件覆盖的本地 DB
   │
   │ teg SDK 同步通知所有 ConfigObserver
   ▼
f.f.u() → e0.z.f0(JSON)  ← Joyose 运行时内存唯一重建入口
   │ 数据来源是 CloudConfig.getDataLists(module) ← 读 teg_config.db.rules
   ▼
k.e.f3250c (customize_game_params) + e0.b0 (fisr_config) + ...
   ▼
GPUTunerService.isSupportGameEnhancePkg(pkg) ← 游戏助手 binder 查询
```

### 关键不变式（修正版）

1. **teg SDK 不读 `SmartP.cloud_config.version`，也不读 `teg_config.rules.rule_version`**。
   它用的是自己在 `SharedPreferences("teg_config_pref")` 里存的
   `pref_local_max_version`（long，默认 0）。
2. **teg SDK apply 差异到 `teg_config.db.rules` 时没做 version 比较**。
   只要 `cloud.data.maxVersion != sp_local_max` 就无条件按云端 delete/insert/update。
3. **Joyose 运行时通过 `CloudConfig.getDataLists(module_key)` 读 teg_config.db**，
   这是 `f.f.u() → e0.z.f0()` 的数据来源。`CloudConfig.getString/getInt/...` 在 Joyose
   代码里**无调用者**。
4. **SmartP.cloud_config 的 `version` 字段不在 teg SDK 的决策面上**。Joyose 自己的某些
   路径可能仍然读 SmartP，但 MIFISR / FI / SR 的支持性判定（本项目主要场景）全部走
   teg 通道。

### 真正的防云控覆盖方法 —— 冻结 `pref_local_max_version`

把 `/data/user/0/com.xiaomi.joyose/shared_prefs/teg_config_pref.xml` 里的
`pref_local_max_version` 刷到 `Long.MAX_VALUE`（`9223372036854775807`）。之后：

- teg SDK 上报 `version=Long.MAX_VALUE`
- 云端 `maxVersion` 只可能 ≤ 此值
- `com.xiaomi.teg.config.f.a()` 进入 `jE == j2` 分支 → 返回 `f2840b=false` →
  不 apply rules、不通知 observers、**`e0.z.f0` 不会被触发**

[module/bin/joyose-edit.sh](module/bin/joyose-edit.sh) 里的 `teg-freeze` 子命令
就是做这件事：先 `am force-stop com.xiaomi.joyose`（必须，SharedPreferences 有
进程内缓存），再用 `sed` 改 XML。`teg-unfreeze` 写回 0 恢复正常。

UI 入口在 [src/ui/LockView.vue](src/ui/LockView.vue) 第一个面板。**副作用**：冻结
是对整个 teg SDK 的，所有走 teg 的云控模块（不只 booster_config）都会停下来。

### DB 侧 `version` 字段的地位（补充说明）

[src/state/session.ts](src/state/session.ts) 里 `lockCloudVersion()` 仍然把
`cloud_config.version` / `params.header.version` / `rules.rule_version` 刷到
`2099xxxx`。**根据当前反编译证据，teg SDK 不读这些字段**，所以单靠它拦不住云控
覆盖。保留这个功能是因为：

- Joyose 内部可能有 **未反编译到的**备用路径读 `SmartP.cloud_config.version`
- 保持 version 数值单调递增是云控生态里的良好惯例（避免本地显得"比云端旧"）
- 便于用户一眼看出哪些 config_name 处于"锁定"状态

但**要真正防住云控覆盖必须用 `teg-freeze`**，version 锁只是辅助。

### 触发 `e0.z.f0` 重建的完整事件面

`f.f.u()`（唯一调用 `e0.z.f0` 的桥梁）被以下事件触发：

- **Joyose 冷启动**（`f.f.<init>` / 其他 init 路径）
- **teg SDK `ConfigObserver.onChanged`**（只在云端真的下发新规则时触发 —— 冻结后永远不触发）
- **`profile_local` 广播**（手动调试用）
- **`PACKAGE_ADDED` / `PACKAGE_REMOVED` 广播**（Joyose 自己注册的）
- **`Settings.Secure.device_provisioned` 变化**（ContentObserver）

冻结 teg 只是断了第 2 条；其他 4 条仍会让 Joyose reload 一次 —— 但因为 `teg_config.db`
已经被冻住，reload 读出来的还是用户改过的数据，内存仍是用户值。

### 实务要点（修正版）

1. **运行时真源是 teg_config.db，不是 SmartP.cloud_config**。`pushAll` 里
   `syncRuleContent()` 自动把 cloud_config.params 镜像到 teg rules.rule_content
   —— 真正对游戏助手可见的改动就是通过这次镜像生效的。
2. 想改动**真正稳定保留**，必须调用 `bridge.tegFreeze()`。`pushAll` 本身不自动冻结
   teg（因为这是个"一次性"的保护动作，用户意图明确时才做）。
3. 红米路径的 `teg_config.db.rules` 是空表，`pushAll` 对空 rules 有跳过分支。
   这种机型上 `teg-freeze` 仍然有意义（teg SDK SP 依然会被用），只是本地 DB 本来就没
   画质增强相关 rule。
4. 想"临时测试一条云控改动"而不锁定：改完 push 就能看到效果；但只要 teg SDK 下一次
   拉到云端新版本就会被覆盖。想稳就冻结。

---

## MIFISR 底层链路：Joyose → libmigl（已反编译验证）

Joyose 策略层只是 scheduler + notifier，**真正的插帧 / 超分最终执行在三个 so 的协作里**：
小米 `libmigl.so`（GL 后端，逐游戏 EGL hook）、小米 `libmivk.so`（VK 后端调度）、
以及 Qualcomm `libframeestimationVK.so`（VK 侧光流算法，见下方「Vulkan 侧的双 so 组合」）。
市场代号 "**Elite Gaming**" 是小米 17 系列 + 骁龙 8 Elite 2 的 joint branding。

### 端到端数据流

```text
Joyose (system_server)
  │ setEnhanceStatus(pkg, 0/1/2/4)
  │ ┌───────────────────┐
  │ │ 广播双通道并行      │
  │ ▼                   ▼
  │ binder             文件
  │ IGameInfoUpdate    /data/system/mcd/gameInfo
  │ ::updateGameInfo   （Joyose 自己 createNewFile + chmod 0666；
  │ (string)            行格式 "<eventId> <pkg> <args...>"）
  ▼
libmigl.so   （每个游戏进程 dlopen 一份；SELinux mcd_data_file 允许访问）
  │
  │  eventProcess() 启动时把 BnGameInfoUpdate binder 注册回
  │  "xiaomi.joyose" service (transact code=35)
  │  → Joyose 之后只用 binder 推送，文件通道是备份
  │
  ├─ GameInfoMonitor::updateGameInfo(str)
  │    │ istringstream >> eventId
  │    │ switch(eventId):
  │    │   1 → parseGameSceneId
  │    │   2 → parseGamePictureInfo
  │    │   3 → parseGameVrsAptLevel (6 个 level)
  │    │   4 → parseCommonVrs
  │    │   5 → parseFilterConfig
  │    │   7 → parseMiFiEnable        → this.mifiEnable (+84)
  │    │  10 → parseALRLevel
  │    │  11 → parseFISREnable        → this.fisrEnable (+88)  ★
  │    │  12 → parseCompositeSceneType
  │    └ 写之前都 strcmp(currentPkg, pkg) 守卫，只接受当前包的事件
  │
  ├─ MiGLSettings::getFISREnable()  ← 消费者入口（仅 libmigl 内部用）
  │
  └─ 每帧结束（MiGL<Game><Feat>Module::PostFrameEnd）
       │
       ▼
     MAFE2XX::Instance()  （每个游戏自己的 pipeline 单例）
       · Yuanshen → MAFE2GS
       · Hkrpg    → MAFE2XQ
       · 其他游戏各自一个
       │
       ├ status 位图分解（和 Joyose `enhance.b.setEnhanceStatus` 字节级吻合）：
       │   enableFI = (fisrEnable == 1 || fisrEnable == 4)
       │   enableSR = (fisrEnable == 2 || fisrEnable == 4)
       │
       ├ FI pipeline：EstimationProcess → FrameEstimationCore → BlitSwapframe
       │   · 用 GL hook 收集 velocity / depth / view matrix / jitter
       │   · 基于前后帧做光流估计，合成中间帧
       │   · Blit 到 SwapBuffer 队列，让用户看到双倍帧率
       │
       └ SR pipeline：CheckMiSRIsRightConfig → initMISR(w,h) / UpSample
           · 分辨率变化时 NativeWindowManager::setBuffersGeometry 重配 Surface
           · UpSample 做纹理上采样
```

## 底层 so 层：libmivk / libmigl / libframeestimation{VK,GL3}

MIFISR 的画面效果最终落在**四个 so 的协作**上：小米的两个调度层
（`libmivk.so` VK 通路、`libmigl.so` GL 通路）+ Qualcomm 的两个算法层
（`libframeestimationVK.so` VK 光流、`libframeestimationGL3.so` GL 光流 + SGSR2 超分）。
libmivk 是标准 Vulkan Layer（非标准注入 —— ROM 里 `implicit_layer.d` 无 manifest，
小米改造过 `libvulkan.so` 按硬编码路径从 `/system_ext/lib64/` 加载，logcat 里
`mivk added global layer 'hyminjsogtvkgralibper'`，Layer 名混淆推测为对抗米哈游
`mhyprot` 反作弊的 VK Layer chain 白名单检测）；libmigl 走**小米改造过的
`libEGL.so`**（定义了 `MiGLStub` 接口）。两个 Qualcomm 算法 so 都是**运行时
`dlopen`** 而非 NEEDED，原因：非骁龙 8 Elite 2 机型没这些 so，NEEDED 会让整个
调度层加载失败；延迟加载也让非白名单游戏跳过算法层开销；Qualcomm 更新算法库
也不影响调度 so 的 ABI。

两个调度 so 共用一套 binder 接口 `IGameInfoUpdate` 接 Joyose 事件，按 key 后缀
区分：libmigl 注册 `"<pkg>"`，libmivk 注册 `"<pkg>_mivk"`，同一个游戏进程可以
GL + VK 两个 callback 并存。Joyose 的 `s0.s.l0(pkg, text)` 内 `U.n(str, text)`
按 key 查 callback 表分派。所以 **MIFISR = 小米调度壳 + Qualcomm 算法芯**，
不是完整自研栈。

### libmivk.so

标准 Vulkan Layer。三大导出 `MiVkGetInstanceProcAddr` / `MiVkCreateInstance` /
`MiVkCreateDevice`。binder 事件接收器 `android::MiVkAppInfoMonitor`，
`UpdateAppInfo(str)` 解析 `"<eventId> <pkg> <args>"`（对等 GL 侧 `GameInfoMonitor::updateGameInfo`）。
MIFISR 事件走 `ParseVkMiFISRSwitch` 把 status 写到实例 offset `+2796`，消费入口
`MiVkAppInfoMonitor::GetMiFISRStatus()`。VK 侧独家还有 `ParseTargetFps` 事件 ——
GL 侧没有，可能是未来的目标帧率推送通道。

每帧边界在 `PreVkQueuePresentKHR`，其他 hook 点非常密：pipeline 创建 / renderpass
/ command buffer / descriptor set / memory map，每个 per-game Module 注册约 40 个
Vk 函数 Pre/Post hook 收集 TAA pass / velocity attachment / depth buffer 等渲染
信息。`MiVkContextManager::InitInstanceInfo` / `InitDeviceInfo` 在 VK instance /
device 创建后挂 per-game processor。单例是 `vulkan::mivk::MiVkSettings` +
`MiVkStubImpl`，processor / module 基类是 `MiVkProcessorBase` / `MiVkModuleBase`。
per-game 示例：星铁 `MiVkStarRailMIFIModule`（完整 insertion+SR pipeline、~40 个
VK API hook），原神 `MiVkYuanshenProcessor`（构造函数挂 ~20 个子 module lambda，
包含 `MiVkYuanshenMISRModuleData` / `MiVkYuanshenDRRModuleData`，但**无显式命名
MIFI** 的子 module，可能通过运行时 SPV 替换实现），绝区零 `EliteGamingLayerZZZ`。

**VK 侧独家能力** —— `MiVkIPAModuleCommon` 在 `vkCreateShaderModule` /
`vkCreateGraphicsPipelines` hook 点**运行时替换 SPV 字节码**：`CheckSpvMD5` 识别
目标 shader、`ReadIPAShaderFromSpvFile` 读预编译 SPV、`CheckPipelineHasTargetShaderModule`
判断是否要替换。GL 侧无对应机制。原神 VK 渲染大量用这种运行时 shader 替换做
画质 / 性能优化。

**VK 侧独家 parser** —— `MiVkGmemModuleCommon::GetModuleData` @ `0x41ea88`
（0x70c 字节完整函数）解 `mivk_settings.app_params[*].gmem.image_dimension` 字符串
（nested `split(";") → split(":") → split("x") + std::stoi`），存入 vector
`<GmemImageDimInfo{w, h, n0, n1, n2}>`。字段字符串 `"image_dimension"` @ `0x105ed9`、
`"gmem"` @ `0xdbe26`。

**libmivk 内嵌超分** —— 自带 AMD FSR1（EASU+RCAS 开源）+ Mi SR 做空间超分，
per-game 上层 pipeline（`EliteGamingLayerSR` / `EliteGamingLayerZZZ` /
`AFMEGenshinImpactManager` 等）按游戏决定用哪条超分路径。**libmivk 自身就能做
超分，不依赖 libframeestimationVK**；Qualcomm VK 算法 so 只负责光流插帧。

**算法委托** —— libmivk 通过 `dlopen` + `dlsym` 调 libframeestimationVK 的三个
C API，路径 `/system_ext/lib64/libframeestimationVK.so` 的完整字符串在 .rodata
里（`strings libmivk.so | grep libframeestimation` 一眼可见，`readelf -d libmivk.so
| grep NEEDED` 则无命中——这就是 dlopen 运行时加载的特征）。具体 dlopen 点在
`vulkan::mivk::afme::AFMEManager::Initialize()` 或 `AFMECommonManager::InitializeAFMELib`
附近（未深挖确认）。

### libmigl.so

GL 侧调度 Layer。装载链：每个 EGL context 创建时小米改造版 `libEGL.so` 调
`android::MiGLStubImpl::initMiGL(egl_connection_t*, isEarly)`，内部依次执行
`earlyInitMiGL` / `lateInitMiGL` → `android::installMiGLHooks(gl_hooks_t*)` 把
`android::gMiGLHooks` 覆盖进 libEGL 内部约 500 个 GL 函数的 dispatch table →
`next::InitNextFunctions` 保存原函数指针（detour pattern）→
`MiGLRenderContext::InitApplicationProcessor` 按包名选对应 `MiGL<Game>Processor`
→ `Processor::RegisterGlobalGLHooks` → 每个 `Module::RegisterGlobalModuleGLHooks`。
每个 Module 注册约 15 个 GL 函数 Pre/Post hook，每帧边界在 `PostFrameEnd`。

binder 事件接收器 `android::GameInfoMonitor`，`updateGameInfo(str)` 解析 eventId。
MIFISR 走 `parseFISREnable` 把 status 写到 `fisrEnable` 成员（offset `+88`），
配套 `parseMiFiEnable` → 成员 `+84`。eventProcess 启动时把 `BnGameInfoUpdate`
binder 注册回 xiaomi.joyose service（transact code=35）。状态缓存单例
`android::migl::MiGLSettings` 有约 23 个 getter（`getFISREnable` / `getMiFiEnable`
等），消费者**仅在 libmigl 内部**（全系统 grep
`_ZN7android4migl12MiGLSettings13getFISREnableEv` 只有 libmigl 自己引用）。
per-game 示例：`MAFE2GS`（原神）/ `MAFE2XQ`（星铁），`EstimationProcess` /
`FrameEstimationCore` / `BlitSwapframe` 是 FI 核心，`CheckMiSRIsRightConfig` /
`initMISR(w,h)` / `UpSample` 是 SR 核心（分辨率变化时
`NativeWindowManager::setBuffersGeometry` 重配 Surface）。

**两条光流 / 超分 pipeline 并列**：

1. **自家 `MAFE2XX`** —— 前述 per-game 单例（`MAFE2GS` 原神、`MAFE2XQ` 星铁 等），
   每帧在 `PostFrameEnd` 读 `MiGLSettings::getFISREnable()` 按 status 位图分解
   `enableFI` / `enableSR` 两条流水，GL 侧收集 velocity / depth / view matrix /
   jitter 自建光流估计合成中间帧。
2. **Qualcomm 通路** `AutoAFMECoreSR::init()` @ `0x183b04` 无条件 `dlopen`
   `/system_ext/lib64/libframeestimationGL3.so` 并 `dlsym` 四个 C API
   （`AFME_GL_Init` / `Upscale` / `Estimate` / `Destroy`）。上层超分入口
   `AutoAFMECoreSR::UpscaleCoreSGSR2()` @ `0x1879b0` 调 `AFME_GL_Upscale`。
   该路径**不受任何 JSON 字段控制** —— 曾经怀疑 `fi_config.useAFME` 字段影响
   加载，实测该字符串在 libmigl .rodata 里存在但**无任何代码 xref**，是死字段。

两条 pipeline 由 `AutoAFMECoreSR` 做上层调度，按 per-game 决定走哪条。

### libframeestimationVK.so

Qualcomm AFME VK 算法层。Layer 名 `VK_LAYER_QUALCOMM_afme`，marketing 名
**`AFME20-VK`**，内部版本字符串 **`v1.7.1.250819`**（`AFME_VK_Init` 日志，编译
日期 2025-08-19）。**3 个 C 导出**：

| API | 作用 |
|---|---|
| `AFME_VK_Init` | 初始化，返回 opaque handle |
| `AFME_VK_Estimate` | 每帧光流估计 |
| `AFME_VK_Destroy` | 清理 |

**初始化顺序** —— `AFME_VK_Init` 必须在 `CFrameEstimationVK::InitVkContext()`
之后调（错误日志 `"Please call InitVkContext() before Init()"`），印证 libmivk
侧的使用模式：先建 VK context，再 `dlsym` 调 `AFME_VK_Init` 拿到 handle，之后
`Estimate` / `Destroy` 都复用这个 handle。

**光流实现双路径** —— `CFrameEstimationVK::Estimate` /
`CFrameGenVK::PrepareVKOpticalFlowResources` 首选 `VK_NV_optical_flow` 扩展
（Adreno 驱动当前未支持，打 `"UnSupported vkCreateOpticalFlowSessionNV"` 日志后
fallback），实际运行走 `GL_QCOM_motion_estimation`（`glTexEstimateMotionQCOM`），
通过**自建 EGL context** 跨 API 桥接 —— 这正是 libmigl 会出现在 VK 游戏进程
`/proc/<pid>/maps` 里的原因。`CFrameGenBypassVK::GenerateFrame` 是纯 GL 路径 fallback。

**调试属性** `debug.afme20.bypass` / `debug.afme20.debugview` /
`vendor.debug.egl.profiler` —— 检测到 Snapdragon / EGL Profiler 连接时 Layer 自动
禁用。该库**不反向依赖 libmivk**（纯算法库，无回调）。

### libframeestimationGL3.so

Qualcomm AFME GL 算法层 + SGSR2 超分。内部版本字符串 `"AFME3 lib v0.3.250529"`
@ `0x3fb20`（编译 2025-05-29）。**内部自称 AFME3**（而非 AFME20-VK），和 VK 版
是两条独立产品线编号。~982 函数，依赖 libGLESv3 + libEGL + `GL_QCOM_motion_estimation`
扩展。**4 个 C 导出**（比 VK 版多 `Upscale`，GL 侧算法库**同时承担光流 + 超分两条
pipeline**）：

| API | 偏移 | 作用 |
|---|---|---|
| `AFME_GL_Init` | `0x15568` | 初始化，入参 `InitInfo` 结构 |
| `AFME_GL_Upscale` | `0x15644` | SGSR1 / SGSR2 超分 |
| `AFME_GL_Estimate` | `0x156a0` | 每帧光流估计 |
| `AFME_GL_Destroy` | `0x156fc` | 清理 |

**`AFME_GL_Init` 入参结构** `{depth_w, depth_h, color_w, color_h, hr_w, hr_h, flags}`
共 7 个 uint32。flags bit 8~12 解析成 5 个独立标志存到 instance offset+1060，
后续 Upscale 时选择 pipeline 变体。调用方（libmigl）把分辨率和 mode 位通过
**结构体传参**传进来 —— 字段名到 so 层已消失，这解释了为什么
`force_on_mode_resolution_level` 这类 JSON 字段名在 libmivk / libmigl /
libframeestimationGL3 三个 so 里都搜不到字符串。

**超分算法身份坐实 = SGSR2**（Snapdragon Game Super Resolution v2，高通开源）。
字符串证据 @ `0x3ff31~0x3ff96`：`"SGSR version: %d"` / `"SGSR1 Upscale"` /
`"SGSR2 Sharpen"` / `"SGSR Activate"` / `"SGSR Upscale"` / `"SGSR3Pass Convert"`。
SGSR2 的 4 个独立 pass 符号（`CFrameGenGL::*`）：

| pass | 偏移 | 作用 |
|---|---|---|
| `Sgsr2Convert` | `0x19328` | 颜色 / 深度格式转换，3 输入纹理 |
| `Sgsr2Upscale` | `0x194e0` | 超分主 pass，4 输入+1 输出 compute dispatch |
| `Sgsr2Sharpen` | `0x19684` | 可选锐化 pass |
| `Sgsr2Activate` | `0x19788` | motion-aware 融合，5 输入 |

还有 `Sgsr1` @ `0x18e6c` 作为 fallback 保留。版本分派点 `CFrameGenGL::Upscale`
@ `0x18840` 读 flag[1068]：`2` → SGSR2 全 pipeline（Convert→Upscale→可选
Sharpen→Activate）；`1` → SGSR1 单 pass；其他 → 错误日志
`"Unknown SGSR version: %d"`。

**GPU / 驱动 / 调试守卫**（`AFME_GL_Init` 早期）—— 仅允许 **Adreno 830 / 840**
（=骁龙 8 Gen 3 / 8 Elite / 8 Elite 2），Driver 必须 v820~v835；检测到
`debug.egl.profiler` / `vendor.debug.egl.profiler` / `debug.rdoc.*` 时 init 失败
（RenderDoc / Snapdragon Profiler 连着时 AFME 自动禁用，与 VK 版守卫一致）；
`debug.afme.bypass=1` 强制走 70 字节空 pipeline（纯调试用）。

### per-game Processor 白名单

GL / VK 两侧都挂 per-game Processor 调度。**权威清单见
[src/presets/mifisr/known-games.ts](src/presets/mifisr/known-games.ts)**。下面是
按游戏合并后的白名单表：

| 游戏 | 包名（主 + 国际） | libmigl Processor | libmivk Processor | 备注 |
|---|---|---|---|---|
| 原神 | `com.miHoYo.Yuanshen` / `GenshinImpact` | `MiGLYuanshenProcessor`（MIFI） | `MiVkYuanshenProcessor`（MISR / DRR / AFMEGenshinImpact 等 20+） | GL+VK 都有 |
| 崩铁 | `com.miHoYo.hkrpg` / `com.HoYoverse.hkrpgoversea` | `MiGLHkrpgProcessor`（AFME / MIFI / ALR / AptSsao / DRR / VRS） | `MiVkStarRailProcessor`（`MiVkStarRailMIFIModule` hook ~40 个 VK API） | GL+VK 都有（GL 命名 Hkrpg，VK 命名 StarRail） |
| 王者荣耀 | `com.tencent.tmgp.sgame` / 体验服 `sgamece` | `MiGLSGameProcessor`（ABR） | `MiVkSGameProcessor` | GL+VK 都有 |
| 和平精英 | `com.tencent.tmgp.pubgmhd` | `MiGLPUBGProcessor`（IRR） | — | GL 独占 |
| 英雄联盟手游 / Wild Rift | `com.tencent.lolm` / `com.riotgames.league.wildrift` | `MiGLLOLMProcessor` | — | GL 独占 |
| 蛋仔派对 | `com.netease.party` | `MiGLDanZaiProcessor` | — | GL 独占 |
| 元梦之星 | `com.tencent.letsgo` | `MiGLYuanMengProcessor` | — | GL 独占 |
| 宝可梦大集结 | `com.tencent.pokemonunite.cn` | `MiGLPokemonProcessor` | — | GL 独占 |
| 金铲铲之战 | `com.tencent.jkchess` | `MiGLJKChessProcessor` | — | GL 独占 |
| 倩女幽魂 | `com.netease.l10` | `MiGLQNYHProcessor` | — | GL 独占（L10 = 内部代号） |
| 王牌竞速 | `com.netease.aceracer` | `MiGLWPJSProcessor` | — | GL 独占 |
| 决战平安京 | `com.netease.moba` | `MiGLPAJProcessor` | — | GL 独占 |
| 绝区零 | `com.miHoYo.Nap` | — | `MiVkZZZProcessor`（`EliteGamingLayerZZZ`） | VK 独占（Nap = 游戏内城市代号） |
| 鸣潮 | `com.kurogame.mingchao` | — | `MiVkMingChaoProcessor` | VK 独占，**包名字符串不在二进制字符串表**，走外部 JSON 运行时加载 |
| 三角洲行动 | `com.tencent.tmgp.dfm` | — | `MiVkDeltaforceProcessor`（`MiVkDeltaforceSceneRecognizerModule`） | VK 独占 |
| 安兔兔跑分 | `com.antutu.benchmark.full:ue` / `:unreal` | — | `MiVkAntutuProcessor`（SD / VRS / VRSV11） | VK 独占 |

**`MiGLSharedProcessor` 挂靠的共用基类白名单**（无独立 Processor，~15 个 GL
Pre/Post hook）：棋牌 / 卡牌 8 款（哈利波特：魔法觉醒·网易、欢乐斗地主·腾讯、
欢乐麻将·腾讯、天天象棋·腾讯、三国杀·小米版、掼蛋·小米版、欢喜斗地主·小米版、
象棋·小米版），视频 app 2 款（腾讯视频 `com.tencent.qqlive`、抖音
`com.ss.android.ugc.aweme` —— **非游戏** 也在白名单内，说明 MiGL hook 对视频帧
插值场景同样生效）。

**白名单总规模**：GL 侧 12 款独立 Processor + 10 款 Shared 挂靠 = 22 款；VK 侧
7 款独立 Processor。交集 3 款（原神、崩铁、王者）。包名后缀约定：`.mi` = 小米
应用商店版 apk（MIUI 渠道）；`.bilibili` = B 服；`oversea` / `HoYoverse` 前缀 =
海外/国际发行版本。

**鸣潮这类新 Processor 的加载机制**走外部 JSON 运行时加载（而非硬编码在二进制
字符串表），未来二进制字符串扫描将不能反映完整白名单 —— 真机上发现
known-games.ts 缺了某游戏欢迎反馈补充。

### 对项目的关键启示

1. **`feature:SR + strategy:MIFISR` 是合法配置** —— Ultra 云控原厂就这么下发，
   `l.i.k(pkg, 2)` 动态把 MIFISR 实例校准成 status=2，下游 libmigl / libmivk 收
   event 11 参数是 2，消费端分解成 `enableSR=true, enableFI=false`，只走超分。
2. **MIFISR 按游戏硬编码 —— 只有 per-game 白名单游戏改 DB 才能在画面上看到效果**。
   包名不在上表里，`mifisrStandard()` 下发的配置虽然能让 Joyose 策略层激活
   （eventId 11 照推），但渲染层没对应 Processor / Module 响应，画面上看不到
   效果。GL / VK 白名单不完全重叠，真机需 `cat /proc/<pid>/maps | grep -E
   'libmigl|libmivk'` 确认游戏当前渲染后端。
3. **gameInfo 文件是备份通道，binder 是主通道**。用户改 SmartP 不会影响 binder
   推送行为；文件只有其他没 binder 的角色（bootanim / surfaceflinger 探测）会
   尝试读，且受 SELinux `mcd_data_file` 标签保护。

### 运行时诊断命令

```bash
# 确认游戏进程加载了全部 4 个 so（Vulkan 游戏期望 4 行，GL 游戏期望 2~3 行）
adb shell 'PID=$(pidof com.miHoYo.hkrpg); grep -E "libmivk|libmigl|libframeestimation" /proc/$PID/maps | awk "{print \$NF}" | sort -u'
# 期望（Vulkan 后端游戏）：
# /system_ext/lib64/libframeestimationGL3.so
# /system_ext/lib64/libframeestimationVK.so
# /system_ext/lib64/libmigl.so
# /system_ext/lib64/libmivk.so

# 验证两个调度 so 的 dlopen 目标 + 算法库导出符号名
adb shell 'strings /system_ext/lib64/libmivk.so | grep -E "libframeestimation|AFME_VK_"'
# 期望：libframeestimationVK.so 路径 + AFME_VK_Init / Estimate / Destroy 三个符号
adb shell 'strings /system_ext/lib64/libmigl.so | grep -E "libframeestimation|AFME_GL_"'
# 期望：libframeestimationGL3.so 路径 + AFME_GL_Init / Upscale / Estimate / Destroy 四个符号

# 内部版本号
adb shell 'strings /system_ext/lib64/libframeestimationVK.so | grep -E "^v[0-9]|AFME20"'
# 期望命中：AFME20-VK / v1.7.1.250819
adb shell 'strings /system_ext/lib64/libframeestimationGL3.so | grep "AFME3 lib"'
# 期望命中：AFME3 lib v0.3.250529

# 关算法层（仅调试用）
adb shell setprop debug.afme20.bypass 1      # VK 通路旁路
adb shell setprop debug.afme.bypass 1        # GL 通路旁路
```

---

## 其他路径（未实测，仅基于样本 DB 形状保留）

> 以下三条通路的代码 **项目维护者没有实机验证**。parser / harness 能处理样本 DB
> 里出现的字符串（字节级 round-trip），但真机上改 DB 是否生效**不保证**。

### 骁龙 8 Elite（15 / 15 Pro）：AFME / FRC / FSR

`fisr_config.strategy` 值：`AFME`、`FRC`、`FSR`。per-game 配置在
`frc_game_params`，字符串格式是下划线 11 字段（`parseFrc` / `serializeFrc`）：

```text
<pkg>_<minFps>_<targetFps>_<srcFps>_<modeFps>_<T1>_<T2>_<T3>_<T4>_<resolution>_<fi>_<sr>
```

白名单字段：`support_resolution_enhance_config` + `support_enhance_targetfps`。
FrcView 提供编辑入口，受 "backendMismatch" 守卫限制只在 `activeBackend==='qualcomm'`
时可达。

### 红米独显（K90 Pro Max）：Novatek

`fisr_config.strategy` 前缀 `NT#FI` / `NT#FISR`。per-game 配置在
`novatek_game_params`，格式 `<包名>_<SetA>_<SetGpu>_<SetB>`，每 Set 7 个 `#` 字段
（`parseNovatek` / `serializeNovatek`）。社区验证过的温控解锁数值 `95/93/93/91` 作为
"一键温控"按钮写进 NovatekView。`teg_config.db.rules` 表在这种机型上是空的
（`pushAll` 有专门的 `rows.length === 0` 跳过分支）。

### MIVK / MIGL（所有后端横切）

`mivk_settings.app_params` / `migl_settings.game_params` 是渲染 hook 注入通道
（mifi / misr / VRS / HSRE / DRR / GMEM / SceneRecognition 等 module 的
`name:level` 开关）。**与 Joyose 策略层并列**：`l.i`（MIFISR）、`l.b`（DMI）、
`l.j`（MISR）等策略类运行时都不读 mivk_settings；mifi / misr 是否在画面上真正
生效由 `q0.s`（`SmartPhoneTag_MiGLConfig`）独立决定，见上文 "MIVK support_module
里的 mifi / misr" 一节。MivkView 管这部分。

---

## 源码导览

```text
src/
  parsers/                    纯 TS，零 DOM 依赖；npm test 全覆盖
    mifisr-string.ts          ★ MIFISR 格式（# 分 7 字段 + 逗号多 srcFps）
    frc-string.ts             骁龙老 FRC 11 下划线字段（未实测）
    novatek-string.ts         Novatek 独显（未实测）
    fisr-config.ts            策略预设（mifisrStandard / qualcommStandard / novatekStandard）
    mivk-migl.ts              MIVK / MIGL 条目 + support_module 辅助
    support-module.ts         "name:level" 数组编解码 + MODULE_META 语义表
    per-game.ts               mqs_enhance_list / cgame_df / migt / pkg:fps
  db/
    constants.ts              DB 路径 + selector 字面量类型
    dbio.ts                   sql.js 封装 + base64 辅助
    schema.ts                 类型 + detectPaths + detectActiveBackend + 磁盘指纹
  history/
    diff.ts                   扁平 JSON diff
    envelope.ts               teg rule_content 与 cloud params 互同步
    store.ts                  追加式历史文件
  root/
    bridge.ts                 ksu.exec 包装；白名单强制；含 visionStatus()
  state/
    session.ts                单一响应式 store；pullAll / pushAll / activeBackend
    dialog.ts                 Promise 化的应用内 modal（替代 window.prompt/confirm）
    toast.ts                  应用内 toast 队列
    theme.ts                  light/dark/auto 主题
    prefs.ts                  UI 偏好（暂未启用）
  ui/                         Vue 3 SFC
    MifisrView.vue            ★ 17 系列 MIFISR 编辑面板
    FrcView.vue               骁龙老通路编辑面板（未实测）
    NovatekView.vue           红米独显编辑面板（未实测）
    MivkView.vue              MIVK / MIGL 编辑
    ...
  main.ts                     createApp(App).mount('#app')
  App.vue                     外壳 + 按 activeBackend 条件渲染对应面板 nav

module/                       KernelSU 模块载荷
  bin/joyose-edit.sh          **所有**特权操作唯一入口
                              · DB: pull/push/backup/revert/history-*
                              · 进程: restart (am force-stop Joyose)
                              · 属性只读: vision-status
                              · teg SDK SP: teg-status / teg-freeze / teg-unfreeze
  customize.sh                安装钩子
  post-fs-data.sh             首启动备份（不会自动 resetprop vendor flag）
  module.prop

scripts/package-module.mjs    打包 dist/ + module/ 成 release/*.zip
tests/run.mts                 npm test 主体，120 项对五台样本 DB 的端到端校验
```

---

## 命令速查

### 开发

```bash
npm install                # 一次性
npm test                   # Node harness（tsx），120 项对五台样本 DB 端到端
npm run typecheck          # vue-tsc --noEmit
npm run build              # typecheck + vite build
npm run package            # 把 dist/ + module/ 打包成 release/*.zip
npm run dev                # vite 热更新；没有 ksu.exec，桥会报 "unavailable"
```

### 设备诊断（adb）

```bash
# 开 Joyose debug log（y0.d.a 级别才会到 logcat）
adb shell setprop persist.sys.smartop.debug true
adb shell am force-stop com.xiaomi.joyose

# MIFISR 激活核心日志
adb logcat -v time -s SmartPhoneTag_i:V SmartPhoneTag_b:V GPUTunerService:V VulkanModeRecognizer:V

# FISR 激活的物化证据（11 <pkg> <status> 行）
adb shell 'grep hkrpg /data/system/mcd/gameInfo'

# teg SDK 云控基准版本
adb shell 'grep pref_local_max_version /data/user/0/com.xiaomi.joyose/shared_prefs/teg_config_pref.xml'

# 游戏进程是否加载 libmivk / libmigl
PID=$(adb shell pidof com.miHoYo.hkrpg)
adb shell "cat /proc/$PID/maps | grep -E 'libmivk|libmigl'"
```

---

## 样本 DB（`tests/<机型>/`）

**别从仓库删掉**。parser 的 round-trip 测试以它们为锚。

| 目录                       | 机型              | 后端            | 备注                                                                                                 |
| -------------------------- | ----------------- | --------------- | ---------------------------------------------------------------------------------------------------- |
| `tests/Xiaomi 17 Pro Max/` | Xiaomi 17 Pro Max | MIFISR          | **用户主设备**。原厂无 vendor FRC/SR flag；cloud 里 fisr_config 为空，customize_game_params 也没下发 |
| `tests/Xiaomi 17 Ultra/`   | Xiaomi 17 Ultra   | MIFISR          | 已下发 `game_mifisr_config` 4 条 + `feature:SR` policy；vendor FRC/SR flag 都 true                   |
| `tests/Xiaomi 15/`         | Xiaomi 15         | Qualcomm legacy | **未实测**。4 条 FRC 条目 + `feature:FI/strategy:AFME`                                               |
| `tests/Xiaomi 15 Pro/`     | Xiaomi 15 Pro     | Qualcomm legacy | **未实测**。4 FRC + 4 MIGL 游戏                                                                      |
| `tests/Redmi K90 Pro Max/` | Redmi K90 Pro Max | Novatek         | **未实测**。71 条 novatek_game_params；`rules` 表空                                                  |

---

## 不变式（动代码时要遵守）

### 通用

1. **永远别绕过 `bin/joyose-edit.sh`。** TS 桥（`src/root/bridge.ts`）白名单与
   shell 白名单一一对应；新增操作两边一起加。
2. **DB 写回必须保留 uid / gid。** shell 用 `stat -c %u:%g` 实时读原文件。
3. **每次 `pushAll` 都要先自动备份**。`state/session.ts::pushAll` 先调
   `bridge.backup()` 再 `bridge.push(...)`。
4. **`rules.rule_content` 必须镜像 `cloud_config.params`。** `pushAll` 在导出字节前
   自动跑 `syncRuleContent()`。`rules` 为空的红米路径例外。
5. **历史是追加式。** `history-save` 每次新文件；回滚也写新记录。唯一裁剪通道是
   `history-clear <keep>`。
6. **所有 parser 必须逐字节 round-trip。** harness 对五台样本 DB 里每条 per-game
   字符串强制校验 `serialize(parse(s)) === s`。
7. **模块不自动改 vendor 系统属性。** `ro.vendor.gpp.frc.support` /
   `ro.vendor.xiaomi.sr.support` 由用户自己 resetprop 或装别的模块处理；MifisrView
   做只读诊断和手动命令指引。

### MIFISR 专属

1. **`mifisrStandard()` 返回 FI / SR / FISR 三条 policy，strategy 统一 `MIFISR`**
   —— 对齐 17 Ultra 云控原厂下发（Ultra 自己甚至只给一条 `SR+MIFISR`）。历史
   文档里"FI 必须绑 DMI / SR 必须绑 MISR 或 DPQ"的说法基于对 `a()` 的误读，已
   废弃；实际 MIFISR 一个实例就能靠 `k(pkg,status)` 扮演所有 status 码。如果
   用户在 JsonEditorView 里手动把某条 policy 的 strategy 改成 `DMI`，记得同步
   维护 `customize_game_params.dp_fi_config` 里的 `<pkg>_<src,tgt>` 条目。
2. **`support_game_mode` 是 2 位位图 `[MGAME, TGAME]`，`1`=启用 / `0`=禁用**。
   `mifisrStandard()` 默认 `"1#1"`（两模式都启用）；Ultra 云控原厂给 hkrpg 下发
   `"0#1"`（均衡禁、性能启）。用户 2026-04-22 实测确认此语义。
3. **FI 和 FISR policy 的 `support_max_refresh` 必须各自配自己那一条，无 fallback**。
   `l.i.n` 通过 `q.d.f(pkg, status)` 按 status 转 feature 名精确查自己这条
   policy 的字段；**SR 分支不读此字段**。`X#Y` 是均衡 / 性能**两档的 Hz 整数上限**
   （`q.d.c` 走 `split("#") + parseInt(parts[idx])`，MGAME→[0] 非 MGAME→[1]），
   `l.i.r` 做 `min(gameFps*2, supportMaxFps)` 决定最终刷新率。缺失时 Joyose 默认 60，
   两档 FI 倍帧都被 `min(fps*2, 60) = 60` silently clamp 回原帧率。
   `mifisrStandard()` 默认写 `"60#120"`（与 15 系列 AFME / 17 系列 MIFISR
   官方下发一致：均衡档节能 cap 60Hz 等效禁用 FI，性能档满血 120Hz），
   MifisrView 检测缺失会弹 warn banner。
4. **UI 端 FISR 不是独立按钮，是"同时勾 FI + SR"的升级**。
   `isSupportEnhance` 完全不查 FISR feature（`k.e.s = o(FI) || p(SR)`），只看
   FI 或 SR 是否在 policy 列表；只有 FISR 没 FI/SR 时**整个面板消失**。
   `isSupportSuperResolutionWithFrameInsert = !k.e.q = !(FI∧SR∧!FISR)` 决定
   UI 能否走到 status=4 分支。MifisrView 对这些 bad 组合有 error/warn banner。
5. **`support_resolution_leave` 是激活前硬检查**（`l.i.t`）。policy 带此字段
   且当前游戏渲染 level 不在白名单时，打 `invalid render resolution` 并**静默
   拒激活**。留空 = 允许所有分辨率。MifisrView 检测到字段存在时显示 info
   banner 引导用户。
6. **MIFISR 不读 `dp_fi_config`**；本模块走默认 MIFISR 路径时不再自动同步
   `customize_game_params.dp_fi_config`。`syncDpFiConfig` / `deriveDpFiValue`
   函数仍在 MifisrView 里保留，仅供手动切到 DMI 的专家用户使用。
7. **MIFISR 字符串包名禁止含 `_`**（`serializeMifisr` 发现会抛）。跟老 FRC 同约束。
8. **feature 字符串大小写敏感**：必须是 `"FI"` / `"SR"` / `"FISR"`。
9. **`support_vk` 字段只对 `com.miHoYo.hkrpg` 生效**（`k.b.isSupportEnhance`
   和 `q.i.isSupportEnhance` 两处都对 pkg 做精确字符串匹配）。其他游戏的
   group 不管填不填这个字段都等效无害。项目 MifisrView 对星铁高亮"★ 星铁必开"。
10. **防云控覆盖的真正方法是 `teg-freeze`**，不是 LockView 的 DB version 锁。
    teg SDK 比较的是 SharedPreferences 里 `pref_local_max_version`，**不读**
    `SmartP.cloud_config.version` 和 `teg_config.rules.rule_version`；且 apply
    rules 到 `teg_config.db` 时无 per-rule version 比较。LockView 里老的
    `lockCloudVersion` 只作为辅助保留。

### 骁龙 / Novatek 专属（未实测）

1. **FRC 包名里禁止 `_`**（`serializeFrc` 抛）。
2. **Novatek 温控解锁预设 = 95 / 93 / 93 / 91**（社区验证）。

---

## 已知的坑

- **`sql-wasm.wasm` 靠 Vite 插件复制。** 没有 `vite.config.ts` 里的 `copySqlWasm()`
  插件，sql.js 在 WebView 里会 404。
- **红米的 `teg_config.db.rules` 是空表。** `pushAll` 要能容忍，当前 `rows.length === 0`
  时跳过信封同步。
- **`noUnusedLocals` / `noUnusedParameters` 被关了。** 原因是 Vue SFC 的
  `defineProps` 残留会误报；不要擅自打开。
- **17 PM 原厂 vendor 层没声明 FRC/SR flag**，游戏助手直接拒绝渲染画质增强面板。
  改 DB 没用，用户必须自己 resetprop。MifisrView 顶部 banner 显式说明这事。
- **DMI 静默失败**：`l.b.n()` 查不到 `dp_fi_config` 映射时日志是
  "invaild targetFps"（拼写错是原厂这样），`d(str)` 直接中止。UI 勾选框看着亮但不跑。
- **`support_game_mode` 非 "X#Y" 格式会让 Joyose 忽略 mode 位**：`k.e.u()` 的
  `split("#")` 长度 ≠ 2 时代码路径会 fallback 到"只看 feature 是否在列表里"，等价于
  `"1#1"` 但**静默**——不报错、用户不知道 mode 限制失效了。MifisrView 的
  `setGameModeBit` 用 `isValidSupportGameMode` 做运行时校验拒绝非法值；JsonEditorView
  里直接手改仍无保护，评审时注意。
- **`syncDpFiConfig` 从 MIFISR srcFps 派生、不保留云端多段**：
  [src/ui/MifisrView.vue](src/ui/MifisrView.vue) 的 `deriveDpFiValue`/`syncDpFiConfig`
  把 MIFISR 条目的 `srcFpsList` 翻译成 `dp_fi_config` 的 `src,tgt[;src,tgt...]`。
  设计意图是"两边 srcFps 一致"，所以如果云端给某个包的 `dp_fi_config` 预先下发了
  **比 MIFISR srcFps 更多的段**，同步会丢多余段。想保留请去 JsonEditorView 手改。
- **`findGroupForPkg` 没有 "OTHER" 组兜底**：反编译 `k.e.u/f` 在包名未命中直接
  配置时会扫描一次 `game_list.includes("OTHER")` 的组作兜底；
  [src/parsers/fisr-config.ts](src/parsers/fisr-config.ts) 的 `findGroupForPkg`
  只按精确包名匹配。5 台样本 DB 的 `fisr_config` 都没出现过 `"OTHER"` 组，所以
  目前不补；如果观察到云端启用 OTHER 兜底再加逻辑。
- **早期曾观察到"FISR + FI + SR 三条共存会让 FI 花屏"**（2026-04-23 星铁）：
  2026-04-24 追加实测确认，问题根因**不是 FISR policy 本身**，而是当时用的
  `support_max_refresh="120#120"` 把均衡档也拉到 120Hz，叠加 SR / FISR 的功耗
  与显存峰值触发驱动异常。换成官方同款 `60#120`（均衡档 cap 60Hz → FI 被
  `min(60×2, 60)=60` 削回源帧率等效禁用；性能档 `min(60×2, 120)=120` 满血）后，
  三条 policy 共存单开 / 合体 / 单 SR 都正常。
  之前关于 `MiVkStarRailMIFIModule` 按 fisr_config 分叉 FI hook 的推断是对
  libmivk .so 的臆测，没有反编译证据，已翻案。`hasFisrBreaksFi` banner 与
  computed 已从 [src/ui/MifisrView.vue](src/ui/MifisrView.vue) 移除。

---

## 扩展流程

### 加新的 per-game 格式

1. `src/parsers/` 下加 `parseX` + `serializeX` + `validateX` + `blankX`。
2. harness 加 round-trip 校验（对每台样本 DB）。
3. 如果影响导航高亮，改 `src/db/schema.ts::detectPaths` 和 `detectActiveBackend`。
4. `src/ui/` 下加专门 Vue view；挂到 `App.vue` 的侧栏；按 `state.activeBackend` 条件渲染。
5. 如果字段被 Joyose 镜像到 `rules.rule_content`，不需要额外管道 —— `pushAll`
   整行同步。

### 加新的 KernelSU 子命令

1. `module/bin/joyose-edit.sh` 加 `cmd_foo` 函数，挂到 `case` 分派。
2. 用 `safe_name` / 数字正则做参数校验。
3. `src/root/bridge.ts` 加打了类型的 wrapper，调 `runKsu()` 后 `JSON.parse`。
4. 尽量用 `DriverBackedStore` 风格的 mock 做单测，参考 `tests/run.mts::runHistoryStoreTest`。

### 反编译 Joyose 查字段语义

项目里的 MIFISR 行为推断都基于反编译 `com.xiaomi.joyose` APK（通过 jadx MCP）。
关键类：

- `com.xiaomi.joyose.enhance.a` —— 主入口，boot 决策
- `com.xiaomi.joyose.enhance.b` —— combiner（setEnhanceStatus 的 0/1/2/4 分派）
- `q.i` —— MIFISREnhanceContext（17 系列用 q.i 的假设已证伪，实际走 k.b）
- `q.g` —— fisr_config JSON parser
- `q.e` —— feature ↔ status 硬编码映射
- `k.b` —— CustomizeEnhanceContext（17 系列实际用的）
- `k.e` —— CustomizeUtil：customize_game_params parser + strategy 注册（`n()` 方法）
- `l.b` / `l.c` / `l.i` / `l.j` —— DMI / DPQ / MIFISR / MISR 策略实现（**策略层，
  不读 mivk_settings**）
- `q0.s` —— MiGL / MiVK `xrender` 配置生成器（tag `SmartPhoneTag_MiGLConfig`），
  解析 `support_module` 的 `name:level` 数组，决定哪些渲染 module 真注入进程
- `s0.t` / `s0.o` —— `GameThermalMonitor` / `GamePidMonitor`，跟 MIVK 无关；
  `l.i.n()` 里调它们俩的 `u()` / `m()` 是停旧监测线程，不是触发 MIVK hook
- `s0.s` —— `SmartPhoneTag_GameSceneIdMonitor`，把事件编码成
  `"<eventId> <pkg> <args>"` 写入 `/data/system/mcd/gameInfo`（eventId 表见 MIFISR
  底层链路一节；MIFISR 走 eventId 11）；同时走 `com.xiaomi.joyose.IJoyoseInterface`
  binder 主动推给 libmigl

---
> Source: [YuKongA/JoyoseEdit](https://github.com/YuKongA/JoyoseEdit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
