## duckvm

> 基于文本匹配的 VMP Handler 还原工具。通过解析 ARM64 trace 日志，利用 VMP dispatch 模式（`LDR [X24,...] + BR`）自动识别 handler 入口，遇到未知 handler 停下让人工命名，逐步还原完整 handler 序列。

# DuckReVM

基于文本匹配的 VMP Handler 还原工具。通过解析 ARM64 trace 日志，利用 VMP dispatch 模式（`LDR [X24,...] + BR`）自动识别 handler 入口，遇到未知 handler 停下让人工命名，逐步还原完整 handler 序列。

**核心原则**：
- 这个方案不考虑 B 指令，一份 trace 就是一条执行流，也就是一份算法，不再有其他情况
- 这是**解释器**不是文本解析器 — 解释 VM 指令语义，不要硬匹配 ARM64
- 每条指令位置是固定的，变了含义就不一样
- 必须理解 handler 本质，然后去读实际 trace 数据

## handler 格式

几乎都可以套用

```text
.text:00000000002B05F4 vm_handler_STORE_REG                    ; DATA XREF: .data.rel.ro:00000000003680E0↓o
.text:00000000002B05F4                 LDR             W8, [X0] ; vm_op_store: *(regs[base] + (int16_t)offset) = regs[value], PC += 1
.text:00000000002B05F8                 MOV             W13, #0x30 ; '0' ; W13 = 48 (每条指令元数据大小，用于计算下一条指令地址)
.text:00000000002B05FC                 LDR             W9, [X19] ; W9 = PC
.text:00000000002B0600                 LDR             X12, [X20] ; X12 = *instruction_stream = 指令元数据基址
.text:00000000002B0604                 AND             X10, X8, #0xFF ; X10 = BYTE0(op) = base寄存器索引
.text:00000000002B0608                 UBFX            X11, X8, #8, #8 ; X11 = BYTE1(op) = value寄存器索引
.text:00000000002B060C                 ADD             W9, W9, #1 ; W9 = PC + 1
.text:00000000002B0610                 SBFX            X8, X8, #0x10, #0x10 ; X8 = sign_extend(HIWORD(op)) = 有符号内存偏移
.text:00000000002B0614                 LDR             X10, [X23,X10,LSL#3] ; X10 = regs[base] (基址值)
.text:00000000002B0618                 NOP
.text:00000000002B061C                 SMADDL          X0, W9, W13, X12 ; X0 = (PC+1)*48 + metadata_base = 下一条指令元数据地址
.text:00000000002B0620                 LDR             X11, [X23,X11,LSL#3] ; X11 = regs[value] (要写入的值)
.text:00000000002B0624                 STR             W9, [X19] ; PC = PC + 1
.text:00000000002B0628                 STR             X11, [X8,X10] ; >>> 核心操作: *(regs[base] + offset) = regs[value] (STORE)
.text:00000000002B062C                 LDR             X8, [X0,#0x28] ; X8 = next_insn.metadata[0x28] = 下一个opcode索引
.text:00000000002B0630                 LDR             X8, [X24,X8,LSL#3] ; X8 = opcode_handler_table[next_opcode]
.text:00000000002B0634                 BR              X8      ; BR → 跳转下一个handler
```

## 已知陷阱

### 长度检查必须扣掉 end_addr

`interpret_handler()` 第 1084 行过滤掉了 `offset == cfg.end_addr` 的指令（即 BR），所以 `handler_insts.size()` 比 trace 中看到的指令数**少 1**。

简单 handler 的 `insts.size()` 通常在 9~15，长度检查用 `< 8` 即可。之前多次因为设 `< 16` 导致返回空串、handler 不出现在输出中。

### PAC/加密指针不是 bug

VMP 代码中常见高位非零的"巨值"指针（如 `0x9383fb018a0c23c2`）。这是 ARM64 PAC (Pointer Authentication) 的认证码或者 VMP 自己的指针加密，不是工具 bug。IR 注释应如实反映 trace 中的值。

### RegAccess 字段名

`RegAccess` 的寄存器名字段是 `reg_name`，不是 `reg`。

### STORE32 的 offset 是无符号提取

某些 compound handler 中的 STORE32 用 `lsl #0x20 + and 0xFFFF000000000000` 提取 offset（无符号），与其他 handler 的 `sign_extend_16` 不同。写解释器时需看 IDA 确认。


## 构建

