## easy-input-maker

> 本仓库用于公开的 EasyInput Maker 社区固件开发。AI Agent 和贡献者必须把所有准备提交的内容视为公开材料。

# EasyInput Maker · AI 贡献指南

本仓库用于公开的 EasyInput Maker 社区固件开发。AI Agent 和贡献者必须把所有准备提交的内容视为公开材料。

## 开始前

1. 阅读 `README.md`、`AI_DEVELOPMENT.md` 和当前任务相关代码。
2. 涉及引脚、BOOT、GPIO8、USB、音频、睡眠或唤醒时，先阅读 `docs/hardware/easyinput-v2-safety.md`。
3. 涉及许可证、第三方代码或素材时，检查 `LICENSE`、`THIRD_PARTY_NOTICES.md` 和对应资源说明。

## 开发边界

- 目标硬件为 EasyInput V2.0 / ESP32-S3，构建基线为 ESP-IDF 5.5.5。
- 可宿主测试的纯逻辑优先放在 `components/keyboard/`；ESP-IDF 硬件适配放在 `main/platform/`。
- 不擅自修改 GPIO、BOOT 流程、分区、设备身份、通信协议或 GPIO8 共享电源生命周期。
- 不提交真实 Wi-Fi 凭据、同步密钥、账号、设备唯一标识、本机绝对路径、原始串口输出、内部仓库信息或未整理的执行记录。
- 运行日志不得输出密码、密钥、SSID、网络主机、BLE 地址、用户文本或用户自定义快捷键内容。
- 构建成功、自动测试通过、烧录成功和实板功能正常是不同证据，报告时必须明确区分。

## 默认验证

```bash
cmake -S host_test -B build-host -DCMAKE_BUILD_TYPE=Debug
cmake --build build-host
ctest --test-dir build-host --output-on-failure
idf.py build
```

构建目录、二进制、映射文件、缓存和日志不得进入版本控制。烧录会覆盖设备固件，只有使用者明确要求并确认目标设备后才能执行。

## 公开贡献

- Issue 和 Pull Request 只记录可公开复现的现象、必要证据和最终修改。
- 不粘贴完整私有日志；仅摘录完成脱敏且与问题直接相关的最小片段。
- 项目自有材料遵守根目录非商业许可证；第三方材料继续遵守其原许可证。

<!-- project-flow-cy:start -->
## 项目协作流程

本仓库使用根目录唯一一套 `flow/` 与 `docs/` 进行协作。`flow/` 管理项目推进，`docs/` 存放可以公开的产品、技术与教学资料。

### 开工前

1. 阅读 `flow/charter.md`、`flow/plan.md` 和 `flow/进展.md` 顶部一条记录。
2. 有具体任务卡时，阅读 `flow/tasks/` 中对应文件。
3. 所有准备写入 `flow/` 的内容同样视为将公开发布，不记录内部讨论、私有仓库、人员隐私、凭据、原始日志或未经整理的执行过程。

### 收工前

1. 仅在确有公开协作价值时，向 `flow/进展.md` 顶部追加脱敏后的结果、公开产出路径和下一步。
2. 可公开的推进决策写入 `flow/decisions.md`；可公开且可复用的问题经验写入 `flow/踩坑记录.md`。
3. 进展与交接只写“指针 + 增量”，不复制产品文档或代码内容。

### 目录与规则

- `flow/charter.md`：项目目标、范围、约束与成功标准。
- `flow/plan.md`：已经确认的实施计划。
- `flow/进展.md`：公开协作进展，最新记录置顶。
- `flow/decisions.md`：可公开的过程决策。
- `flow/踩坑记录.md`：可公开的问题与经验。
- `flow/tasks/`：任务模板与公开任务卡。
- `flow/规范/`：`project-flow-cy` 的协作方法规范。
- 完整规则见 `flow/规范/工作流程.md` 和 `flow/规范/文档维护SOP.md`。
<!-- project-flow-cy:end -->

---
> Source: [CY-CHENYUE/easy-input-maker](https://github.com/CY-CHENYUE/easy-input-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
