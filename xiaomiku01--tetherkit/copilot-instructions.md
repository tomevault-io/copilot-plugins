## tetherkit

> > 本文件是给 AI agent（以及接手的人类）看的**工作记忆**。

# AGENTS.md —— TetherKit 实现备忘

> 本文件是给 AI agent（以及接手的人类）看的**工作记忆**。
> 每完成一个提交都要更新对应章节。开始任何工作前先读一遍本文件，避免重复踩坑。

---

## 1. 项目一句话

macOS **用户态** RNDIS 驱动：USB 侧用 libusb 与 RNDIS 设备（Android 手机 USB 网络共享等）
通话，网卡侧用 `feth` 虚拟网卡对 + BPF 直接读写原始以太帧，把设备变成一张系统可见的网卡。

---

## 2. 已实测确认的环境事实（**不要重复验证，直接引用**）

实测机器：macOS 26.5.1 (Darwin 25.5.0)、Apple Silicon arm64、Apple clang 21.0.0。

### 2.1 工具链

| 事实 | 结论 | 验证方式 |
|---|---|---|
| Apple clang 21 的 C++23 | **可用**：`std::expected`、`std::byteswap`、`std::format`、`jthread`、`stop_token`、`latch`、`counting_semaphore` 全部通过编译+运行 | 实测编译运行 |
| `std::expected` / `std::byteswap` 在 `-std=c++20` 下 | **不可用**，是 C++23 库特性。项目因此定为 C++23 | 实测编译报错 |
| `std::format` 浮点格式化 | 依赖 libc++ 的 `std::to_chars`，带 availability 标注，**部署目标必须 ≥ macOS 13.3**，否则编译失败 | 实测 `-mmacosx-version-min=13.0` 报 `'to_chars' is unavailable: introduced in macOS 13.3` |
| `std::hardware_destructive_interference_size` | Apple libc++ **未提供**。必须自己定义缓存行常量 | 实测编译报错 |
| **缓存行大小** | **128 字节**（不是 64！）`hw.cachelinesize: 128`。SPSC 队列的 false-sharing 填充必须按 128 对齐 | `sysctl hw.cachelinesize` |
| L1D 缓存 | 65536 字节 | `sysctl hw.l1dcachesize` |
| CPU | 10 逻辑核，其中 **4 个性能核**（`hw.perflevel0.logicalcpu = 4`）。数据路径线程需要用 QoS 争取性能核 | `sysctl hw.ncpu hw.perflevel0.logicalcpu` |
| `-mcpu=apple-m1` / `-mcpu=native` | 都可用；但默认**不开**（见 `cmake/Optimizations.cmake` 里的取舍说明） | 实测编译 |
| ninja | **未安装**，用默认 Unix Makefiles 生成器 | `which ninja` |
| CMake | 4.3.3 | `cmake --version` |

### 2.2 libusb

| 事实 | 结论 |
|---|---|
| 版本 | 1.0.30.12037，`/opt/homebrew/opt/libusb` |
| pkg-config | 可用。注意头文件目录是 `.../include/libusb-1.0`（非常规），`FindLibUSB.cmake` 已处理 |
| hotplug | `libusb_has_capability(LIBUSB_CAP_HAS_HOTPLUG)` 返回 **1**，支持 |
| 链接依赖 | darwin 后端需要 `IOKit`、`CoreFoundation`、`Security` 三个 framework |
| **本机 USB 设备** | 立项时为 **0**（`libusb_get_device_list` 与 `ioreg -c IOUSBHostDevice` 都是 0 条），架构因此按「USB 逻辑必须能用 mock 后端离线测试」来设计。后来接上过真实 RNDIS 设备做验证（见第 6 节），但**这个设计约束仍然有效** —— 不要假设跑测试时一定有设备 |

### 2.3 BPF（Darwin）