```bash
./build.sh
# 或手动:
mkdir -p build && cd build && cmake .. -DCMAKE_BUILD_TYPE=Release && make -j$(nproc)
```

产物: `./build/duckrevm`

## 使用

**基础模式**（还原 handler 序列）：
```bash
./build/duckrevm trace_logs/code.log
```

工作流: 运行 → 遇到未知 handler 停下 → 去 IDA 分析该地址 → 添加到 `trace_logs/handlers_config.json` → 重新运行，重复直到还原完毕。

## 架构

- `src/trace_parser.cpp` - 解析 trace 文件，格式: `index : 0xaddr [0xoffset] "mnemonic operands" (registers)`
  - 自动提取寄存器读写信息到 `TraceInstruction.reads` 和 `TraceInstruction.writes`
- `src/handler_matcher.cpp` - 核心匹配引擎，识别 dispatch 模式、BL 折叠、嵌套 VMP 处理
- `src/output_writer.cpp` - 输出结果
- `src/handler_interpreter.cpp` - 具体的 handler 解释器插件
- `trace_logs/handlers_config.json` - handler 地址配置（含 start/end addr）

## 关键设计

- 使用 **offset 地址**匹配（非绝对地址），兼容不同基址
- **BL 折叠**: 遇到 BL 调用时用栈跟踪返回地址，折叠 native 函数内部指令，但保留嵌套 VMP handler
- **Dispatch 检测**: 前一条是 BR + 往前 5 条内有 `LDR Xn, [X24,...]` → 判定为 handler 入口
- **500 条指令上限**: 防止匹配跑飞，超限截断并警告
- **寄存器读写解析**: 解析时一次性提取所有 `(r)reg=value` 和 `(w)reg=value` 信息
- C++11，无第三方依赖，流式解析

## handlers_config.json 格式

```json
{
  "ctx": { "pc_reg": "x19", "ctx_base": "x23" },
  "handlers": [
    { "addr": "0x2c0cc0", "end_addr": "0x2c0d50", "name": "COMPOUND_STORE_INIT" }
  ]
}
```

`end_addr` 指定后精确匹配结束位置；未指定则扫描到 BR 指令结束。

## 解释器 (IR 生成)

接下来任务就是根据工具产出的未识别 handler 继续还原。

使用 IDA MCP 和 trace 日志进行，下面是完整工作流：

### 1. 运行工具找未识别 handler
```bash
./build/duckrevm trace_logs/code.log
```
工具会停在第一个未识别 handler 并输出：
```json
{"index": 10f93, "offset": 0x2b137c, "mnemonic":"ldr w8, [x19]"}
```

### 2. 提取 trace 指令序列
```bash
grep "0x2b137c" trace_logs/code.log -A 20
```
找到完整 handler 执行序列，识别 dispatch 边界（最后的 `br` 指令）。

### 3. 用 IDA MCP 分析语义
```python
mcp__ida-pro-mcp__disasm(addr="0x2b137c", max_instructions=20)
```
- 看 IDA 注释理解 handler 做了什么（如 `LOAD_INDIRECT`、`MOV_REG`）
- 关注关键指令：操作数读取、数据计算、结果写入
- 识别编码格式（BYTE0/BYTE1/HIWORD）

### 4. 写 JSON 配置
添加到 `trace_logs/handlers_config.json`：
```json
{
  "addr": "0x2b137c",
  "end_addr": "0x2b13bc",
  "name": "LOAD_INDIRECT"
}
```

### 5. 实现解释器函数
在 `src/handler_interpreter.cpp` 里实现：

**标准模板**：
```cpp
static std::string interpret_xxx(const std::vector<TraceInstruction> &insts)
{
    // 1. 长度检查
    if (insts.size() < N)
        return "";
    
    // 2. 从固定位置读取操作数（通常是 insts[3]）
    if (insts[3].writes.empty())
        return "";
    uint64_t op = insts[3].writes[0].value;
    
    // 3. 解码操作数字段
    int dst_reg = (int)((op >> 8) & 0xFF);  // BYTE1
    int64_t offset = sign_extend_16(op >> 16);  // HIWORD
    
    // 4. 提取运行时实际值（用于标注）
    uint64_t actual_value = 0;
    if (insts.size() > M && !insts[M].writes.empty())
        actual_value = insts[M].writes[0].value;
    
    // 5. 生成 IR，末尾标注实际值
    std::ostringstream ir;
    ir << "load VM_REG[" << std::dec << dst_reg << "], [ 0x" << std::hex << base
       << " + 0x" << offset << " ]  // = 0x" << actual_value;
    return ir.str();
}
```

