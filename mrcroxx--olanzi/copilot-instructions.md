## olanzi

> > 本文件是**项目级记忆**，每次会话开始时自动加载。修改约定前请先读这里。

# AGENTS.md · 项目 Memory

> 本文件是**项目级记忆**，每次会话开始时自动加载。修改约定前请先读这里。

---

## 1. 语言约定

| 场景 | 语言 |
|---|---|
| 与用户对话 | **中文** |
| 代码注释 | 中文 |
| 提交信息 | **英文**（仓库公开，历史应对英文读者可读） |
| 文档 | **中英双语（强制，见 §2）** |

---

## 2. ⭐ 文档双语强制要求

**`README` 和 `docs/` 下的每一份文档都必须有中英两个版本。**

### 命名规范

⭐ **英文是默认入口，文件名不带语言后缀。** 中文版加 `.zh.md`。

| 英文版（默认，无后缀） | 中文版 |
|---|---|
| `README.md` | `README.zh.md` |
| `docs/01-ulanzi-studio-scope.md` | `docs/01-ulanzi-studio-scope.zh.md` |
| `docs/NN-<ascii-slug>.md` | `docs/NN-<ascii-slug>.zh.md` |

- 文件名**一律用 ASCII slug**（不要用中文文件名）
- 编号 `NN` 中英一致，便于对照
- 两版是**同级目录下的兄弟文件**，不另建子目录
- ⚠️ **不要再新增 `.en.md`** —— 无后缀的那份就是英文版。仓库公开，
  默认入口必须是英文，中文是可选语言

### 硬性规则

1. **新增文档必须同时产出中英两份。** 只写一份视为未完成。英文用无后缀文件名。
2. **修改任一语言版本后，必须同步另一版本。** 不允许长期漂移。
3. **两版的 Markdown 结构必须一一对应**：标题层级、表格列数、代码块、列表层级全部一致。
4. **每份文档头部都要有文档集导航**，中文版链中文、英文版链英文。
5. **中英互链**：英文版头部放 `[中文](....zh.md)`，中文版头部放 `[English](....md)`（指向无后缀的英文版）。
6. **改完必须跑校验**：

   ```bash
   python3 tools/check_docs.py
   ```

   它检查 7 项：中文版是否存在（含孤立的 `.zh.md` 反向检查）、两版结构是否一一对应
   （标题层级 / 代码块数 / 表格形状）、**十六进制数值是否一一保真**、互链是否正常、
   `docs/` 下每份文档是否有 `📚` 导航行、
   **英文版散文是否残留中文**（围栏代码块与行内代码中的中文是合法引用，豁免）、
   相对链接是否全部可解析。**不通过就是没做完。**

### 翻译规范 —— 什么**不**翻译

以下内容**原样保留**，一个字都不要改：

- 十六进制 / 字节序列（`01 06 50 04`、`0xFFF1`、`ca ba a5 ca …`）
- 代码块中的**代码本身**（Python / bash / 汇编指令）
- 命令与参数（`--set-key`、`python3 vibekey.py --keys`）
- 文件路径（`~/ulanzi-re/raw/…`）
- 符号名（`+[MessageHelper setDeviceButtonShortcutFunctionMessage:…]`）
- 错误码（`0xE00002E2`、`kIOReturnNotPermitted`）
- markdown 表格结构与对齐
- emoji 标记（`✅ ⚠️ ⭐ ⌨ ⟳ ❌`）
- 证据强度标签 `[CONFIRMED]` / `[INFERRED]`（本来就是英文）

#### ⚠️ 代码块里的中文分两类，别搞混

| 类型 | 处理 | 例 |
|---|---|---|
| **真实程序输出 / 终端记录** | **一字不改**，中文原样留着（它是证据） | `13:36:28.960 按键 ⌨ 旋钮 按下 PrintScreen 178 ms`、`← 未使用` |
| **说明性记法里的中文占位符** | **翻译** | `<类型\|sign<<7> <键码> ×num` → `<type\|sign<<7> <key code> ×num` |
| **代码注释** | 翻译注释，保留 `;` `#` `//` 等标记与代码本身 | `; 帧头 01 06 50 04` → `; frame header 01 06 50 04` |
| **CLI 参数的元变量** | 翻译 | `--log 文件` → `--log FILE` |