| 事实 | 结论 |
|---|---|
| **零拷贝 BPF** | **不存在**。SDK 的 `net/bpf.h` 里**没有** `BIOCSETZBUF` / `BIOCGETZMAX` / `BIOCROTZBUF`（FreeBSD 有，Darwin 没有）。只能用经典 BPF：大 `BIOCSBLEN` + 批量 `read()` |
| 可用 ioctl 全集 | `BIOCGBLEN`(102R) `BIOCSBLEN`(102WR) `BIOCSETF`(103) `BIOCFLUSH`(104) `BIOCPROMISC`(105) `BIOCGDLT`(106) `BIOCGETIF`(107) `BIOCSETIF`(108) `BIOCSRTIMEOUT`(109) `BIOCGRTIMEOUT`(110) `BIOCGSTATS`(111) `BIOCIMMEDIATE`(112) `BIOCVERSION`(113) `BIOCGRSIG`(114) `BIOCSRSIG`(115) `BIOCGHDRCMPLT`(116) `BIOCSHDRCMPLT`(117) `BIOCGSEESENT`(118) `BIOCSSEESENT`(119) `BIOCSDLT`(120) `BIOCGDLTLIST`(121) `BIOCSETFNR`(126) |
| `struct bpf_hdr` | `{ struct BPF_TIMEVAL bh_tstamp; bpf_u_int32 bh_caplen; bpf_u_int32 bh_datalen; u_short bh_hdrlen; }` —— 遍历时**必须**用 `bh_hdrlen` 而非 `sizeof(bpf_hdr)` |
| 对齐宏 | `BPF_ALIGNMENT = sizeof(int32_t) = 4`，`BPF_WORDALIGN(x) = ((x)+3) & ~3` |
| 设备节点 | `/dev/bpf0..3` 存在（Darwin 按需克隆更多节点） |
| **批量写** | ✅ **支持**（macOS 14+）：私有 ioctl `BIOCSBATCHWRITE = _IOW('B',143,int) = 0x8004428f`。缓冲格式与 read 对称（连续的 `bpf_hdr + 帧`，每条按 `BPF_WORDALIGN` 对齐），前置条件是 `BIOCSHDRCMPLT=1`。macOS 13 及更早**没有**此 ioctl → 必须运行时特性探测，失败回落逐帧 write |
| 私有 ioctl 编号（实测核对） | `BIOCSBATCHWRITE`=0x8004428f、`BIOCSNOTSTAMP`=0x80044291（关时间戳，省每帧一次 microtime）、`BIOCSWRITEMAX`=0x8004428c、`BIOCSDIRECTION`=0x8004428a、`BIOCSHEADDROP`=0x80044280。macOS 26 把它们从 `net/bpf.h` 挪到了 `net/bpf_private.h`（**SDK 未提供该文件**） |
| `BIOCSBLEN` 上限 | `sysctl debug.bpf_bufsize_cap` = **33554432（32 MiB）**。超限**不报错**，静默截断并通过 `_IOWR` 把实际值**写回参数** |
| `read()` 缓冲长度 | **必须精确等于** `bd_bufsize`，否则 `bpfread` 开头就返回 `EINVAL`。必须用 `BIOCSBLEN` 的写回值 |
| `BIOCSHDRCMPLT` | **必须设为 1**。=0 时 `bpfwrite` 会剥掉前 14 字节重建帧头（源 MAC 被驱动改写）；=1 才走 `DLIL_OUTPUT_FLAGS_RAW` 原样透传。批量写也硬性要求它是 1 |
| 单帧写入长度上限 | `BPF_WRITE_LEEWAY = 18`，`hdrcmplt=1` 时整帧长度必须 ≤ 接口 MTU + 18（MTU=1500 → 1518） |
| `BIOCSRTIMEOUT` 分辨率 | **10 ms**（内核存 `tvtohz(tv)-1` 个 tick，实测 `kern.clockrate hz=100`）。传 `{0,0}` 会变成**永久阻塞** |
| 最优读取模型 | `BIOCIMMEDIATE=1` + **专用线程阻塞 `read()`**，不要用 kqueue。immediate 下每来一包就唤醒，`read()` 醒来时一次性交付期间累积的全部包（低速低延迟、高速自动大批量，行为类似 NAPI），每批只一次系统调用；kqueue 的就绪判据完全一样却要多一次 `kevent()` |
| `/dev/bpf` 克隆节点 | **不存在**（实测 `ls /dev/bpf` → No such file）。必须遍历 `/dev/bpf%d`：`EBUSY` 试下一个、`ENOENT` 到上限。打开当前最后一个节点时内核会按需再造一个。上限 `sysctl debug.bpf_maxdevices` = 256 |
| `access_bpf` 组 | macOS **没有**（那是 FreeBSD 的做法，实测 `dscl . -list /Groups` 无匹配）。`/dev/bpf*` 是 `0600 root:wheel`，只能 root |

