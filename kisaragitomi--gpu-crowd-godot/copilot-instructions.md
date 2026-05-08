## fluidcrowd

> 该项目尽可能使用computeshader进行计算



在Godot中实现流体驱动群集
该项目尽可能使用computeshader进行计算

## 开发规范
- 每次修改代码结束后，必须确保 `crowd_sim.gd` 中 `const DEBUG := false`，避免 DEBUG 日志（_collect_all_frame / _check_crossing / _log_bow_frame）拖慢运行时性能（DEBUG=true 时每帧额外 50-150ms）

---
> Source: [KisaragiTomi/GPU-Crowd-Godot](https://github.com/KisaragiTomi/GPU-Crowd-Godot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