### 6. Handler 解释器编写规范

#### call 类

**纯 CALL handler**：整个 handler 就是一次调用，所有指令输出原始 trace。
为了不一直看过多的Trace日志，我先把call的输出函数改掉了，仅仅输出一句话，我后面有思路

```cpp
static std::string interpret_call(const std::vector<TraceInstruction> &insts)
{
    return output_call_raw_trace(insts, 0, insts.size() - 1);
}
```

**Compound call handler**（如 `COMPOUND_STORE_CALL_LOAD`、`COMPOUND_MOV_CALL`）：
handler 包含 CALL + 其他 VM 操作。只有 CALL 部分（BL → return）输出原始 trace，
pre-call 和 post-call 的操作（STORE/LOAD/MOV）必须解释为正常 IR。

结构：`pre-call 指令 ... → BL → native dispatch 内部指令 → return(offset=BL_off+4) → post-call 指令 ... → BR`

实现模式：
```cpp
static std::string interpret_compound_xxx_call_yyy(const std::vector<TraceInstruction> &insts)
{
    // 1. 长度检查
    if (insts.size() < N) return "";

    auto at = [&](uint32_t off) -> const TraceInstruction * { ... };

    // 2. 找 BL 指令位置
    size_t bl_pos = 0; uint32_t bl_off = 0;
    for (size_t k = 0; k < insts.size(); k++) {
        if (insts[k].mnemonic == "bl" || insts[k].mnemonic == "BL") {
            bl_pos = k; bl_off = insts[k].offset; break;
        }
    }

    // 3. 找 return（BL_off + 4）
    uint32_t ret_off = bl_off + 4; size_t ret_pos = 0;
    for (size_t k = bl_pos + 1; k < insts.size(); k++) {
        if (insts[k].offset == ret_off) { ret_pos = k; break; }
    }

    std::ostringstream ir;

    // 4. Pre-call: 解释为正常 IR（用 at() 按 offset 查操作数）
    ir << "store VM_REG[...], ...  // = 0x...\n";
    ir << "---\n";

    // 5. Call: 原始 trace（含 BL + native 内部指令 + return）
    ir << output_call_raw_trace(insts, bl_pos, ret_pos);
    ir << "---\n";

    // 6. Post-call: 解释为正常 IR
    ir << "load VM_REG[...], ...  // = 0x...\n";

    return ir.str();
}
```

**辅助函数** `output_call_raw_trace`：输出 `[start, end]` 范围的指令为原始 trace 格式。
定义在 `interpret_call` 上方，所有 call 类 handler 共用。

**BL 折叠**：BL 调用 native 函数时，内部指令 offset 在 handler 的 `[start, end]` 范围之外。
`interpret_handler` 过滤条件只有 `!= cfg.end_addr`，所以 native 内部指令会包含在 handler_insts 中。
Compound call 函数的 pre/post 段通过 `at()` 按 handler 范围内的 offset 查找不会被干扰。

#### BL 专属型 compound handler

识别特征：handler 入口是 `mov x1, x19`，紧接着 `bl sub_XXXXXXXX`，返回后直接标准 dispatch。

```asm
0x2c43a4: MOV X1, X19          ; 传入 PC 寄存器
0x2c43a8: BL sub_2CE014         ; 调用复合函数
0x2c43ac: LDR W8, [X19]        ; ← 标准 dispatch 开始
0x2c43b0: MOV W10, #0x30
0x2c43b4: LDR X9, [X20]
0x2c43bc: SMADDL X0, W8, W10, X9
0x2c43c0: LDR X8, [X0,#0x28]
0x2c43c4: LDR X8, [X24,X8,LSL#3]
0x2c43c8: BR X8
```

**特点**：
- handler 本身只有 `MOV X1, X19` + `BL` + 标准 dispatch，不做任何 VM 操作
- 所有 VM 操作都在被调函数（`sub_XXXXXXXX`）内部完成
- PC 推进也发生在被调函数内部（handler 返回后 PC 已更新）
- 被调函数通常含 opaque predicate 混淆