### 2.4 feth（if_fake）

| 事实 | 结论 |
|---|---|
| 是否存在 | **存在**，`sysctl net.link.fake.*` 可见 |
| 关键 sysctl | `net.link.fake.max_mtu = 2048`、`tx_headroom = 32`、`buflet_size = 512`、`qset_cnt = 4`、`link_layer_aggregation_factor = 96` |
| 创建权限 | **需要 root**：非 root 下 `ifconfig feth0 create` 返回 `SIOCIFCREATE2: Operation not permitted` |
| `struct ifdrv` | **不在**公开 SDK 的 `net/if.h` 中，必须自行声明。带 `#pragma pack(4)`，LP64 下 `sizeof == 40`（偏移 16/24/32）。**大小参与 ioctl 编号计算**，算错就得到不存在的 ioctl 号 → 已用 `static_assert` 钉死。实测 `SIOCSDRVSPEC = 0x8028697b`、`SIOCGDRVSPEC = 0xc028697b` |
| `net/if_fake_var.h` | **不在**公开 SDK 中。`struct if_fake_request` = `uint64_t reserved[4]`（32 字节，**内核校验必须全零**）+ 128 字节 union，**总 160 字节**。`IF_FAKE_S_CMD_SET_PEER = 1`、`IF_FAKE_G_CMD_GET_PEER = 1` |
| 私有 ABI 的版本风险 | **极低，无需降级到 ifconfig**：`if_fake_var.h` 在 xnu-7195(macOS 11) → xnu-12377(macOS 26) 的所有发布 tag 下文件 md5 完全相同，从未变动。Apple 自己的 `ifconfig fethN peer fethM` 走的就是这套 ABI |
| SET_PEER 的内核校验 | 五项，任一不满足返回 `EINVAL`：① `ifd_len >= 160`；② `reserved` 全零；③ peer 必须也是 feth（`ifnet_name()=="feth"` 且 `IFT_ETHER`）；④ 双方都不能已有 peer；⑤ 需要 root |
| **数据流向语义**（已对照 `feth_output_common()` 源码确认） | 主机从 feth0 发出的帧 → 在 feth1 上是 **input** 方向；我们向 feth1 的 BPF `write()` 一帧 → 走 feth1 的 output → 进入 feth0 的 **input** → 被主机 IP 栈收到，**不会** loopback 回 feth1。所以 BPF 只挂 feth1 一个描述符就能同时收发 |
| `BIOCSSEESENT=0` 的作用 | → `bd_direction = BPF_D_IN`，恰好滤掉**我们自己写进去的帧**（它们在 feth1 上是 output 方向）。**不设会形成回环** |
| `IFF_UP` 要求 | `bpfwrite` 里有硬检查 `if ((ifp->if_flags & IFF_UP) == 0) return ENETDOWN` → feth1 必须 UP；feth0 也必须 UP 才能让 IP 栈处理收到的帧 |
| **创建期 sysctl 快照** | ⚠️ `hwcsum` / `fcs` / `tso_support` / `lro` / `trailer_length` / `separate_frame_header` / `max_mtu` 等开关在 `feth_clone_create()` 那一刻被快照进接口，**创建后再改 sysctl 无效**。若 `hwcsum=1`，我们读到的帧校验和是留给硬件算的假值！本机实测默认全部为 0（正是我们要的），但代码里仍在创建前显式校验（`VerifyFethSysctls()`） |
| 内核分配的 MAC | `'f','e','t','h', unit>>8, unit&0xff` → feth0 = `66:65:74:68:00:00`（0x66 的 bit1=1 → 本地管理地址，合法） |
| MAC 设置策略 | 应把**系统侧**（feth0）的 MAC 设为设备汇报的 `OID_802_3_PERMANENT_ADDRESS`（RNDIS 语义下设备就是这块网卡，对端 ARP/DHCP 都按它建）；**驱动侧必须保留内核分配的不同 MAC**，否则两侧 IPv6 链路本地地址相同会触发 DAD 冲突。改 MAC 必须在 `IFF_UP` 之前 |
| `BIOCPROMISC` | 对 feth **不需要**：`feth_output_common` 无条件把帧投给 peer 并 tap，不做 MAC 过滤，能否读到只由方向决定 |
| DHCP | `sudo ipconfig set feth0 DHCP`（临时服务，只活到下次网络配置变更，不出现在系统设置里）。拆除 `sudo ipconfig set feth0 NONE`。`networksetup` 用不了 —— feth 不在 `SCNetworkInterface` 列表里 |
| 性能上限警示 | 社区报告 feth 路径在超过约 5–8 Gbps 后会出现内核 mbuf 溢出并 panic。对本项目风险很低：RNDIS over USB 2.0 HS 实测约 200–300 Mbps，USB 3 下也难超 1–2 Gbps |
| 相关 ioctl（**在**公开 `sys/sockio.h` 中） | `SIOCSIFFLAGS`(i,16) `SIOCGIFFLAGS`(i,17) `SIOCSIFMTU`(i,52) `SIOCSIFLLADDR`(i,60) `SIOCIFCREATE`(i,120) `SIOCIFDESTROY`(i,121) `SIOCIFCREATE2`(i,122) `SIOCSDRVSPEC`(i,123) `SIOCGDRVSPEC`(i,123) |
| `IFNAMSIZ` | 16 |

