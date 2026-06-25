## unnamed-skill

> M1/M2 编码器校准操作文档 — KTH71 ANLC 非线性校准 + 电角度零点标定


# M1/M2 编码器校准操作文档

## 概述

每台电机的编码器校准分两步，**必须按顺序执行**：

1. **KTH71 ANLC 非线性校准** — 消除磁场非线性误差，需电机旋转
2. **电角度零点标定** — 对齐 d 轴零点，需电机静止吸合

> **为什么要先 ANLC 再零点？**
> ANLC 校准修改编码器的非线性补偿表，会改变角度输出。
> 如果先做零点再做 ANLC，零点会被 ANLC 结果偏移掉。

---

## 硬件信息

| 项目 | M1 (Enc1) | M2 (Enc2) |
|------|-----------|-----------|
| SPI | SPI3 | SPI1 |
| CS 引脚 | PA15 | PA4 |
| 编码器芯片 | KTH7111 | KTH7111 |
| 极对数宏 | `M1_POLE_PAIRS` | `M2_POLE_PAIRS` |

---

## 一、KTH71 ANLC 非线性校准

### 前置条件

| 条件 | 要求 | 检查方法 |
|------|------|---------|
| 供电 | 12V 标称, 实测 ~11.9V | Live Watch `g_Vbus_mV` |
| 电流环 | **已校准** (PI 参数有效) | `g_M1_PI_d.kp` / `g_M2_PI_d.kp` 非默认值 |

> **M2**: 触发 `g_DoKth71Calib_M2=1` 后代码全自动 — 自动切开环、用 `g_M2_Iq_Ref_mA`(默认100mA) 作力矩、用 `KTH71_CALIB_RPM`(800RPM) 作校准转速，与 `g_M2_RPM_Cmd` 无关。
> **M1**: 触发 `g_DoKth71Calib_M1=1` 后代码全自动 — 自动切开环、用 `g_M1_Iq_Ref_mA`(默认50mA) 作力矩、校准转速取 `g_M1_RPM_Cmd`(默认400RPM，因800RPM对高电感电机丢步)。

### 操作步骤

1. **直接触发校准** (无需手动设任何参数，代码自动接管):
   - M1: Live Watch 置 `g_DoKth71Calib_M1 = 1`
   - M2: Live Watch 置 `g_DoKth71Calib_M2 = 1`
3. **自动过程** (~15-20 秒):
   - 代码自动切开环，设 Iq 和转速
   - VOFA 自动切到 `VOFA_SRC_KTH71_CALIB`
   - 芯片 PWM 引脚 3.8Hz 闪烁 = BUSY
   - 闪烁停止 = 校准完成或失败
4. **校准完成**: VOFA 切到 `VOFA_SRC_IDLE`（停止发送，保留结果）
5. **自动 MTP 固化**: ANLC 成功后代码自动写 0x16=0x08 + 烧 MTP，无需零点标定即可掉电保存
6. **恢复发送**: 需手动置 `g_VofaSrc = 1`(`VOFA_SRC_NORMAL`)

### VOFA 通道 (校准期间)

| 通道 | 含义 | 正常值 |
|------|------|--------|
| I0 | 调试步骤 | 1(INIT)→2(RAMPUP)→3(POLLING)→**4**(DONE) 或 6(TIMEOUT) |
| I1 | Id (mA) | ~Iq_Ref |
| I2 | Iq (mA) | ~Iq_Ref |
| I3 | Vd (Q15) | PI 输出 |
| I4 | Vq (Q15) | PI 输出 |
| I5 | 角度 (°) | 0~360 连续变化 |

> `g_Kth71CalibMotor`: 0=M2(默认), 1=M1 — 校准时 VOFA 填哪台电机数据

### 判定标准

| 结果 | I0 值 | 含义 |
|------|-------|------|
| **成功** | 4 (DONE) | ANLC_STATUS = DONE(3) |
| **超时** | 6 (TIMEOUT) | 20s 内未完成，检查转速/接线 |
| **失败** | — | ANLC_STATUS = FAIL(2), 检查磁铁/气隙 |

