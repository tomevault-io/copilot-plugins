## comm-protocol-knowledge

> RK3576 ↔ STM32G474 通信协议模块知识库


# RK3576 通信协议模块

## 架构概览

```
RK3576 (Linux)  ──── UART 1152000 8N1 ────  STM32G474 (FOC 控制器)
                PB6(TX) / PB7(RX)
字节序: 小端 (Little-Endian), 与双方 CPU 原生字节序一致
```

### 数据流

```
RX 路径:
  USART1_RDR → DMA1_CH2 Circular → s_rxBuf[128]
  USART1 IDLE ISR (优先级 6) → 更新 s_rxWrIdx (读 DMA NDTR)
  while(1) Protocol_Poll() → 帧解析 + CRC + 命令派发

TX 路径:
  TIM4 ISR (1kHz, 优先级 10) → Comm_OnTIM4_1ms()
    PROTOCOL 模式 → build_status_frame() → s_txBuf → DMA1_CH1 发送
    VOFA 模式     → 设 DMA MemAddr=&g_VofaFrame → Vofa_OnTIM4_1ms()

TX DMA 共享:
  协议状态帧 和 VOFA 帧 共用 DMA1_CH1 → USART1_TDR
  Comm_OnTIM4_1ms() 在 DMA 启动前设 MemoryAddress 确保源缓冲区正确
```

## 文件结构

| 文件 | 职责 |
|---|---|
| `comm_protocol.h` | 类型/常量/帧格式结构体(packed+static_assert)/命令枚举/API 声明 |
| `comm_protocol.c` | CRC-CCITT 查表, RX DMA 初始化, IDLE, 环形缓冲解析, 命令派发, 状态帧 TX, 看门狗 |
| `stm32g4xx_it.c` | USART1_IRQHandler (IDLE), TIM3 看门狗递增, TIM4 改调 Comm_OnTIM4_1ms() |
| `main.c` | Comm_Init() + while(1) Protocol_Poll() + g_UpSeqNo++ |

## 帧格式

```
[0xAA][0x55][Len(1B)][Payload(N B)][CRC16(2B, 小端)]
CRC-CCITT: poly=0x1021, init=0xFFFF, 无反射, 无最终异或
覆盖范围: Header + Len + Payload (即帧中除 CRC 本身外所有字节)
测试向量: CRC("123456789") = 0x29B1
最小帧: 6B, 最大: 2+1+64+2 = 69B
```

## 状态帧结构 (48B 总帧长)

```
Payload (43B):
  [0]     CmdID=0x80     [1]  TxSeqNo     [2]  LastRxSeq    [3]  UpSeqNo
  [4]     ProtoVer=1     [5]  SysFlags    [6]  LastCmdResult
  [7:8]   Vbus_mV        [9:10] CrcErrCount
  ---- M1 (16B) ----
  [11]    CtrlMode       [12] Flags
  [13:16] Speed_mRPM     (i32, 0.001 RPM = milliRPM)
  [17:20] PosFbk         (i32, encoder counts)
  [21:22] Id_mA          [23:24] Iq_mA     [25:26] Id_Eff_mA
  ---- M2 (16B) ----
  [27:42] (与 M1 相同)
```

## CAPS 响应 (14B 总帧长)

```
Payload (9B):
  [0]   CmdID=0x81    [1:2] FwVersion(u16)   [3] ProtoVer
  [4]   AxisCount=2   [5:8] Features(u32, bitmask)

Features: bit0=VOFA, bit1=Follow, bit2=FlashParams, bit3=CurrCalib,
          bit4=KTH71Calib, bit5=ZeroCalib, bit6=StepCapture
```

## 速度分辨率

- 协议层: **int32_t milliRPM (0.001 RPM)**, 范围 ±2,147,483 RPM
- 固件内部: `g_EncX_SpeedFilt` 是 int32_t RPM 整数, 频率法分辨率约 1 RPM
- 状态帧: `speed_mrpm = SpeedFilt * 1000` — 忠实传递固件数据
- **后续优化**: 改用周期测量法可将内部分辨率提升到 0.01 RPM 量级

## 命令总表

| CmdID | 名称 | Payload大小(含CmdID) | 帧总长 | 频率 | 状态 |
|---|---|---|---|---|---|
| 0x01 | QUERY_CAPS | 1B | 6B | 握手 | ✅ |
| 0x10 | SET_MOTION | 6B | 11B | ≤200Hz | ✅ |
| 0x11 | SET_POSITION | 6B | 11B | 偶发 | ✅ |
| 0x12 | SET_IQ_REF | 5B/9B/13B | 10B/14B/18B | 偶发 | ✅ (v3: +Id+AngleDelta) |
| 0x20 | SET_CTRL_MODE | 3B | 8B | 偶发 | ✅ |
| 0x21 | SET_FOLLOW | 2B | 7B | 偶发 | ✅ |
| 0x30 | SET_SPEED_PI | 10B | 15B | 偶发 | ✅ |
| 0x31 | SET_POS_PID | 14B | 19B | 偶发 | TODO |
| 0x32 | SET_ID_ADAPT | 8B | 13B | 偶发 | TODO |
| 0x33 | SET_RAMP_RATE | 4B | 9B | 偶发 | TODO |
| 0x40~0x42 | 校准触发 | 2B | 7B | 偶发 | ✅ |
| 0x43 | FLASH_UNLOCK | 1B | 6B | 偶发 | ✅ |
| 0x44 | FLASH_ERASE | 3B | 8B | 偶发 | ✅ |
| 0x50 | QUERY_STATUS | 1B | 6B | 按需 | ✅ |
| 0x51 | SET_VOFA_MODE | 3B | 8B | 调试 | ✅ |
| 0x52 | EXIT_VOFA | 1B | 6B | 调试 | ✅ |
| 0x53 | SET_STATUS_RATE | 3B | 8B | 偶发 | ✅ |