### 2.5 开发环境的限制（影响测试策略）

- **无 root**：当前会话以 uid 501 运行，无法创建 feth、无法打开 `/dev/bpf*`。
  → 需要 root 的测试必须**可选、可跳过**，并在 CI/本地用 `TETHERKIT_ROOT_TESTS=1` 之类的开关控制。
- **不保证有 USB 设备**：立项时开发机上一个都没有，后来才接上过真实设备做验证。
  → USB 后端必须抽象成接口，提供内存 loopback mock，端到端测试与吞吐基准都跑在 mock 上。
  自动化测试**不得依赖设备在场**。
- 验证状态与仍未验证的事项统一记录在本文件第 6 节。

---

## 3. 目录结构

```
TetherKit/
├── AGENTS.md              本文件：agent 工作记忆
├── README.md              用户向文档：构建、运行、权限、故障排查
├── CMakeLists.txt         顶层构建脚本
├── .clang-format          代码格式（Google 基线，100 列）
├── .clang-tidy            静态检查与命名约定
├── cmake/
│   ├── FindLibUSB.cmake        libusb-1.0 查找（pkg-config 优先 + Homebrew 回退）
│   ├── CompilerWarnings.cmake  警告选项 INTERFACE 目标
│   ├── Sanitizers.cmake        ASan/UBSan/TSan 开关
│   └── Optimizations.cmake     数据路径优化选项与取舍说明
├── include/tetherkit/     公开头文件（按模块分子目录）
├── src/                   实现
│   ├── version.cc.in           CMake 注入版本号的模板
│   └── app/                    命令行入口
├── tests/                 doctest 单元测试（单一二进制 + test-suite 过滤）
├── benchmarks/            自带轻量 harness 的性能基准
├── third_party/doctest/   vendored doctest 2.4.12 单头文件
└── docs/                  设计文档、协议笔记、性能报告
```

依赖方向**严格单向**，禁止反向或循环：

```
tk_common  ←（无依赖）
   ↑
tk_rndis / tk_net / tk_usb  ← tk_common
   ↑
tk_core    ← common + rndis + net + usb
   ↑
tetherkit（可执行）← core
```

---

## 4. 代码规范（与 `.clang-tidy` 中的 CheckOptions 一致）

| 类别 | 约定 | 例 |
|---|---|---|
| 命名空间 | `lower_case` | `tetherkit`、`tetherkit::rndis` |
| 类型（class/struct/enum/别名） | `CamelCase` | `RndisStateMachine`、`BpfLink` |
| 函数与方法 | `CamelCase` | `SendEncapsulatedCommand()` |
| 局部变量与参数 | `lower_case` | `frame_len`、`max_transfer_size` |
| 私有/保护成员变量 | `lower_case_` 后缀下划线 | `handle_`、`request_id_` |
| 常量（全局/constexpr/枚举值） | `k` + `CamelCase` | `kMaxFrameSize`、`kCacheLineSize` |
| 宏 | `UPPER_CASE`（尽量不用宏） | |
| 文件名 | `snake_case`，实现用 `.cc`，头用 `.h` | `spsc_ring.h`、`bpf_link.cc` |

