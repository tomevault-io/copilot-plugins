## uart-dmac

> **Skill（强制）：** `r2-verilator-ip-delivery` v1.2+

# uart_dmac Agent 规则

**Skill（强制）：** `r2-verilator-ip-delivery` v1.2+  
路径：`~/.grok/skills/r2-verilator-ip-delivery/SKILL.md`

## 0. 动刀前必做（不可跳过）

每一刀开始前，**先读 skill 对应门的退出条件**，再写 `docs/CURRENT_TASK.md` 的 Exactly/Does-not-count/Checks/Tag，**用户确认或 task 已 ACTIVE 后才改 RTL**。

每步汇报必须显式写出：

```
【门】GateN / Gate3-slice-<name>
【退出条件】skill 原文要点（1–3 条）
【本步动作】只做其中哪一条
【自证命令】make lint | golden | sim | sim-seeds | cov …
【留痕】commit + tag 名（绿后立即做；未绿禁止 tag 覆盖）
【未满足则停】不进入下一门/下一刀
```

## 1. 门序（禁止跳门）

```
Gate0 脚手架 → Gate1 契约 → Gate2 空壳 → Gate3 红绿切片… → Gate4 G0–G3 签核
```

| 门 | 退出条件（skill） | 本仓状态 |
|---|---|---|
| Gate0 | scaffold + git + make 可跑 | 已过（脚手架在） |
| Gate1 | lint 绿；ICD 无 TBD 关口；waiver 已审 | 部分过（ICD 薄更新；无独立 `gate1-icd` tag） |
| Gate2 | shell case 绿；case_list；接口零漂移 | 部分过（单 harness；无独立 `gate2-shell` tag） |
| Gate3 | 红→绿→重构；sim + sim-seeds；每切片 git tag | **当前所在**；切片至 `v1.8-dma-pack-circ-tor` |
| Gate4 | G0 lint / G1 回归 / G2 cov / G3 互补 + evidence | **未开** |

## 2. Gate3 单刀协议

1. **Oracle/用例先红**（会失败的 TB 或明确 AC 断言）  
2. **最小 RTL** 使该 AC 绿  
3. **全回归**：`make lint && make golden && make sim && make sim-seeds`  
4. **evidence**（hashes / 日志，建议）  
5. **`git commit` + annotated `git tag`** — 未留痕 = 本刀未完成  
6. 半成品只在 `wip/*`；**main 只收绿点**

## 3. 基线与禁止

- 可回退 tag：`v1.7`、`v1.8-dma-pack-circ-tor`  
- 禁止：改 shell 刷绿；无 Oracle 连改 RTL；半成品盖绿点；宣称 Verilator 绿 = 流片签核  
- 禁止：未写 CURRENT_TASK Exactly 就开新功能  

## 4. 下一刀入口

1. 选定切片名（例：`slice-reg-map`）  
2. 写满 `docs/CURRENT_TASK.md` §2（对照 skill Gate3 退出条件）  
3. 汇报【门】块 → 等确认或按已确认 task 执行  
4. 动刀后每微步仍复述退出条件与自证命令  

---
> Source: [alicegaoicgeek/uart-dmac](https://github.com/alicegaoicgeek/uart-dmac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
