## no-long-shell-wait

> 计划/需求改完后禁止长时间挂起等待 Shell；交付即收尾


# 禁止交付后长时间等待 Shell

## 硬性要求

1. **改完即交付**：计划执行、需求修改、缺陷修复完成后，立刻给出结论/变更说明并结束回合，**禁止**为「再等一会儿看输出」而长时间阻塞。
2. **禁止空等**：不得在工作已完成后用 `AwaitShell`、大 `block_until_ms`、反复 poll、或 `sleep` 空转等待终端。
3. **Shell 只服务进行中的步骤**：仅在**下一步立刻依赖**该命令结果时才等待；无关后台任务交给完成通知，不要主动长轮询。
4. **默认短超时**：非编译/测试类命令 `block_until_ms` 宜短（通常 ≤30s）；确需更长时，只等必要结果，到点就汇报进展或失败，不要无限挂起。
5. **本仓库编译/启动**：仍遵守「禁止自动 `cargo build` / `cargo run` / `npm run dev` 等」；不要用长等待编译来「验证交付」。

## 反例 / 正例

```text
❌ 改完代码后 AwaitShell 数分钟「确认日志正常」再回复用户
❌ 任务已完成仍 sleep / 轮询无关后台进程
✅ 改完 → 简述结果与注意点 → 结束；需用户本地重启时直接说明
✅ 仅当下一步必须读命令输出时，短等一次；超时则说明现状并继续或收尾
```

---
> Source: [aiqachat/tkeapi](https://github.com/aiqachat/tkeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