错误处理分层：

- **初始化 / 控制路径**：`std::expected<T, Error>`，错误可携带 errno / libusb 错误码 / 上下文串。
- **数据热路径**：**绝不**返回 `expected`、绝不抛异常、绝不分配。用返回计数 + 原子统计计数器表达失败。
- 全程 `-fno-exceptions`？**不**关异常（doctest 需要），但项目自身代码不 `throw`。

---

## 5. 实现进度

图例：`✅ 已完成` `🚧 进行中` `⬜ 未开始`

| # | 提交 | 状态 | 备注 |
|---|---|---|---|
| 1 | `chore: 初始化项目脚手架与构建系统` | ✅ | CMake + 规范配置 + vendored doctest + 版本注入 |
| 2 | `feat(common): 基础设施层` | ✅ | 错误/日志/字节序/无锁队列/统计/线程 QoS |
| 3 | `feat(bench): 基准 harness 与微基准` | ✅ | 自带 harness，输出 Markdown 直接进 docs/ |
| 4 | `feat(rndis): RNDIS 协议层` | ✅ | 常量 + 控制消息编解码 + 数据包编解码 |
| 5 | `feat(net): feth 虚拟网卡与 BPF 链路层` | ✅ | 含私有 ABI 声明与 LoopbackLink |
| 6 | `perf(common): 无锁队列批量发布` | ✅ | 1514B 提速 2.4 倍、64B 提速 5.0 倍 |
| 7 | `feat(usb): USB 传输层` | ✅ | 设备发现/声明/控制通道/异步传输池 |
| 8 | `feat(rndis): RNDIS 状态机` + CTest 缺陷修复 | ✅ | **顺带发现测试一直在空跑，见第 7 节第 9 条** |
| 9 | `feat(core): 数据路径桥接` | ✅ | 三线程模型 + 背压 + 统计；修了 TX 统计恒为 0 的缺陷 |
| 10 | `feat(app): 运行时编排与命令行` | ✅ | 启动/停机顺序、信号处理、CLI |
| 11 | `docs: 设计文档、协议参考、性能指南` | ✅ | 本次 |

### 当前状态

- **代码量**：库与应用 9378 行、测试 3886 行、基准 852 行、文档 1577 行
- **测试**：16 个 ctest 用例 / 15 个 doctest test-suite，共 168 个用例、6453 条断言，全部通过
- **构建**：`-Werror` 零告警；ThreadSanitizer 下全绿
- **可运行**：`--version` / `--help` / `--list` 均正常；非 root 启动给出清晰提示
- **已验证**：USB 侧在真实 RNDIS 设备上跑通（枚举、声明接口、RNDIS 握手），
  且 USB 这一侧**不需要 root**；feth 私有 ABI、两个私有 BPF ioctl、
  以及「BPF 写入能让对侧 IP 栈收到帧」这个核心前提都已实测确认。详见第 6 节
- **未验证**：端到端吞吐（至今未做过任何压测）

## 6. 验证清单（需要 root 或真实 RNDIS 设备）

### 6.1 已验证：USB 侧（**无需 root**）

已在一台真实 RNDIS 设备上实测。结论按平台事实记录 —— 它们是 macOS/libusb 的行为，
不依赖具体设备型号。

**① 非 root 下 `libusb_open` 不会返回 `LIBUSB_ERROR_ACCESS`，而是直接成功。**

原先的假设是错的。macOS 打开 USB 设备**不需要 root**（不同于 Linux 的 udev 权限
模型）。整个 USB 侧 —— 枚举、`libusb_open`、声明接口、收发传输 —— 都能以普通用户
运行；root **只**为创建 feth 与打开 `/dev/bpf*` 而需要。`--list` 因此明确标注了
「不需要 root」。

**② ⚠️ `libusb_kernel_driver_active() == 1` 在 macOS 上并不预示 `claim` 会失败。**

这是本节最容易踩错的一条。同一台复合设备上实测：

