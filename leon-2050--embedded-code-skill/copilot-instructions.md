## embedded-code-skill

> 嵌入式 C 的 REWRITE/REVIEW/GUIDE：三层整理、低层审查、RTOS/构建咨询。硬件参数须有出处。


# Embedded C 代码助手 Skill

---

## 1. 定位与使用原则

### 1.1 定位

帮助处理嵌入式 C 代码：旧代码整理（**REWRITE**）、低层固件审查（**REVIEW**）、RTOS/构建/调试指导。

本 skill 提供不绑定 IDE/agent 的保守编码规范。寄存器偏移、位定义、IRQ、屏障、cache/DMA、时序等须来自手册、厂商头文件或仓库——**不编造硬件事实**。

### 1.2 使用原则

1. 先判 `REWRITE`/`REVIEW`/`GUIDE`（§1.4）→ 读仓库头文件、命名、SDK、已有驱动样例
2. 遵守 §4 三层架构（reg → drv → app；ISR 回调通知 app）
3. 项目规范优先；CMSIS/厂商结构体直接复用，不为 fallback 再包一层
4. 硬件信息缺口先列出；待定值标 placeholder
5. 输出优先 patch；风格让位于 correctness/并发/硬件风险

### 1.3 前置信息

| 信息 | 要求 | 示例 |
|------|------|------|
| 外设/模块名 | REWRITE 必需 | `uart`, `spi`, `gpio`, `dma` |
| 硬件来源 | 强烈建议 | 参考手册章节、厂商头文件、现有驱动 |
| 芯片或架构 | 强烈建议 | `STM32F4`, `Cortex-M4`, `ESP32` |
| 基地址/位定义 | 生产代码必需 | `UART_BASE_ADDR = 0x4000C000U` |
| 项目约定 | 重写前读取 | status type、命名、SDK、build macros |
| RTOS（如适用） | 驱动层需确认 | FreeRTOS、Zephyr、RT-Thread、裸机 |

缺口先列出；待定值标 `USER_PROVIDED` / `REPO_DERIVED` / `PLACEHOLDER`。

### 1.4 工作模式

**REWRITE**：保留 public API、ABI、寄存器写入顺序与时序序列；按 §4 整理类型/命名/分层。输出：简述 → 缺口 → patch（必要时文件布局）。workaround 标 `/* 有意保留：原因 */`。

**REVIEW**：不产出代码。按寄存器抽象 → 分层/ISR/同步 → volatile/barrier/cache/DMA → 错误处理/内存 顺序查。输出表：`| P0/P1/P2 | 位置 | 问题 | 建议 |`（P0 行为/安全，P1 并发/可移植，P2 风格）。

**GUIDE**：无代码整理/审查需求（RTOS 任务设计、CMake/链接脚本、调试策略等）。按 §5–§9 给建议或示例片段，不走 REVIEW 表。

### 1.5 RED LINES

禁伪造硬件参数；禁低层 `malloc`/VLA；禁公共接口用 `int`/`char`/`long`；禁业务代码裸寄存器地址；禁不可编译输出；**禁违反三层解耦（reg/drv/app 分层，ISR 回调通知 app）**。

---

## 2. Fallback 编码规范

仓库无更强约定时适用；符合则沿用，不符合则不改逻辑地统一。

### 2.1 命名

| 元素 | 规范 | 示例 |
|------|------|------|
| 变量 | `snake_case` | `rx_count` |
| 全局变量 | `g_snake_case` | `g_system_ticks` |
| 函数 | `camelCase` | `uartInit()` |
| 结构体/枚举类型 | `snake_case_t` | `uart_handle_t` |
| 枚举值 | `PREFIXED_SNAKE` | `UART_STATE_IDLE` |
| 常量/宏 | `SCREAMING_SNAKE` | `UART_SR_RX_READY_MASK` |
| 指针 | 项目无约定时用清晰语义名 | `rx_buffer`, `handle`；或 `p_rx_buffer` |