### 常见问题

| 现象 | 原因 | 解决 |
|------|------|------|
| I0 卡在 3 (POLLING) 超时 | 二次校准未复位状态机 | 已修复 (calib_end→delay→calib_start) |
| 电机只转 1s 就停 | 校准转速 delta 过大导致丢步 | 降低 `g_M1_Iq_Ref_mA` / `g_M2_Iq_Ref_mA` 或检查 PI |
| I0 直接跳到 6 | SPI 通信故障 | 检查对应 SPI/CS 接线 (M1=SPI3/PA15, M2=SPI1/PA4) |

---

## 二、电角度零点标定

### 原理

通过 Id 吸合将转子锁定到 d 轴（电角度 0°），读取此时编码器角度，
反算 ZERO 寄存器值写入 KTH7111，使编码器零位与电角度零位对齐。

### 前置条件

| 条件 | 要求 | 检查方法 |
|------|------|---------|
| ANLC 校准 | **已完成** (先做 ANLC) | 上一步 I0=4 |
| 控制模式 | 开环 (`g_M1_CtrlMode = 0` / `g_M2_CtrlMode = 0`) | Live Watch |
| 电机 | **静止** (RPM=0, Iq 有值无所谓) | 标定流程会自动停机 |

### 操作步骤

1. **触发标定**: Live Watch 置对应变量 = 1
   - M1: `g_DoZeroCalib_M1 = 1`
   - M2: `g_DoZeroCalib_M2 = 1`
2. **自动过程** (~4 秒，阻塞式):
   - ① 切开环，等待 1s 消除惯性
   - ② Id = 300mA 吸合，Iq = 0
   - ③ 摆动 4 次对齐: -90° → 0° → +90° → 0°（每次 500ms）
   - ④ 0° 位置保持 2s 锁定
   - ⑤ 暂停 ISR 编码器读取，读角度 + 计算 + 写 ZERO 寄存器
   - ⑥ 写入 MTP（掉电保存，≥400ms）
   - ⑦ 释放 Id/Iq
3. **验证**: 观察 `g_ZeroCalibAngle_M1` / `g_ZeroCalibAngle_M2` 有非零值

### 零点公式

```
RD=1: new_zero = old_zero - enc_angle
RD=0: new_zero = old_zero + enc_angle
```

- `old_zero`: C 变量 `g_ZeroCalibAngle_M1` / `g_ZeroCalibAngle_M2`（不读寄存器，规避 ReadReg bug）
- `enc_angle`: 吸合锁定后的编码器原始角度
- uint16 自然溢出 = mod 65536

### 关键参数

| 参数 | 值 | 定义位置 |
|------|-----|---------|
| 吸合电流 | 300 mA | `ZERO_CALIB_ID_MA` (zero_calib.h) |
| 摆动幅度 | ±90° 电角度 (±16384 Q15) | `ZERO_CALIB_SWING_Q15` |
| 摆动保持 | 500 ms | `ZERO_CALIB_SWING_MS` |
| 锁定保持 | 2000 ms | `ZERO_CALIB_LOCK_MS` |
| 静止等待 | 300 ms | `ZERO_CALIB_SETTLE_MS` |

### 验证方法

1. **Live Watch 检查**: `g_ZeroCalibAngle_M1` / `g_ZeroCalibAngle_M2` 应有合理非零值
2. **闭环测试**: 切速度模式，给正 RPM_Cmd，电机应正转且 `g_Enc1_SpeedFilt` / `g_Enc2_SpeedFilt` 同号
3. **Id 测试**: 闭环下给 Id 指令，电流应顺利建立无振荡

### 常见问题