| 接口签名 | 匹配到的驱动 | `kernel_driver_active` | `libusb_claim_interface` |
|---|---|---|---|
| `02/02/ff` RNDIS 通信 | **无** | 0 | ✅ 成功 |
| `0a/00/00` RNDIS 数据 | `AppleUserECMData`（DriverKit dext） | **1** | ✅ **仍然成功** |
| `02/02/01` CDC-ACM 控制 | `AppleUSBACMControl`（kext） | 1 | ❌ `LIBUSB_ERROR_ACCESS` |
| `03/00/00` HID | `AppleUserUSBHostHIDDevice` | 1 | ❌ `LIBUSB_ERROR_ACCESS` |
| `ff/42/01` 厂商自定义 | 有 | 1 | ❌ `LIBUSB_ERROR_ACCESS` |

`kernel_driver_active` 只反映 IOKit 里有驱动**匹配（matched）**，**不**反映该驱动是否
**独占持有**接口。DriverKit dext（`com.apple.DriverKit.AppleUserECM`）匹配上 `0a/00/00`
数据接口后并不独占，claim 照样成功；而 kext（ACM/HID）是真独占。

**所以：不要拿 `kernel_driver_active` 做预检并提前报错**，那会把本来能用的设备误判成
不可用。唯一可靠的判定是直接 claim 看返回值。又因为 macOS 上没有
`libusb_detach_kernel_driver`（见第 7 节第 7 条），claim 失败就确实无解。

好消息是 RNDIS **通信**接口（`02/02/ff`）压根没有驱动匹配 —— 印证了「macOS 没有
RNDIS 内核驱动」这个立项前提。

复现方法：`--list` 看识别结果；接口级细节用 `ioreg -c IOUSBHostInterface -r -l`
查驱动匹配，再用一段十几行的 libusb 程序逐接口 `claim` 看返回值。

### 6.2 已验证：feth / BPF 侧（**需 root**）

③④⑦ 的复现方法：`sudo TETHERKIT_ROOT_TESTS=1 build/bin/tetherkit_tests --test-suite=net.feth`
（不设该环境变量时这些用例**跳过而非失败**，跳过原因会打印出来），以及直接
`sudo build/bin/tetherkit` 跑一次。⑤⑥ 目前**没有**进测试套件，是用一次性探针测的
（做法写在各条里，⑥ 的缺口已记进 6.3）。

**③ `feth` 配对的私有 ABI 在 macOS 26 上可用。**

`SIOCSDRVSPEC` + `struct if_fake_request` 这条路径完全成立：创建、配对、设 MTU/MAC、
UP、销毁全流程通过。第 2 节里用 `static_assert` 钉死的那些 ioctl 编号和结构体大小
是对的 —— 编号算错的话这里会直接失败。

**④ 两个私有 BPF ioctl 在 macOS 26 上都可用。**

- `BIOCSBATCHWRITE`（一次 `write()` 发多帧）→ 生效，运行日志里显示「批量写 已启用」。
  这坐实了第 7 节第 4b 条的更正：「Darwin BPF 不支持批量写」确实是错的。
- `BIOCSNOTSTAMP`（关每帧时间戳）→ 生效，日志显示「时间戳 已关闭」。

**⑤ `BIOCSBLEN` 的实际上限是 32 MiB，`debug.bpf_maxbufsize` 不是它的约束值。**

按 `BIOCSBLEN` → `BIOCSETIF` → `BIOCGBLEN` 的顺序实测（即挂上接口后读回**有效**值）：

| 请求 | 挂接口后的有效值 |
|---|---|
| 4 KiB / 1 / 4 / 8 / 16 / 32 MiB | 原样生效，一字不改 |
| 64 MiB、2 GiB | 都被钳到 **32 MiB**（`0x2000000`） |

⚠️ 反直觉的一点：`sysctl debug.bpf_maxbufsize` 在本机报的是 **512 KiB**，
比实际能设的小 64 倍 —— **别拿这个 sysctl 去推 `BIOCSBLEN` 的上限**，对不上。
（`debug.bpf_bufsize` 报的 4 KiB 倒确实是默认值。）
另外 32 MiB 只是「设得进去」，不等于「值得设这么大」：默认的 4 MiB 已经够用。

**⑥ BPF `write()` 确实能让对侧 feth 的 IP 栈收到帧 —— 整个方案的核心前提成立。**

用 ARP 往返闭环验证：系统侧 feth 配 `10.99.99.1`，从**驱动侧** feth 的 BPF 写一个
问 `10.99.99.1` 的 ARP 请求，随即在同一个 BPF 上读到了系统侧 IP 栈发回的
**ARP reply**（源 MAC 正是系统侧 feth 的 MAC，宣称拥有该 IP）。