> 判据：**这段中文是"设备/程序真的吐出来的"还是"作者写给人看的说明"？**
> 前者留，后者翻。终端记录是证据，改动它就破坏可审计性。

#### 英文文档里引用程序的中文输出

`vibekey.py` 的输出**就是中文**。英文文档在描述"你会看到什么"时，
**应当用行内代码原样引用中文**，这样读者才能和屏幕上的输出对上：

```markdown
| Category | `按键` (key) / `旋钮` (knob) |
The symptom `✗ 打不开 … 无权限` means the input-monitoring permission is missing.
```

校验器**只检查散文里残留的中文**，行内代码与围栏代码块中的中文一律豁免 ——
所以引用原文不会被判定为漏译。

### 术语表（保持一致）

| 中文 | English |
|---|---|
| 键 / 按键 | key |
| 旋钮 | knob |
| 旋钮按下 | knob press |
| 旋钮 → 右拧 / ← 左拧 | knob twist → right / ← left |
| 控件 | control |
| 厂商通道 | vendor channel |
| 输入接口 | input interface |
| 帧 | frame |
| 明文 / 密文 | plaintext / ciphertext |
| 保活 | keepalive |
| 心跳 | heartbeat |
| 按键配置 / 可编程按键表 | key configuration / programmable key table |
| 设备端 | on-device / device-side |
| 逆向 | reverse engineering |
| 抓包 | capture |
| 反汇编 | disassembly |
| 符号表 | symbol table |
| 验证记录 | verification log |
| 职责边界 | scope / responsibility boundary |
| 踩坑 | pitfalls |
| 实测 | measured / verified |
| 待验证 | to be verified |
| 固件 | firmware |
| 电量 | battery |
| 降噪 | noise reduction |
| 指示灯 | indicator light |
| 组合键 | key combination |
| 修饰键 | modifier key |
| 时延 / 时长 | duration |

### 文档集

| # | English（默认） | 中文 | 内容 |
|---|---|---|---|
| — | `README.md` | `README.zh.md` | 项目总览、快速上手 |
| 01 | `docs/01-ulanzi-studio-scope.md` | `.zh.md` | Studio 职责边界 |
| 02 | `docs/02-vibekey-protocol.md` | `.zh.md` | 协议：TEA / 帧格式 / 命令表 / 按键表 |
| 03 | `docs/03-tool-manual.md` | `.zh.md` | `vibekey.py` 工具手册 |
| 04 | `docs/04-methodology.md` | `.zh.md` | 逆向方法论 |
| 05 | `docs/05-verification-log.md` | `.zh.md` | 验证记录 |

---

## 3. 项目地图

```
olanzi/
├── AGENTS.md                      ← 本文件（项目记忆，不翻译）
├── README.md / README.zh.md       ← 入口（英 / 中）
├── vibekey.py                     ← 工具本体（单文件，零第三方依赖）
├── tools/
│   └── check_docs.py              ← 文档双语一致性校验
└── docs/
    ├── 01-ulanzi-studio-scope.md  (+ .zh.md)
    ├── 02-vibekey-protocol.md     (+ .zh.md)
    ├── 03-tool-manual.md          (+ .zh.md)
    ├── 04-methodology.md          (+ .zh.md)
    ├── 05-verification-log.md     (+ .zh.md)
    └── evidence/                  ← 原始证据留档（不翻译）
```

**逆向中间产物在 `~/ulanzi-re/`**（反汇编、符号表、62 MB 解码日志），
属于临时工作区，**不入库、不翻译**。

---

## 4. 关键技术不变量

> 新会话不需要重新推导这些，直接用。完整细节见 `docs/02-vibekey-protocol.zh.md`。

### 设备

- **AU05**（Vibe Key），VID `0xFFF1` / PID `0x00DD`，序列号 `202606031150`
- ⚠️ **设备唯一标识不公开**：`deviceSn` 与 `flashId` 完整值一律写成 `<REDACTED>`，
  `docs/evidence/` 日志里用 `xx` 掩码（保持字节数与偏移）。序列号保留作示例。
- 控件：**3 个键（上下排列）+ 1 个旋钮 + 1 个电源键**
- 接口 2 = 标准 HID（Consumer `0x01` / Mouse `0x02` / Keyboard `0x03`）
- 接口 3 = 厂商私有（Usage Page `0xFFFC`，Report ID `0x55`）
- **电源键不发报文**（设备硬件处理）

