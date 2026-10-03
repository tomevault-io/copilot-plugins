## fruit-fly-computer

> - 先读 `README.md`、`docs/project-review.md`、`docs/backlog.md`、`CONTRIBUTING.md` 和 `docs/reports/mac-foundation-2026-09-20.md`。

# 项目工作约定

## 事实入口

- 先读 `README.md`、`docs/project-review.md`、`docs/backlog.md`、`CONTRIBUTING.md` 和 `docs/reports/mac-foundation-2026-09-20.md`。
- 原始需求来源为 `docs/reference/fly_computer_plan_v0_2.md`；实施细化见 `docs/project-plan.md`。新用户指令优先。
- Mac 软件基础核心已实现；真实图、add4 与 CUDA 验收待完成。更新状态必须附实际产物或报告。
- 通用沟通规则继承用户的全局约定，此处只保留项目补充。

## 实现边界

- 正式推理只允许逐位编码、真实神经核心、固定特征提取、线性读出、阈值和结果显示；不得加载标签表或用标准加法修补结果。
- 训练与独立验收可生成答案；运行时不得依赖这两个模块。代码审查与运行测试共同核实边界。
- 输入、输出向量低位在前，显示字符串高位在前；连接矩阵为 `W[receiver, sender]`。
- 默认 CPU / CUDA、FP32、稀疏核心、每次复位。MPS 可选；禁止静默后端回落或完整图稠密化。
- 人工图只验证软件，真实子网络必须记录来源和范围；不得把子网络报告称为完整连接组结果。
- 变更图、动力学、端口、精度、训练范围、信号协议时更新配置标识，并重做受影响的验收。

## 产物与验证

- 按 `docs/validation.md` 建立测试；不为目录占位和文档编写形式化测试。
- 大数据和模型放在已忽略目录；精选报告要可追溯到代码、环境、数据、配置、模型哈希。
- 不把 Mac 结果视为 DGX 验收；待执行、失败、跳过都不能记作通过。
- 领取任务后更新 `docs/backlog.md`；环境或里程碑状态变化时补充报告，并更新 README 入口。
- 工作以 GitHub issue 认领和独立分支 PR 交付，等待用户指定负责人；当前不要擅自 @ 或分配其他账号。

---
> Source: [appergb/fruit-fly-computer](https://github.com/appergb/fruit-fly-computer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