这一个实验同时证明了两件事：

- `write()` 到驱动侧 → 帧穿过 feth 对 → **对侧 IP 栈真的收到并处理了**（否则不会应答）；
- `BIOCSSEESENT=0` 的回环抑制正确 —— 读回来的只有 input 方向的 reply，
  没有我们自己刚写进去的那个 request。

⚠️ 但这条路有个前置条件，见第 7 节第 14 条（`BIOCSHDRCMPLT`）。

**⑦ 端到端：RNDIS 握手 → feth 建对 → BPF 挂载 → 优雅停机，全程跑通。**

一次完整启动的关键节点：声明接口拿到 bulk IN/OUT（`wMaxPacketSize` 512）与中断 IN →
RNDIS 协商完成（版本 1.0、MTU 1500）→ 查到设备 MAC 与链路速率 → 状态机进入
「数据已就绪」→ feth 对创建并配对、系统侧 MAC 改为设备 MAC → BPF 挂上驱动侧。

`SIGTERM` 后拆除顺序正确，**feth 接口无残留，默认路由全程未被改动**。

### 6.3 待验证

- [ ] 端到端吞吐（USB 2.0 high-speed 下能达到多少）。尚未做过任何压测 ——
      至今每次运行 RX/TX 都是 0 pps（没配 IP、没造流量）。
- [ ] `net.feth` 套件目前**没有**覆盖第 6.2 节 ⑦ 的 ARP 往返闭环 ——
      现有用例只断言「读不回自己写的帧」，不能证明对侧真的收到了。
      该闭环值得补成一个 root 用例，否则这条结论只活在本文档里。
- [ ] Android 各版本的 RNDIS quirk（`MaxTransferSize` / `PacketAlignmentFactor` 异常值）。
      → 手头没有 Android 设备，暂时无法验证。

---

## 7. 已知坑与规避（**踩过的坑写在这里，不要重复踩**）

1. **缓存行是 128 字节**，不是习惯性的 64。SPSC 队列若按 64 对齐，生产者/消费者索引仍会
   落在同一缓存行上，false sharing 依旧存在。项目统一用 `kCacheLineSize = 128`。
2. **`std::format` 需要部署目标 ≥ 13.3**，否则报 `to_chars unavailable`。CMake 已强制。
3. **`std::hardware_destructive_interference_size` 在 Apple libc++ 上不存在**，别用。
4. **BPF 没有零拷贝**，别去找 `BIOCSETZBUF`。但**有批量写** —— 见下一条。
4b. ~~「Darwin BPF 不支持批量写」是错的~~。第 2 节最初记录的这条结论已更正：
   macOS 14+ 有私有 ioctl `BIOCSBATCHWRITE`，一次 `write()` 能发多帧。
   本项目已实现并做特性探测（`BpfLink::WriteFramesBatched`），
   macOS 13 及更早自动回落到逐帧 write。
5. **BPF `BIOCSBLEN` 必须在 `BIOCSETIF` 之前设置**，顺序错了缓冲区大小不生效。
6. **BPF 遍历必须用 `bh_hdrlen`**，不能用 `sizeof(struct bpf_hdr)` —— 头部后面有对齐填充。
7. **libusb 在 macOS 上没有 `detach_kernel_driver`**。若接口被内核驱动占用，只能走
   codeless kext / dext 或 re-enumerate，不能靠 libusb 解决。
8. **别默认跑测试时有设备、有 root**，所以「跑不起来」不等于「代码错了」；
   离线可验证的部分必须全部覆盖到 mock 测试里，需要 root 的用例要能干净跳过。
9. **CTest 的 `add_test` 里绝不能给参数加引号。** 写
   `COMMAND tetherkit_tests --test-suite="${_suite}"` 时，CMake 会把引号当作参数
   内容原样传下去（`ctest -V` 可见传的是 `"--test-suite="foo""`），doctest 匹配不到
   任何用例 → 跑 0 个用例 → doctest 对零匹配返回退出码 0 → ctest 报 Passed。
   **整套测试静默失效却全绿**，实际掩盖了 4 个真实失败。
   正确写法是 `--test-suite=${_suite}`（无引号）。