### 加密

- **TEA / ECB / 8 字节分组 / 32 轮**（不是 XTEA）
- delta `0x9E3779B9`，解密 sum 起始 `0xC6EF3720`
- 密钥 `ca ba a5 ca 6d 8a 2a bc ba 9e 5a ca ca 8b b8 9b`
- 验证：`TEA_Enc(00×8) == 38 90 c4 99 a3 60 aa ad`

### 帧

- **明文 64 字节，只发前 63 字节**（第 64 字节位置被 report ID 占用）
- **解密只解 7 个分组（56 字节）**，末 7 字节是不可解填充
- 帧头 `cmd = frame[0] & 0x1F`；回复标记 `frame[0] & 0x80` 或 `frame[3] & 0x10`

### 控件映射（出厂值）

| 控件 | index | 键码 |
|---|---|---|
| 键 1（上） | 0 | `0x01` ErrorRollOver（**无效码**） |
| 键 2（中） | 1 | `0x28` Enter |
| 键 3（下） | 2 | `0x29` Esc |
| 旋钮 按下 | 3 | `0x46` PrintScreen |
| 旋钮 → 右拧 | 4 | `0x4F` RightArrow |
| 旋钮 ← 左拧 | 5 | `0x2A` Backspace |

### 按键配置读写

```
读:  → 01 06 50 01 <index>
     ← 81 06 50 11 <index> 01 <num> <类型|sign<<7> <键码> ×num
写:  → 01 06 50 04 <index> 01 <num> <类型|sign<<7> <键码> ×num
     ← 81 06 50 14 …      (access 0x14 = 写确认)
```

- `类型 = 0x02` = 普通按键（`0x03` = 系统/多媒体）
- **写入持久化在设备内**
- `index 6/7` 存在但未使用

### 区分旋钮与按键

- **旋钮转动 = 瞬时脉冲 0–10 ms**
- **人手按键 = 80–2000 ms**
- 阈值取 **25 ms**（`vibekey.py` 的 `INSTANT_MS`）

---

## 5. 硬性技术约束（踩过的坑，别再踩）

| 约束 | 说明 |
|---|---|
| **不要用 `IOHIDManagerOpen`** | 全有全无，一个设备失败整个调用失败。必须 `IOServiceGetMatchingServices` + 逐个 `IOHIDDeviceOpen` |
| **输入监控权限失败要大声报错** | `0xE00002E2` 时设备照样注入按键但一条报文都收不到，静默跳过会导致"按了没反应"却查不出原因 |
| **`0xE00002C5` 常是自己造成的** | 多半是自己的抓包进程占着设备，不是权限问题 |
| **xlog 解压用 `zlib.decompressobj(-15)`** | raw deflate，不是 `15`/`31` |
| **日志里的 JSON 是转义的** | 正则要写 `\\"index\\"` 而不是 `"index"` |
| **macOS 没有 `timeout` 命令** | 用 Python 的 socket 超时或自己 sleep |
| **别用 `grep` 搜 62 MB 日志** | `maximum repetition exceeds 255`，用 Python `re` |
| **精确定位函数用 lldb** | `lldb -o "disassemble -n '<symbol>'"`；LLVM objdump 对 Mach-O 地址区间支持有问题 |
| **按键会注入焦点窗口** | 工具默认 `stty -echo`，并同时写日志文件 |
| **ctypes 回调里的异常会被静默吞掉** | 只打一行 `Exception ignored` 就继续跑。症状**和"设备关机"一模一样** —— 曾据此做过一次错误诊断。改 `Monitor` 时务必确认回调里没有会抛异常的表达（尤其 `@staticmethod` 里别写 `self.`） |
| **判断设备是否离线，看原始明文而不是"有没有回复"** | 真离线：`06 03 0a 11` **`00`**（status=0x00），且只有 `cmd=0x06` 会回；代码 bug：**一行都不打印**。两者必须分清 |

---

## 6. 安全规则