### 2.2 类型与错误处理

- 公共接口优先使用 `<stdint.h>`、`<stdbool.h>`、`<stddef.h>`；默认 `uint8_t` / `uint16_t` / `uint32_t`、`int32_t`、`bool`
- 不把 `int`、`char`、`long` 作为默认跨平台接口类型

项目没有既有 status 类型时，公共函数默认返回 `embedded_code_status_t`：

```c
typedef enum {
    EmbedCode_Ok           =  0,
    EmbedCode_ErrNullPtr   = -1,
    EmbedCode_ErrInvalidArg = -2,
    EmbedCode_ErrTimeout   = -3,
    EmbedCode_ErrBusy      = -4,
    EmbedCode_ErrNotInit   = -5,
} embedded_code_status_t;

#define VALIDATE_NOT_NULL(ptr) \
    do { if ((ptr) == NULL) return EmbedCode_ErrNullPtr; } while (0)

#define VALIDATE_INIT(handle) \
    do { if (!(handle) || !(handle)->initialized) return EmbedCode_ErrNotInit; } while (0)
```

### 2.3 数据结构

重写或修改时：状态/错误/标志 → `enum`；配置/上下文 → `struct`；>3 标量参数 → 结构体指针；位域用 `MASK` 宏，禁裸魔数。

### 2.3.1 结构体模式

默认拆成配置、运行时句柄和状态：

```c
typedef struct {
    uint32_t base_address;
    uint32_t baud_rate;
} uart_config_t;

typedef struct {
    bool            initialized;
    volatile bool   tx_busy;
    uint8_t        *rx_buffer;
    uint16_t        rx_head;
    uint16_t        rx_tail;
    uart_config_t   config;
} uart_handle_t;

typedef enum {
    UART_STATE_IDLE = 0,
    UART_STATE_TX_BUSY,
    UART_STATE_RX_ACTIVE,
    UART_STATE_ERROR,
} uart_state_t;
```

### 2.4 注释

#### 2.4.1 注释原则

- 注释语言遵循项目约定；项目无约定时可用中文或用户指定语言。
- 注释解释硬件原因、约束、时序和意图；不要逐行复述代码。
- 注释应回答"为什么"而非"是什么"——代码本身说明"是什么"，注释补充"为什么这样写"。
- 保持注释与代码同步更新；过时注释比无注释更有害。
- **寄存器层注释要详细**：每个寄存器必须标注偏移地址 `/* 0xNN */`、位域含义 `bit[n]=功能`、读写属性。底层代码是硬件交互的唯一依据，注释不够详细会导致后续维护者误操作。
- **应用层注释侧重意图**：说明业务目的、调用约束、错误处理策略，不需要重复底层已有的位域信息。

#### 2.4.2 必须注释的场景

| 场景 | 说明 | 示例 |
|------|------|------|
| **操作步骤** | 驱动层和应用层函数体内的关键步骤必须用编号注释标注操作顺序 | 见下方示例 |
| 硬件约束 | 寄存器写入顺序、时序要求、errata workaround | `/* 必须先写 CTRL 再写 DATA，否则 FIFO 错位（Errata 3.2） */` |
| 非显而易见的逻辑 | 算法选择原因、特殊边界处理 | `/* 使用查表法而非计算，节省 12 个 CPU 周期 */` |
| 临时方案 | workaround、TODO、待确认项 | `/* FIXME: 临时绕过芯片 Rev.A 的 DMA 冻结问题 */` |
| 安全关键路径 | 看门狗刷新点、冗余检查、故障注入点 | `/* 喂狗必须在 SPI 传输完成后，否则超时复位 */` |
| 并发/中断相关 | critical section、屏障、volatile 使用原因 | `/* 关中断保护 tx_tail，ISR 中会修改 */` |
| 魔数来源 | 非自明的常量值来源 | `/* 1200 = 72MHz / (16 * 3750)，参考手册 §23.4.2 */` |

