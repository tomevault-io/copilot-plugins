## invoke-opencode-acp

> Claude Code skill · 委托复杂任务至 OpenCode 子代理 · 节省 50-90% token

# invoke-opencode-acp

Claude Code skill · 委托复杂任务至 OpenCode 子代理 · 节省 50-90% token

## 关键约束

**精简冗余**：机器可读 > 人类可读 · 符号/缩写/列表 · 删助词
**冲突信代码**：代码为真 · 文档描述与实现冲突时信代码
**客观诚实**：不主观评价 · 不因用户情绪转移立场 · 不编造事实 · 立刻暴露不确定信息
**触发条件**：≥2 文件修改 · 重构 · 批量操作 · 代码审查 · 多步推理 · git 操作
**避免**：单文件快速编辑 · 递归调用 OpenCode
**Token 机制**：委托至 OpenCode → 使用替代模型（GLM-4.7/Qwen）→ 仅摘要入主对话 · Tradeoff: 能力下降
**TodoWrite**：≥3 步骤任务必须建 todo · 实时更新状态 · 完成立即标记
**子任务模型**：Task 工具可用 `model: "haiku"` 节省成本（简单任务）

## Skill 使用

调用后必须等待完成再执行其他操作 · 避免立即自行实现

## 技术栈

Claude Code CLI · OpenCode CLI（`npm install -g opencode`）· Node.js

## 核心命令

交互式（>30m）：
```bash
node ~/.claude/skills/invoke-opencode-acp/acp_client.cjs "$PWD" "任务描述" -o /tmp/output.txt
```
一次性（<30m）：
```bash
node ~/.claude/skills/invoke-opencode-acp/acp_client.cjs "$PWD" "任务描述" -o /tmp/output.txt -t 300
```
参数：`-o FILE` 必需 · `-t SECONDS` 默认 1800 · `-v` 详细模式→stderr

## 工作流

**交互式检查循环**：
1. `Bash(..., run_in_background=True)` → task_id
2. `TaskOutput(task_id, block=True, timeout=300000)` → 等待 5 分钟
3. `completed` → Read 输出文件 → 返回结果
4. `running` → `AskUserQuestion` 是否继续 → 继续则回到 2
5. 终止 → `KillShell(task_id)` 自动清理

## 目录结构

```
skills/invoke-opencode-acp/
  SKILL.md
  acp_client.cjs
tests/
  test_acp_client.js
```

## 代码规范

≥5 处非空断言立即重构类型架构（🚫 eslint-disable 批量压制）

## 输出格式

禁预告文字 · 状态用符号 ✓/✗/→ · 一次性批量 Edit · 数据优先 · 直达结论 · 工具间隔零输出 · 错误格式 ✗ {位置}:{类型} · 代码块零注释 · ≥2 条用列表 · 路径缩写（. 项目根 · ~ 主目录）· 禁总结性重复 · 进度 {当前}/{总数} · 提问直入

---
> Source: [phonowell/invoke-opencode-acp](https://github.com/phonowell/invoke-opencode-acp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