| 操作 | 风险 |
|---|---|
| `--probe` / `--keys` / `--list` / `--descriptor` / `--poll` | **只读，安全** |
| `--set-key` | **写设备配置**。改动持久化但可还原；试验时**优先选"改坏也没关系"的目标**（例如出厂就是无效码的键 1） |
| `access=0x04` 的其他写命令 | **不要乱发** —— 命令表里有几十条写命令（亮度/麦克风/指示灯/OTA），参数格式未明前可能让设备进入异常状态 |
| 电源键长按 | **可能直接关机**，不要测 |

**应急恢复（回到出厂值）**：

```bash
python3 vibekey.py --set-key 0=0x01 --set-key 1=0x28 --set-key 2=0x29 \
                   --set-key 3=0x46 --set-key 4=0x4F --set-key 5=0x2A
```

---

## 7. 证据标准

写文档时沿用第一期的标注约定：

- **[CONFIRMED]** —— 有直接证据：实测数据、二进制字面量、或设备回复
- **[INFERRED]** —— 由间接证据推断，**待验证**

**永远不要把自己的推断写成结论。**
（第一期把 `route: "device-direct"` 当结论写下来，后来发现是死代码，教训。）

**失败尝试也要留档**，见 `docs/05-verification-log.zh.md` —— 这比成功经验更省时间。

---

## 8. 协作偏好

- 用户愿意**动手验证**（按键、改配置、授权）。需要实测时**直接说明要按哪个控件、什么顺序**。
- 需要用户配合时，给出**严格有序、带间隔**的操作步骤，这样才好做时间对齐。
- 用户的现场观察**优先级高于推断** —— "只有三个键"这条信息曾一次性纠正了错误模型。
- 长任务优先并行（`workflow` / `subagent`），不要把串行等待堆在主上下文里。

---

## 9. Rules (English)

> This section mirrors the binding rules above for English-speaking contributors.

- **Conversation and code comments are in Chinese; commit messages are in English**
  (the repository is public, so its history should be readable to English-speaking contributors).
- **English is the repository's default language**: entry-point filenames carry no language
  suffix. Chinese is an opt-in alternative suffixed with `.zh.md`.
- **All documentation under `README` and `docs/` MUST exist in both Chinese and English.**
  - English (default): `README.md`, `docs/NN-<ascii-slug>.md`
  - Chinese: `README.zh.md`, `docs/NN-<ascii-slug>.zh.md`
  - Never add new `.en.md` files — the unsuffixed file *is* the English version.
  - Filenames are always ASCII slugs — never Chinese filenames.
  - Both versions must share identical Markdown structure (headings, tables, code blocks).
  - Each version's header links to its counterpart: `[中文](….zh.md)` in English,
    `[English](….md)` in Chinese.
  - Editing one language requires updating the other.
- **Never translate** hex/byte sequences, code blocks, commands, file paths, symbol names,
  error codes, table structure, emoji markers, or the `[CONFIRMED]`/`[INFERRED]` tags.
- Use the glossary in §2 for terminology consistency.
- **Quoting real program output:** `vibekey.py` prints Chinese, so the English docs SHOULD
  quote Chinese literals in inline code (`` `按键` ``, `` `✗ 打不开 … 无权限` ``) so readers can
  match what's on screen. The checker only flags Chinese left in **prose**; inline code and
  fenced blocks are exempt.
- `~/ulanzi-re/` holds throwaway reverse-engineering artifacts — not committed, not translated.
- **Device-unique identifiers are not published**: always write the full `deviceSn` and `flashId`
  as `<REDACTED>`, and mask them with `xx` in `docs/evidence/` logs (preserving byte count and
  offsets). The serial number is kept as a worked example.
- **Exceptions raised inside a ctypes callback are swallowed silently.** ctypes prints one
  `Exception ignored` line and keeps running, so the symptom is **indistinguishable from a
  powered-off device** — this once produced a wrong diagnosis. When editing `Monitor`, make sure
  no expression in a callback can raise (especially: never reference `self.` inside a
  `@staticmethod`).
- **Decide "is the device offline?" from the raw plaintext, not from "did a reply arrive".**
  Genuinely offline: `06 03 0a 11` **`00`** (status=0x00) and only `cmd=0x06` answers.
  A code bug: **nothing prints at all**. Never confuse the two.
- Follow the technical constraints in §5 and the safety rules in §6.

---
> Source: [MrCroxx/olanzi](https://github.com/MrCroxx/olanzi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