**操作步骤注释要求（驱动层 + 应用层均适用）：**

驱动层函数和应用层函数体内，每个关键操作步骤必须用编号注释标注，让读者不用看手册也能理解操作流程：

```c
embedded_code_status_t uartDrvInit(uart_drv_handle_t *p_handle, const uart_config_t *p_config)
{
    /* Step 1: 参数校验 */
    VALIDATE_NOT_NULL(p_handle);
    VALIDATE_NOT_NULL(p_config);

    /* Step 2: 禁用 UART，避免配置过程中产生意外中断 */
    UART_REG->ctrl.CTRL &= ~UART_CTRL_ENABLE_MASK;

    /* Step 3: 配置波特率（值来自参考手册 §23.4.2 公式） */
    UART_REG->ctrl.BAUD = calcBaudDiv(p_config->baud_rate);

    /* Step 4: 清除挂起状态和 FIFO 残留数据 */
    UART_REG->status.STATUS = 0U;

    /* Step 5: 使能外设，标记初始化完成 */
    UART_REG->ctrl.CTRL |= UART_CTRL_ENABLE_MASK;
    p_handle->initialized = true;
    return EmbedCode_Ok;
}
```

应用层同理：`Step 1: 参数校验` → `Step 2: 填充驱动配置` → `Step 3: 调用驱动层` → `Step 4: 初始化缓冲区和状态`。

#### 2.4.3 注释格式

**函数注释**（公共 API 必须有，静态辅助函数酌情）：

```c
/**
 * @brief  初始化 UART 并配置波特率
 * @param  handle  句柄指针，调用前需填充 config
 * @param  baud    目标波特率
 * @return EmbedCode_Ok 成功；EmbedCode_ErrNullPtr / EmbedCode_ErrInvalidArg
 * @note   调用前需确保 GPIO 已配置为 AF 模式
 */
embedded_code_status_t uartInit(uart_handle_t *handle, uint32_t baud);
```

**内联注释**：关键行上方或同行，解释原因而非复述代码。

```c
uint32_t timeout = UART_TX_TIMEOUT_MS;
while (!(UART_REG->status.STATUS & UART_STATUS_TX_EMPTY) && --timeout) {
    /* 等待发送完成，超时防硬件死锁 */
}
```

**结构体/寄存器注释**：字段含义非自明时才加，寄存器字段标注偏移和位域。

```c
typedef struct {
    uint8_t *rx_buffer;     /* 应用层分配，驱动层不管理生命周期 */
    uint16_t rx_head;       /* ISR 写入位置，主循环只读 */
    volatile bool tx_busy;  /* ISR 在发送完成时清零 */
} uart_handle_t;
```

#### 2.4.4 标记约定

项目无既有约定时，使用以下统一标记：

| 标记 | 含义 | 何时移除 |
|------|------|----------|
| `USER_PROVIDED` | 需用户填入真实硬件值 | 用户提供后 |
| `PLACEHOLDER` | 临时占位，功能正确但不完整 | 硬件确认后 |
| `REPO_DERIVED` | 从仓库推导，可能不准确 | 对照手册后 |
| `TODO:` | 待实现或待优化 | 完成后 |
| `FIXME:` | 已知缺陷或临时 workaround | 修复后 |
| `HACK:` | 依赖特定版本/硬件的临时方案 | 正式方案后 |
| `NOTE:` | 重要说明 | 永久保留 |
| `WARNING:` | 风险点，误改可能损坏硬件 | 永久保留 |
| `OPTIMIZE:` | 已验证的优化候选 | 实施后 |

用法：`/* FIXME: Rev.A 芯片需额外 2 个 NOP 等待 FIFO 稳定 */`

#### 2.4.5 文件头注释

- Doxygen 文件头只在项目已有模式时添加。若需要：

```c
/**
 * @file    uart_drv.c
 * @brief   UART 驱动层：寄存器操作，不含业务逻辑
 * @author  Your Name
 * @date    2024-01-01
 * @note    单元测试时需替换为 mock 层
 */
```