## 看门狗

- `g_CommWatchdogMs`: TIM3 ISR 每 1ms 无条件递增, Protocol_Poll 中有效帧清零
- 超时阈值: `g_CommTimeoutMs` 默认 200ms
- 超时动作: 切开环 + Iq/Id 归零 (零电流自由滑行)
- RK3576 空闲时至少 5Hz 发 QUERY_STATUS 保活
- **校准期间放宽**: handle_trig_calib() 设 g_CommTimeoutMs=20000, PollCalibTriggers() 完成后恢复 200

## 与 VOFA 共存

- `g_CommMode`: COMM_MODE_PROTOCOL (默认, 上电) / COMM_MODE_VOFA
- `vofa_engine.c` 完全不修改
- VOFA 模式切换: CMD_SET_VOFA_MODE (0x51) / CMD_EXIT_VOFA (0x52)
- VOFA 模式下仍可收命令 (Protocol_Poll 持续运行)
- VOFA 模式下 QUERY_STATUS 可临时插入一帧状态帧
- DMA 源地址切换: PROTOCOL 用 s_txBuf, VOFA 用 &g_VofaFrame

## NVIC 优先级 (以 CubeMX .ioc 生成为准, PRIORITYGROUP_4)

| 优先级 | 中断 | 说明 |
|---|---|---|
| 0 | HRTIM1 FLT | 硬件故障保护 |
| 1 | ADC1_2 (17kHz) | FOC 电流环 |
| 2 | HRTIM1 Master | PWM 周期更新 |
| 3 | TIM3 (1kHz) | 速度环/位置环/校准/看门狗递增 |
| 7 | DMA1_CH2 (RX完成) | CubeMX 配, Comm_Init 已禁用 |
| 8 | DMA1_CH1 (TX完成) | CubeMX 配, TX DMA 完成清理 |
| 9 | USART1 (IDLE) | CubeMX 配, RX 写指针更新 |
| 10 | TIM4 (1kHz) | 通信 TX / VOFA 发送 |
| 15 | SysTick | HAL_IncTick |

> 中断优先级由 CubeMX 统一管理, 代码中不手动设置 NVIC_SetPriority。

## 全局变量

| 变量 | 定义位置 | 用途 |
|---|---|---|
| `g_CommMode` | comm_protocol.c | PROTOCOL / VOFA 模式切换 |
| `g_UpSeqNo` | comm_protocol.c | while(1) 递增, 主循环存活证明 |
| `g_CrcErrCount` | comm_protocol.c | CRC 校验失败累计 |
| `g_CommWatchdogMs` | comm_protocol.c | 看门狗计数 (TIM3 每 1ms 无条件递增) |
| `g_CommTimeoutMs` | comm_protocol.c | 看门狗超时阈值, 默认 200ms |

## CubeMX 注意事项

- **USART1_IRQn**: CubeMX 已启用, 优先级 (9,0), NVIC 配置由 MX_USART1_UART_Init() 完成
- **DMA1_CH1**: 优先级 (8,0), TX DMA 完成
- **DMA1_CH2**: 优先级 (7,0), Comm_Init() 中已禁用 (Circular 不需要完成中断)
- **USART1_IRQHandler**: 定义在 CubeMX 生成的 stm32g4xx_it.c, IDLE 处理代码放在 USER CODE BEGIN USART1_IRQn 0
- **Comm_Init() 不再设 NVIC**: 仅做 DMA 地址绑定 + LL_USART_EnableIT_IDLE, NVIC 全部由 CubeMX 管理
- **中断优先级只由 CubeMX 管理**, 代码中不手动调用 NVIC_SetPriority
- 如重新生成代码, 需确认 USER CODE 区域保留

## volatile 共享变量

ISR (TIM4) 和 while(1) 之间共享的 static 变量:
- `s_lastRxSeq` / `s_lastCmdResult` / `s_pendingCapsRsp` / `s_forceStatusOnce` — 均已标 volatile
- while(1) 写, TIM4 ISR 读 → volatile 确保编译器不优化掉读取

## 待实现 / 后续迭代

- [x] SET_SPEED_PI (0x30) 已实现: Motor(u8)+kp(f32)+ki(f32)=9B
- [x] SET_IQ_REF 扩展: 4B/8B/12B 可选 (含 Id + AngleDelta), 向后兼容
- [x] watchdog_safety_action + IQ_REF 超时均清零 AngleDelta_Target
- [x] 校准看门狗: handle_trig_calib 放宽 20s, PollCalibTriggers 完成后恢复 200ms
- [ ] SET_POS_PID / SET_ID_ADAPT / SET_RAMP_RATE 参数设置命令
- [ ] 速度估算改周期测量法, 提升低速分辨率
- [x] g_CommWatchdogMs 递增已在 TIM3 ISR 中添加 (无条件, 优先于校准/速度环)
- [x] NVIC 优先级已由 CubeMX 统一配置: DMA1_CH1=8, DMA1_CH2=7, USART1=9
- [ ] I²t 热保护模型
- [ ] SET_MOTION 的 M2_target 限幅需与实际机械限位对齐
- [ ] 模式切换原子边界 (TIM3 ISR 生效 + Iq 斜坡归零)
- [ ] 校准改非阻塞状态机 (ZeroCalib + KTH71)
- [ ] IWDG 硬件看门狗配置

---
> Source: [TigerBruce/Double_SPET_FOC](https://github.com/TigerBruce/Double_SPET_FOC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
