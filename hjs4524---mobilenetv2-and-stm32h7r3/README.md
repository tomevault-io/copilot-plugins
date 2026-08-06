# -MobileNetV2-and-STM32H7R3-

> Source: [hjs4524/-MobileNetV2-and-STM32H7R3-](https://github.com/hjs4524/-MobileNetV2-and-STM32H7R3-). Graded, signed and kept current as the models change by [TomeVault](https://tomevault.io)

这是一个基于 STM32H7R3 和 OpenMV 的机械臂动态抓取项目。MobileNetV2负责在MCU端进行嵌入式AI推理，OpenMV 负责识别桌面目标并输出目标中心像素坐标，STM32 通过串口接收坐标数据，经过二维手眼标定转换为桌面坐标，再结合 4 自由度机械臂逆运动学计算舵机角度，最终完成目标抓取、搬运、投放和回到 Home 位的完整流程。  项目使用 Keil MDK 开发，底层基于 STM32 HAL，包含 USART 通信、TIM PWM 舵机驱动、6 路舵机控制、目标稳定判定、桌面坐标转换和抓取流程控制等模块。适合作为嵌入式视觉抓取、OpenMV 与 STM32 通信、机械臂运动控制的学习和实验项目

## GitHub Copilot Config

The `copilot-instructions.md` file in this directory is the project config converted for GitHub Copilot.
Original source: `AGENTS.md` in [hjs4524/-MobileNetV2-and-STM32H7R3-](https://github.com/hjs4524/-MobileNetV2-and-STM32H7R3-).

## Also available for

- **Claude Code** — `CLAUDE.md`
- **GitHub Copilot** — `copilot-instructions.md`
- **Cursor** — `project-config.mdc`
- **Gemini CLI** — `GEMINI.md`
- **Windsurf** — `project-config.md`

Available for 6 platforms including Claude Code, Cursor, Windsurf, and more. Source: [github.com/hjs4524/-MobileNetV2-and-STM32H7R3-](https://github.com/hjs4524/-MobileNetV2-and-STM32H7R3-)

---

Own this repo? Install the TomeVault Relay to keep every platform's copy in sync on every push: [https://tomevault.io/install](https://tomevault.io/install).

<!-- genome:d-c-p -->