---

## 3. 寄存器抽象

- 每个外设 block 一个独立 `*_reg.h`，使用分层 `*_reg_t` 寄存器结构体或复用 vendor/CMSIS 已有结构体
- **分层规则**：功能模块独立子结构体（`*_mod_reg_t`），顶层按偏移组合；间隙用 `uint8_t reserved[n]` 或占位 `volatile uint32_t` 填充
- 每个寄存器用 `/* 0xNN */` 标注偏移地址，便于对照手册 review
- 一个明确入口访问寄存器（`*_REG` 或项目已有 wrapper），位字段使用 `MASK/SHIFT` 宏
- 不把裸寄存器地址写散在业务逻辑里
- 对 read-modify-write、reserved bits、write-one-to-clear、unlock sequence 保持谨慎

```c
/* ========== spi_reg.h ========== */
#define SPI_BASE_ADDR  (0xA0010000U)

/* 功能模块：控制与配置 */
typedef struct {
    volatile uint32_t CTRL;       /* 0x00 控制寄存器：bit[0]=EN, bit[2:1]=MODE, bit[3]=IE */
    volatile uint32_t MODE;       /* 0x04 模式寄存器：bit[0]=CPOL, bit[1]=CPHA */
    volatile uint32_t BAUD;       /* 0x08 波特率分频寄存器 */
    volatile uint32_t RESERVED0;  /* 0x0C 保留，仅作偏移占位，通常不访问 */
} spi_ctrl_reg_t;

/* 功能模块：状态与中断 */
typedef struct {
    volatile uint32_t STATUS;     /* 0x10 状态寄存器：bit[0]=TX_EMPTY, bit[1]=RX_FULL, bit[2]=BUSY */
    volatile uint32_t INT_EN;     /* 0x14 中断使能：bit[0]=TX_CPLT_IE, bit[1]=RX_CPLT_IE */
} spi_status_reg_t;

/* 功能模块：数据 */
typedef struct {
    volatile uint32_t DATA;       /* 0x18 数据寄存器：写入=TX FIFO，读取=RX FIFO */
} spi_data_reg_t;

/* 顶层：按偏移组合各功能模块 */
typedef struct {
    spi_ctrl_reg_t   ctrl;    /* 0x00 - 0x0F 控制与配置 */
    spi_status_reg_t status;  /* 0x10 - 0x17 状态与中断 */
    spi_data_reg_t   data;    /* 0x18 - 0x1B 数据 */
} spi_reg_t;

#define SPI_CTRL_EN_MASK    (1U << 0)
#define SPI_CTRL_MODE_MASK  (3U << 1)
#define SPI_REG  ((spi_reg_t *)SPI_BASE_ADDR)
/* SPI_REG->ctrl.CTRL |= SPI_CTRL_EN_MASK; */
```

---

## 4. 驱动模板

模板仅示组织方式；offset/reset/errata 须来自目标资料。三层五文件：reg（纯定义）→ drv（寄存器/ISR/DMA）→ app（缓冲/协议/API）。

### 4.1 统一结构

```text
module/
├── module_reg.h    # 寄存器结构体、位定义、基地址宏 — 无 .c，无函数实现
├── module_drv.h    # 驱动层：寄存器读写、ISR、DMA  — 不含业务逻辑、不分配 buffer
├── module_drv.c
├── module.h        # 应用层：缓冲管理、协议处理、对外 API — 不直写寄存器、不含 ISR
└── module.c
```

调用链：应用层 → 驱动层 → 寄存器层。驱动层 ISR 通过函数指针回调通知应用层，不直接操作 ring buffer。

### 4.2 接口模板

