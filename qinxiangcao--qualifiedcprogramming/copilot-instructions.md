## qualifiedcprogramming

> 本仓库用于验证 C 程序。默认方法是先验证抽象 monad 程序的正确性，再验证带 annotation 的 C 程序通过 `safeExec` refine 该抽象程序。

# QCP C Program Verification Guide

本仓库用于验证 C 程序。默认方法是先验证抽象 monad 程序的正确性，再验证带 annotation 的 C 程序通过 `safeExec` refine 该抽象程序。

## Global Rules

- 只参考 `LLM_friendly_cases` 版本的示例，不参考 `QCP_democases` 版本。
- 必须优先使用 `qcp-mcp` 和 `rocq-mcp` 进行交互式检查。
- 遇到 manual VC 时，先判断该 VC 是否语义可证；如果不可证，回到 C annotation 修正，不要硬写 Rocq proof。
- 证明风格必须模仿 `LLM_friendly_cases` 中已经完成的 proof，优先使用项目提供的 tactics，不要把证明全部展开成低层手工证明。
- 不修改生成的 `*_goal.v`、`*_proof_auto.v`、`*_goal_check.v`。
- 只修改 C annotation、对应的 `*_proof_manual.v`、以及必要的 `*_lib.v` 辅助定义或引理。
- 不引入 `Axiom`，不留下 `Admitted`。
- 如果修改了 annotation，必须重新运行 symbolic execution 生成最新 VC，再继续证明。
- 每次重新运行 symbolic execution 或重新生成 Rocq VC 后，都必须重新检查完整 witness 列表；不要假设旧 VC 编号、内容或旧证明仍然匹配。
- 每次 symbolic execution 成功后，**必须**先逐条分析返回结果中的所有 `manual_witness` goal，判断其语义上是否可证明；确认可证明后才使用 `qcp-mcp proof` 导出并进入 Rocq 证明。
- 只人工处理 manual VC；auto-solved 的 VC 不需要导出、证明或回填。
- 处理 manual VC 时，必须先用 `qcp-mcp proof` 按“导出一个、检查一个、交互式证明一个、保存一个”的节奏推进，不要一次性批量导出后跳着证明。
- 复用旧证明前必须确认当前 VC 和旧 VC 相同或仅有无关变量名变化；如果 VC 有实质变化，必须重新用 `rocq-mcp` 交互式跑通证明。
- 所有 `qcp-mcp proof` 导出的 manual VC 都在临时证明文件中跑通后，才运行 symexec 生成或刷新正式 `*_goal.v`、`*_proof_auto.v`、`*_proof_manual.v`、`*_goal_check.v`。
- 最终完成标准是 symbolic execution 成功、所有 manual VC 完成、对应 `*_goal_check.v` 编译通过。
- 禁止使用`entailer!`
- 仅参考查看当前 `qcp-binary-democases/` 下的文件。
- `*_lib.v` 文件的证明也请你使用 rocq-mcp ,特别是出现编译不通过的时候，一定要先用rocq-mcp跑通，再证明，所有的rocq证明都要遵守这个规定。
- `*_lib.v` 请按需添加，需要使用相关引理再添加，不可以一下子生产一堆引理和证明。

## Verification Workflow

1. 确认目标 C 文件、对应 Rocq 输出目录、以及要验证的函数。
2. 在 `QCP_examples/LLM_friendly_cases` 和 `SeparationLogic/examples/LLM_friendly_cases` 中查找相似案例。
3. 如果是 refinement 验证，先查看对应抽象 monad 程序、`safeExec` 相关 specification、以及抽象程序的 Hoare correctness theorem。
4. 使用 `annotation-and-symbolic-execution` skill 补充 annotation，并用 `qcp-mcp symbolic` 检查到目标 C 文件尾部。
5. 重新检查 symbolic 输出的完整 witness 列表，确认 manual/auto-solved 分类；auto-solved VC 不再人工处理。
6. 逐条阅读 symbolic 返回的所有 `manual_witness` goal，做自然语言语义可证性分析；若某个 goal 语义上不可证，先修 annotation、spec、invariant、`safeExec` 前条件或 `*_lib.v` 定义。
7. 使用 `vc-proving` skill 处理 symbolic 输出中的 manual VC。每个函数的 VC 使用同一流程处理，不因函数不同而跳过导出、比较或交互式证明步骤。
8. 对 manual VC 逐个处理：用 `qcp-mcp proof` 导出当前 VC 到临时文件，和已有证明对应的旧 VC 比较；未变化则可复用旧证明，变化则先在 `rocq-mcp` 中交互式跑通证明，并保存到 `SeparationLogic/tmp_vc/`。
9. 所有 `qcp-mcp proof` 导出的 manual VC 都在临时证明文件中跑通后，再运行 symexec 二进制完整生成或刷新 `*_goal.v`、`*_proof_auto.v`、`*_proof_manual.v`、`*_goal_check.v`。
10. 按 VC 主体形状而不是仅按编号，把已跑通的临时 proof 回填到 `*_proof_manual.v`。
11. 编译对应 `*_goal_check.v`，确认没有遗漏的 VC。
12. 清除 `*_lib.v` 中没有用到的多余引理和定义（多余定义如果在C中import了也同样清除）

## Skill Routing

- 端到端验证 C 程序：使用 `C-code-verification`。
- 编写或修正 C annotation、运行 `qcp-mcp symbolic`、定位 symbolic execution 错误、在 manual VC 临时证明全部通过后完整运行 symexec：使用 `annotation-and-symbolic-execution`。
- 检查 VC 是否满足、定位问题属于 annotation 还是 manual proof：使用 `vc-checking`。
- 证明 manual VC、选择 separation logic tactics、处理 `safeExec` / refinement proof：使用 `vc-proving`。

## Reference Examples

普通 separation logic VC proof 优先参考：

- `SeparationLogic/examples/LLM_friendly_cases/sum_proof_manual.v`
- `SeparationLogic/examples/LLM_friendly_cases/sll_proof_manual.v`
- `SeparationLogic/examples/LLM_friendly_cases/functional_queue_proof_manual.v`

refinement / `safeExec` proof 优先参考：

- `SeparationLogic/examples/LLM_friendly_cases/sll_merge_rel_proof_manual.v`
- `SeparationLogic/examples/LLM_friendly_cases/kmp_rel_proof_manual.v`

annotation 形状可以参考但不要模仿其 unfinished proof：

- `QCP_examples/LLM_friendly_cases/int_array_merge_rel.c`
- `QCP_examples/LLM_friendly_cases/eval.c`

---
> Source: [QinxiangCao/QualifiedCProgramming](https://github.com/QinxiangCao/QualifiedCProgramming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