10. **给 ctest 用例加「至少跑到一条断言」的保险时，必须用
   `FAIL_REGULAR_EXPRESSION` 而不是 `PASS_REGULAR_EXPRESSION`。** 后者会
   **取代**退出码判定（CMake 的文档行为），一旦设上，真实的断言失败反而被忽略 ——
   等于把上一条刚修好的坑又挖回来。`FAIL_` 是叠加在退出码之上的。
11. 时间相关的类**不要在构造函数里自己读时钟**。`PeriodicTimer` 原来那样写，
   导致构造函数读到的时刻比调用方手里的 `now` 略晚，「now + period」反而还没到期，
   单元测试随机失败且不可复现。改成显式传入计时起点（默认值保留便利写法）。
12. **⚠️ macOS 上的同步 libusb 中断传输会永久阻塞，timeout 参数根本不被遵守。**
   一旦踩上，症状是控制线程卡死、统计不再输出、连 SIGTERM 都响应不了、
   退出时 feth 接口残留。栈长这样：
   ```
   Poll → WaitForNotification → do_sync_bulk_transfer
        → sync_transfer_wait_for_completion → handle_events → poll(∞)
   ```
   原因链两层（均可在 libusb 源码里核对）：① darwin 后端给所有 transfer 打
   `USBI_TRANSFER_OS_HANDLES_TIMEOUT`，于是 `libusb_get_next_timeout` 跳过它们、
   返回「无超时」，同步等待里的 `poll()` 无限期阻塞；② 而 IOKit 侧 darwin 的
   `submit_interrupt_transfer` 用的是 **`ReadPipeAsync`（无超时变体）**，
   不是 bulk 用的 `ReadPipeAsyncTO` —— 所以中断传输的 timeout **压根不生效**。
   **把 timeout 从 0 改成 1 ms 并不能解决**：第 ② 条决定了它对中断端点自始至终无效。
   唯一正确解法：提交常驻的**异步**中断传输，让它在事件线程上完成，
   `WaitForNotification` 只查一个原子标志，永不进入 libusb 的等待路径。
   → 已如此实现，见 `UsbControlChannel::StartNotificationListener`。
13. 顺带记住：`timeout = 0` 在 libusb/darwin 上表示**无限等待**，不是「不等待」。
   想「只探一下」要用非零的最小值（`kProbeOnlyTimeoutMillis`），
   而且如上所述对中断端点连这个都不管用。
14. **不设 `BIOCSHDRCMPLT = 1` 的话，往 feth 上的 BPF `write()` 直接返回 `ENXIO`
   （"Device not configured"），根本不是「帧头被改写」这么轻。** 做过对照实验：
   同一段代码、同一个接口，唯一差别就是设不设这个 ioctl —— 不设必 `ENXIO`，
   设了 `write` 立刻成功。
   `bpf_link.cc` 里原来的注释只说了 `hdrcmplt=0` 会「剥掉前 14 字节重建帧头」，
   那是 `hdrcmplt` 的**语义**；在 feth 上的**实际后果**是写操作压根不成立
   （`if_fake` 的输出路径处理不了 `AF_UNSPEC` 那条重建分支）。
   排查 `ENXIO` 时很容易误以为是接口没起来或名字写错 —— 先查这个 ioctl。
   顺序上它必须在 `BIOCSETIF` 之前，且批量写（`BIOCSBATCHWRITE`）硬性要求它是 1。

---

## 8. 常用命令

```bash
# 配置 + 构建（警告当错误）
cmake -S . -B build -DTETHERKIT_WARNINGS_AS_ERRORS=ON && cmake --build build -j10

# 跑测试
ctest --test-dir build --output-on-failure

# ThreadSanitizer 验证无锁队列（重要！）
cmake -S . -B build-tsan -DTETHERKIT_ENABLE_TSAN=ON && cmake --build build-tsan -j10 \
  && ctest --test-dir build-tsan --output-on-failure

# 性能基准（务必用未开消毒器的 Release 构建）
cmake -S . -B build-rel -DCMAKE_BUILD_TYPE=Release && cmake --build build-rel -j10 \
  && ./build-rel/bin/tetherkit_bench
```

---
> Source: [XiaoMiku01/TetherKit](https://github.com/XiaoMiku01/TetherKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