| 模块 | 驱动层 (`_drv.h`) | 应用层 (`.h`) |
|------|-------------------|---------------|
| UART | `uartDrvInit/DeInit/SendByte/RecvByte/EnableIRQ/RegisterCallbacks/StartDmaTx/StartDmaRx/IRQHandler` | `uartInit/DeInit/Send/Recv/SendIT/RecvIT/SendDMA/RecvDMA` |
| SPI | `spiDrvInit/Transfer/SetCS/EnableIRQ/RegisterCallback/StartDmaTransfer/IRQHandler` | `spiInit/Transfer/SelectCS/TransferIT/TransferDMA` |
| GPIO | `gpioDrvInit/WritePin/ReadPin/TogglePin/EnableIRQ/RegisterCallback/IRQHandler` | `gpioInit/WritePin/ReadPin/TogglePin/SetCallback` |
| DMA | `dmaDrvChannelInit/Start/Abort/IsComplete/IRQHandler` | `dmaChannelInit/StartTransfer/AbortTransfer/IsTransferComplete` |

app 的 IT/DMA 只编排缓冲与回调，委托 drv，不直写寄存器、不实现 ISR。

**Init 模式（驱动层）**：禁用外设 → 配置参数（值来自厂商或 PLACEHOLDER）→ 清挂起标志 → 使能 → 标记 initialized

**ISR 模式（驱动层）**：读 STATUS → 按 mask 分支 → 回调通知应用层 → 清中断标志 → 从不阻塞

### 4.3 关键结构

寄存器列按 `模块.寄存器` 格式表示分层访问（如 `ctrl.CTRL`）。

| 模块 | 寄存器（功能模块.寄存器） | 驱动层 handle | 应用层 handle |
|------|--------------------------|--------------|--------------|
| UART | `ctrl.CTRL/BAUD, fifo.DATA, status.STATUS, int.INT_EN` | `regs, rx/tx_callback` | `rx_buffer, rx_head, rx_tail, drv_handle` |
| SPI | `ctrl.CTRL/MODE/BAUD, status.STATUS/INT_EN, data.DATA` | `regs, cs_callback` | `drv_handle` |
| I2C | `ctrl.CTRL, addr.ADDR, fifo.DATA, status.STATUS` | `regs, addr, direction` | `timeout, bus_state, drv_handle` |
| DMA | `global.STATUS, ch[n].CTRL/SRC/DST/LEN` | `regs, channel_cfg[]` | `buffers[], drv_handle` |
| CAN | `ctrl.CTRL/BIT_TIMING, status.STATUS, fifo.TX/RX_DATA` | `regs, bit_timing` | `can_msg_t{id,dlc,data[8]}, drv_handle` |
| GPIO | `mode.MODE, input.DATA, output.DATA/BSR` | `regs, pin_map` | `pin_callbacks[], gpio_mode_t, drv_handle` |
| Timer | `ctrl.CTRL, count.COUNT, reload.AUTO_RELOAD/PRESCALER` | `regs, prescaler, auto_reload` | `period, callback, drv_handle` |
| Watchdog | `ctrl.KEY/RELOAD, status.STATUS` | `regs, key_reg` | `timeout_ms, drv_handle` |
| MIL-STD-1553 | `ctrl.CMD, status.STATUS, data.DATA[n]` | `regs, mode` | `msg_t, drv_handle` |

**反模式**：寄存器散落成地址宏、应用层直写寄存器、驱动层分配 buffer/处理协议帧、ISR 中阻塞、顶层结构体用扁平字段不分功能模块。

---

## 5. 架构规则

架构相关代码包括 ISR、barrier、DMA、cache、interrupt controller、SMP、memory ordering、CSR/SPR 和 board bring-up。

### 5.1 Quick Ref