| 现象 | 原因 | 解决 |
|------|------|------|
| 摆动时电机不动 | Id 不够大 | 增大 `ZERO_CALIB_ID_MA` |
| 标定后闭环振荡 | 零点偏差过大 | 重新标定，确认 ANLC 已校准 |
| 标定后正负转速反 | 编码器方向与 FOC 坐标系相反 | 软件取反速度或交换 A/B |
| `g_ZeroCalibAngle_M1` / `g_ZeroCalibAngle_M2` 为 0 | SPI 读角度失败 | 检查对应 SPI/CS 接线 |
| 上电后 PWM 引脚闪烁 (自动重校准) | 0x16 被 MTP 烧入 0x18 (REG_CAL=1) | 已修复: ANLC 和零点标定均在 MTP 前显式写 0x16=0x08 |

---

## 三、完整校准流程 (推荐顺序)

以下流程 M1 和 M2 **各自独立执行**，按需替换变量后缀。

```
┌──────────────────────────────────────────────────────────────┐
│           完整校准流程 (首次上电 / 换电机后)                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: 电流环校准                                           │
│  ─────────────────                                           │
│  M1: g_DoCurrLoopCalib_M1 = 1                                │
│  M2: g_DoCurrLoopCalib_M2 = 1                                │
│  → 自动测 Rs/Ls → PI 参数写入 → Flash 保存                    │
│  → VOFA 停在 IDLE, 检查 I4(Rs) / I5(Ls)                      │
│                                                              │
│  Step 2: KTH71 ANLC 非线性校准                                │
│  ─────────────────────────                                    │
│  确认开环转动正常                                              │
│  M1: g_DoKth71Calib_M1 = 1                                   │
│  M2: g_DoKth71Calib_M2 = 1                                   │
│  → 芯片 PWM 闪烁 → I0=4(DONE)                                │
│  → 成功后自动写 0x16=0x08 + MTP 固化                          │
│  → VOFA 停在 IDLE                                             │
│                                                              │
│  Step 3: 电角度零点标定                                        │
│  ─────────────────────                                        │
│  M1: g_DoZeroCalib_M1 = 1  → 检查 g_ZeroCalibAngle_M1        │
│  M2: g_DoZeroCalib_M2 = 1  → 检查 g_ZeroCalibAngle_M2        │
│  → Id 吸合摆动 ~4s → ZERO 写入 MTP                            │
│                                                              │
│  Step 4: 验证                                                 │
│  ─────────                                                    │
│  M1: g_M1_CtrlMode = 1, g_M1_RPM_Cmd = 100                   │
│      观察 g_Enc1_SpeedFilt ≈ 100                              │
│  M2: g_M2_CtrlMode = 1, g_M2_RPM_Cmd = 100                   │
│      观察 g_Enc2_SpeedFilt ≈ 100                              │
│  VOFA I7(Id) / I8(Iq) 应稳定无振荡                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 四、相关变量速查

| 用途 | M1 变量 | M2 变量 |
|------|---------|---------|
| 触发 ANLC 校准 (置 1) | `g_DoKth71Calib_M1` | `g_DoKth71Calib_M2` |
| 触发零点标定 (置 1) | `g_DoZeroCalib_M1` | `g_DoZeroCalib_M2` |
| 零点角度 (uint16_t) | `g_ZeroCalibAngle_M1` | `g_ZeroCalibAngle_M2` |
| 编码器实时角度 (16-bit) | `g_Enc1_Angle` | `g_Enc2_Angle` |
| 编码器滤波转速 (RPM) | `g_Enc1_SpeedFilt` | `g_Enc2_SpeedFilt` |
| 开环力矩 (mA) | `g_M1_Iq_Ref_mA` (默认 50) | `g_M2_Iq_Ref_mA` (默认 100) |
| 开环转速 (RPM) | `g_M1_RPM_Cmd` (默认 400) | `g_M2_RPM_Cmd` (默认 0) |
| 控制模式 | `g_M1_CtrlMode` | `g_M2_CtrlMode` |
| VOFA 电机选择 | `g_Kth71CalibMotor` = 1 | `g_Kth71CalibMotor` = 0 (默认) |
| VOFA 数据源 | `g_VofaSrc` = 1 恢复正常 | 同左 |

---
> Source: [TigerBruce/Double_SPET_FOC](https://github.com/TigerBruce/Double_SPET_FOC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