**分析步骤**：
1. 看到 `mov x1, x19; bl ...` 入口，立刻去 IDA 打开被调函数
2. 重点分析被调函数：从 `[X0]`、`[X0,#8]`、`[X0,#0x10]` 等处读取操作码
3. 追踪每条操作码的 decode（AND/UBFX/SBFX）和执行（LDR/STR/ORR 等）
4. `at()` 查找 trace 偏移要用**被调函数的 offset**（0x2ceXXX 等），不是 handler 的 offset
5. 最后 PC 写入 `STR Wx, [X1]` 找 PC 最终值，确定 PC += N

**示例**：
| handler | 被调函数 | 操作 | PC |
|---------|---------|------|-----|
| `0x2c43a4` | `sub_2CE014` | LOAD + LDRSW + LOAD (从 [X0+0x10],[X0+8],[X0]) | +4 |
| `0x2c3610` | `sub_2CD3A4` | (待分析) | ? |

**代码模板**：
```cpp
// COMPOUND_XXX: BL 专属型，调用 sub_YYYYYY 执行 N 条 VM 操作
// Op1 (从 [X0+0x10]): ...
// Op2 (从 [X0+0x08]): ...
// Op3 (从 [X0]):      ...
// 关键 trace 偏移(来自 sub_YYYYYY):
// - 0xYYYYYY: op1 操作码
// - 0xYYYYYY: op1 结果
static std::string interpret_compound_xxx(const std::vector<TraceInstruction> &insts)
{
    if (insts.size() < 50) return "";  // 被调函数通常较长
    auto at = [&](uint32_t off) -> const TraceInstruction * { ... };
    std::ostringstream ir;
    // 按被调函数的 offset 查找操作码和结果
    auto *op1 = at(0x2ce028);  // 用被调函数内的 offset
    ...
}
```

注意：字段名是 `reg_name` 不是 `reg`。TraceInstruction 结构体无 `reg` 成员。

#### b 类
遇见bxx handler直接把b部分跳过

#### Hex 格式

| 场景 | 格式 | 示例 |
|------|------|------|
| 64 位地址/指针值 | 固定 16 位 hex，`setfill('0')` + `setw(16)` | `0x000000007376629da0` |
| 64 位通用值（实际数据） | 不补零，自然宽度 | `0x7376629da0` |
| 32 位值 | 自然宽度 | `0x4a`, `0x90019` |
| 小整数（寄存器索引、偏移量） | 十进制 `std::dec` | `VM_REG[9]`, `+ 0x138` |


```cpp
// 64 位地址 — 固定 16 位 hex
ir << "0x" << std::hex << std::setfill('0') << std::setw(16) << addr;

// 64 位数据值 — 自然宽度
ir << "0x" << std::hex << value;
```

**规则**：地址/指针类输出固定 16 位便于对齐比较；普通数据值不补零避免冗余。每次 `<< std::hex` 之后如果需要输出十进制，必须 `<< std::dec` 恢复。

#### IR 注释

```cpp
// 加载/存储类 — 用 // = 标注实际值
ir << "load VM_REG[" << std::dec << dst << "], [ ... ]  // = 0x" << std::hex << actual_value;

// 计算类 — 用 // = 标注结果
ir << "add VM_REG[" << std::dec << dst << "], ...  // = 0x" << std::hex << result;

// MOV 类 — 同上
ir << "mov VM_REG[" << std::dec << dst << "], VM_REG[" << src
   << "]  // = 0x" << std::hex << actual_value;
```

注释格式统一为 `  // = 0x...`（两个空格 + `//` + 一个空格 + `=` + 一个空格），紧跟 IR 语句。
handler IR之间以横线分割，区分出来每个handler做了什么

#### IR 语法

```
mov     VM_REG[dst], VM_REG[src]
mov     VM_REG[dst], 0x{imm}
mov_hi  VM_REG[dst], 0x{result}
add     VM_REG[dst], VM_REG[base] + 0x{offset}
load    VM_REG[dst], [ VM_REG[base] + 0x{offset} ]
load    VM_REG[dst], [ 0x{base_addr} + 0x{offset} ]
load_sw VM_REG[dst], [ VM_REG[base] + 0x{offset} ]
load_imm VM_REG[dst], [ 0x{base_addr} + 0x{offset} ]
store   VM_REG[val], [ VM_REG[base] + 0x{offset} ]
store32 VM_REG[val], [ VM_REG[base] + 0x{offset} ]
```

多步 handler 每步一行，用 `\n` 分隔。

#### 查找指令

**优先用 offset 查找**（`at()` lambda），不要用硬编码索引：