| 架构 | Barrier | Interrupt | CSR/SPR | 代表芯片 |
|------|---------|-----------|---------|---------|
| ARM Cortex-M | `__DMB()/__DSB()/__ISB()` | NVIC | N/A | STM32, GD32, NXP |
| ARM Cortex-A | `dmb ish` | GIC | system registers | i.MX6/7, STM32MP |
| RISC-V | `fence` | PLIC/CLINT | `csrr` | FE310, CH32V |
| ESP32 (Xtensa) | `esp_cpu_dsb()` | INT matrix | `WSR`/`RSR` | ESP32, S2/S3 |
| PowerPC | `msync` /* PLACEHOLDER: 确认 `sync` vs `msync` 差异 */ | PIC | `mfspr` | MPC5748 |
| SPARC V8 | `stbar` /* PLACEHOLDER: 可能需补充 `set`/`lda` 完整性 */ | INTC | `rd psr` | LEON |

### 5.2 ESP32 关键差异

- ISR 必须标记 `IRAM_ATTR`，否则 Flash 操作期间崩溃；变量用 `DRAM_ATTR`
- 中断中使用 `...FromISR()` 后缀 FreeRTOS API
- SPI/I2C 通过 `spi_bus_initialize()` 等高层 API，非直接写寄存器
- 双核：Core 0 跑 WiFi/BT 协议栈，应用放 Core 1

### 5.3 RP2040 关键差异

- Pico SDK，双 Cortex-M0+；`multicore_launch_core1(entry)` 启动 Core 1
- 通过 `multicore_fifo_pop_blocking()` 跨核通信
- DMA 使用 `dma_claim_unused_channel()` + `dma_channel_config_t`

### 5.4 NRF52 关键差异

- nrfx 驱动层 + SoftDevice BLE 协议栈
- GPIO 中断通过 `nrfx_gpiote_in_init()` 回调注册
- 应用中断优先级必须 > SoftDevice 优先级
- 时序敏感操作放 PPI（可编程外设互连）

### 5.5 未知架构处理

1. 根据芯片、工具链、vendor headers 和已有低层代码判断架构
2. 不确定时查官方文档或要求用户提供资料
3. 不能确认时，只给出架构无关的审查建议或保守改写，barrier/interrupt/cache maintenance 标成 placeholder
4. 不猜测 barrier、cache maintenance、DMA ownership、IRQ number 或 interrupt-controller 行为

---

## 6. RTOS 指导

### 6.1 任务设计

| 关注点 | 规范 |
|--------|------|
| 栈大小 | 按实际使用量 + 20-30% 裕量，不盲目取默认值 |
| 优先级 | 优先级反转风险高的任务使用互斥量（非二值信号量） |
| 创建顺序 | 先创建同步原语，再创建使用它们的任务 |
| 看门狗 | 长期阻塞任务必须有喂狗机制 |

### 6.2 线程安全

- **任务间**：互斥量保护（`osMutexAcquire/Release`）
- **ISR→任务**：队列（FreeRTOS `xQueueSendFromISR` + `portYIELD_FROM_ISR`）
- **简单标志**：`<stdatomic.h>` 原子操作或 `volatile` + 显式 barrier

### 6.3 ISR 规则

ISR 禁阻塞；FreeRTOS 用 `...FromISR()`；只做读状态、清标志、通知、触发 DMA。

### 6.4 常用 API 对照

| 功能 | FreeRTOS | Zephyr | RT-Thread |
|------|----------|--------|-----------|
| 任务 | `xTaskCreate()` | `k_thread_create()` | `rt_thread_create()` |
| 互斥量 | `xSemaphoreCreateMutex()` | `K_MUTEX_DEFINE` | `rt_mutex_create()` |
| 信号量 | `xSemaphoreCreateBinary()` | `k_sem_init()` | `rt_sem_create()` |
| 队列 | `xQueueCreate()` | `k_msgq_init()` | `rt_mq_create()` |
| 延时 | `vTaskDelay(pdMS_TO_TICKS(ms))` | `k_msleep(ms)` | `rt_thread_mdelay(ms)` |

### 6.5 死锁预防

- 多锁场景按固定顺序获取
- 超时等待替代无限等待
- 持锁期间不调用可能阻塞的 API
- 优先级继承互斥量优先于二值信号量

---

## 7. 构建系统与链接

### 7.1 Linker Script 关键点

```ld
MEMORY { FLASH (rx): ORIGIN=0x08000000, LENGTH=512K  RAM (rwx): ORIGIN=0x20000000, LENGTH=128K }
SECTIONS {
    .text   : { *(.text*) }   > FLASH
    .rodata : { *(.rodata*) } > FLASH
    .data   : { __data_start = .; *(.data*) __data_end = .; } > RAM AT > FLASH
    .bss    : { __bss_start = .; *(.bss*) *(COMMON) __bss_end = .; } > RAM
}
```

### 7.2 Startup 代码

`Reset_Handler`：1) 搬运 `.data`（Flash→RAM）→ 2) 清零 `.bss` → 3) `SystemInit()` → 4) `main()`

### 7.3 编译器 Attribute

| 用途 | 写法 |
|------|------|
| 中断函数 | `__attribute__((interrupt("IRQ")))` |
| 指定 section | `__attribute__((section(".dma_buf")))` |
| 对齐 | `__attribute__((aligned(32)))` |
| 弱符号 | `__attribute__((weak))` |
| 始终内联 | `__attribute__((always_inline))` |

### 7.4 CMake 交叉编译

```cmake
set(CMAKE_SYSTEM_NAME Generic)
set(TOOLCHAIN_PREFIX arm-none-eabi-)
set(CMAKE_C_COMPILER ${TOOLCHAIN_PREFIX}gcc)
set(CMAKE_C_FLAGS_INIT "-mcpu=cortex-m4 -mthumb -Wall -Wextra")
set(CMAKE_EXE_LINKER_FLAGS_INIT "-T${CMAKE_SOURCE_DIR}/linker.ld -Wl,--gc-sections")
```

---

## 8. 测试与调试

### 8.1 HAL Mock 模式

通过函数指针表实现可替换 HAL：

```c
typedef struct {
    embedded_code_status_t (*init)(const uart_config_t *);
    embedded_code_status_t (*send)(const uint8_t *, uint16_t);
} uart_hal_t;
```

生产用 `uart_hal_hw`，测试用 `uart_hal_mock`，驱动通过 `handle->hal` 指针操作。

### 8.2 断言分级

- `STATIC_ASSERT(cond, msg)` → `_Static_assert`，编译期
- `ASSERT(cond)` → 运行时，调用 `assertFailed(file, line)`
- `SOFT_ASSERT(cond, action)` → 生产代码不 crash，记录错误后执行 action

### 8.3 On-Target 调试约定

| 约定 | 说明 |
|------|------|
| 调试引脚 | 保留至少 1 个 GPIO 用于 SWO/printf |
| 错误码追踪 | 错误路径记录文件/行号，宏开关控制 |
| 栈溢出检测 | 任务栈底部填 watermark pattern，运行后检查 |
| 日志级别 | ERROR > WARN > INFO > DEBUG，生产代码只保留 ERROR/WARN |

---

## 9. 行业领域

| 领域 | 关键词 | 默认要求 | 不要默认声明 |
|------|--------|----------|-------------|
| Aerospace | DO-178C, DAL, ARINC | 无动态分配、确定性行为、无递归、需求 ID 可追踪 | DAL 等级、MC/DC 覆盖率 |
| Military | 1553B, SpaceWire, MIL-STD | 冗余、SEU 防护、BIT diagnostics | MIL-STD 等级 |
| Industrial | IEC 61508, SIL, PLC | safe state 明确、watchdog supervision | SIL 等级、SPFM/LFM |
| Automotive | ISO 26262, ASIL, CANFD | 接口隔离、故障传播控制 | ASIL 等级 |
| General | — | 不默认声明认证要求 | — |

只有用户或项目资料明确标准和等级时才写合规结论。

---

## 10. 内存与并发