```cpp
auto at = [&](uint32_t off) -> const TraceInstruction * {
    for (auto &i : insts)
        if (i.offset == off)
            return &i;
    return nullptr;
};

auto *op = at(0x2b92f0);   // ✅ 用指令在 handler 内的 offset
if (!op || op->writes.empty())
    return "";
```

**例外**：简单 handler（≤16 条指令、结构固定不变）可直接用 `insts[N]` 索引——仅在确定 handler 入口指令就是 `insts[0]` 的场合。

#### 代码模板

```cpp
// {HANDLER_NAME}: 一句话描述语义
// 编码: BYTE0=xxx, BYTE1=xxx, HIWORD=xxx
// 关键指令位置(按 offset 查找):
// - 0x{off1}: {指令} → {含义}
// - 0x{off2}: {指令} → {含义}
static std::string interpret_xxx(const std::vector<TraceInstruction> &insts)
{
    // 1. 长度检查
    if (insts.size() < N)
        return "";

    auto at = [&](uint32_t off) -> const TraceInstruction * {
        for (auto &i : insts)
            if (i.offset == off)
                return &i;
        return nullptr;
    };

    // 2. 读取操作数
    auto *op_inst = at(0x{off});
    if (!op_inst || op_inst->writes.empty())
        return "";
    uint64_t op = op_inst->writes[0].value;

    // 3. 解码
    int field1 = (int)(op & 0xFF);           // BYTE0
    int field2 = (int)((op >> 8) & 0xFF);    // BYTE1
    int field3 = (int)((op >> 16) & 0xFF);   // BYTE2
    int64_t imm = sign_extend_16(op >> 16);  // HIWORD

    // 4. 提取运行时实际值
    uint64_t actual = 0;
    if (auto *v = at(0x{off}))
        if (!v->writes.empty())
            actual = v->writes[0].value;

    // 5. 生成 IR
    std::ostringstream ir;
    ir << "...";
    return ir.str();
}
```

#### 变量命名

- 操作数字段: `src_reg`, `dst_reg`, `base_reg`, `val_reg`（语义化）
- 偏移量: `offset`, `off1`, `off2`
- 运行时值: `actual_value`, `loaded_val`, `base_val`, `result`
- 函数索引: `call_idx`
- 不要用 `op1`/`op2` 当变量名——用语义化的如 `mov_op`/`call_idx`

### 7. 注册到 dispatch_map
```cpp
static const std::unordered_map<std::string, HandlerFunc> dispatch_map = {
    { "LOAD_INDIRECT", interpret_load_indirect },
    { "MOV_REG", interpret_mov_reg },
};
```

### 8. 编译验证
```bash
cd build && make -j$(nproc)
./build/duckrevm trace_logs/code.log
tail -20 build/handlers.txt
```

**示例输出**：
```
load VM_REG[5], [ 0x73d6638a10 + 0x138 ]  // = 0x73d6638b48
mov VM_REG[27], VM_REG[12]
```


### Handler 类型说明

- **simple**: 单步 handler，PC += 1，指令数 10~16 条
- **compound**: 复合 handler，PC += N（执行多条 VM 指令），通常 20~40 条指令
- **call**: 调用类 handler，BL 跳转到 vm_call_dispatch，输出原始 trace 格式

### 编码模式速查

| 指令 | BYTE0 | BYTE1 | HIWORD | 注 |
|------|-------|-------|--------|-----|
| MOV_IMM | unused | dst_reg | imm16(s) | |
| MOV_HI | unused | dst_reg | imm16(s) | LDRSW 加载 + AND 0xFFFFFF..0000 清除低16 |
| LOAD_MEM | src_reg | dst_reg | offset(s16) | LDR 64-bit |
| LOAD_SW | src_reg | dst_reg | offset(s16) | LDRSW 32-bit 符号扩展 |
| STORE_REG | base_reg | val_reg | offset(s16) | STR 64-bit |
| CALL | call_idx | — | — | 直接读取 [x0] 作为调用索引 |

### 混淆处理

部分 handler（如 COMPOUND_LOADSW_STORE32_LOADMEM）内含 opaque predicate 混淆（固定跳转 + 虚假分支）。Trace 执行路径是确定的，按实际 trace 偏移提取值即可，不需要理解混淆逻辑。

---
> Source: [djskncxm/DuckVM](https://github.com/djskncxm/DuckVM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