- 低层驱动、ISR 和 hot path 默认不用 `malloc/free/calloc/realloc`
- 禁止 VLA；缓冲区大小、timeout、retry count 使用命名常量
- critical section 要短、显式，沿用项目本地 helper
- ISR、DMA、cache coherency、volatile 和 memory ordering 必须保留硬件相关顺序
- 不发明 barrier、cache maintenance、DMA ownership 或 interrupt-controller 细节

---

## 11. 反例集

**1. 寄存器散落**：❌ `#define SPI_CTRL_ADDR (*(volatile uint32_t*)0xA0010000U)` + 魔法数字 → ✅ `SPI_REG->ctrl.CTRL = SPI_CTRL_INIT_VAL`

**1b. 扁平寄存器结构体**：❌ `typedef struct { volatile uint32_t CTRL; volatile uint32_t STATUS; ... } uart_reg_t;` → ✅ 按功能模块分层：`uart_ctrl_reg_t`、`uart_fifo_reg_t`、`uart_status_reg_t` 组合成 `uart_reg_t`

**2. DMA cache coherency**：❌ 直接 DMA 读写无 cache 处理 → ✅ 按架构处理：Cortex-M7/M33 等带 D-Cache 可用 `SCB_CleanDCache_by_Addr` / `SCB_InvalidateDCache_by_Addr`；无 cache 或不确定时放 non-cacheable section 并标注 `PLACEHOLDER`

**3. ISR 阻塞**：❌ ISR 内 `xSemaphoreTake(..., portMAX_DELAY)` → ✅ `xSemaphoreGiveFromISR()` + `portYIELD_FROM_ISR()`

**4. volatile 漏用**：❌ `bool g_transfer_done; while(!g_transfer_done){}` → ✅ `volatile bool g_transfer_done;`

**5. 优先级反转**：❌ `xSemaphoreCreateBinary()` 保护共享资源 → ✅ `xSemaphoreCreateMutex()` 优先级继承

---

## 12. 回查清单与维护自检

### 12.1 回查清单

- [ ] 仓库已有代码符合本规范则沿用，不符合则在不改变逻辑的前提下修改
- [ ] 驱动层与应用层分层清晰：应用层不直写寄存器，驱动层不含业务逻辑，ISR 通过回调通知应用层
- [ ] 硬件常量来自用户、仓库或 placeholder
- [ ] 不编造寄存器/IRQ/barrier/cache 规则
- [ ] 复用 vendor/CMSIS 结构
- [ ] 寄存器结构体按功能模块分层：每个模块有独立子结构体，顶层按偏移组合，间隙用 RESERVED 填充
- [ ] 无裸寄存器地址散落在业务逻辑
- [ ] 无默认动态内存和 VLA
- [ ] 命名/类型/错误处理符合本 skill 规范
- [ ] 优先使用结构体、枚举组织数据，无散落的独立变量、裸整数状态码或魔法常量
- [ ] REWRITE 保留行为/ABI/时序顺序
- [ ] REVIEW correctness 和硬件风险优先于风格
- [ ] ISR 无阻塞，RTOS 用 FromISR API
- [ ] 驱动层和应用层函数体内有编号步骤注释（Step 1 / Step 2 / ...）标注关键操作顺序
- [ ] DMA buffer 处理 cache coherency
- [ ] 共享变量正确使用 volatile/atomic/互斥量

### 12.2 维护自检

修改本 skill 后，用以下场景做 smoke check：

- `REWRITE`：保留 public API、ABI、寄存器写入顺序
- `REVIEW`：优先指出 race/volatile/barrier/ownership 风险
- RTOS：共享数据有互斥保护，ISR 只用 FromISR
- 领域：DO-178C/MIL-STD/IEC 61508/ISO 26262 只在用户明确时写合规结论

通过标准：不编造硬件事实、仓库代码符合本规范或在不改变逻辑前提下统一、子领域无丢失或矛盾。

---
> Source: [leon-2050/embedded-code-skill](https://github.com/leon-2050/embedded-code-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
