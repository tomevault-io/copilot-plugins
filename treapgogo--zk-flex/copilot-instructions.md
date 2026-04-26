## zk-flex

> This codebase contains Scaffold-ETH 2 (SE-2), everything you need to build dApps on Ethereum. Its tech stack is NextJS, RainbowKit, Wagmi and Typescript. Supports Hardhat and Foundry.


This codebase contains Scaffold-ETH 2 (SE-2), everything you need to build dApps on Ethereum. Its tech stack is NextJS, RainbowKit, Wagmi and Typescript. Supports Hardhat and Foundry.

It's a yarn monorepo that contains following packages:

- FOUNDRY (`packages/foundry`): The solidity framework to write, test and deploy EVM Smart Contracts.
- NextJS (`packages/nextjs`): The UI framework extended with utilities to make interacting with Smart Contracts easy (using Next.js App Router, not Pages Router).


The usual dev flow is:

- Start SE-2 locally:
  - `yarn chain`: Starts a local blockchain network
  - `yarn deploy`: Deploys SE-2 default contract
  - `yarn start`: Starts the frontend
- Write a Smart Contract (modify the deployment script in `packages/foundry/script` if needed)
- Deploy it locally (`yarn deploy`)
- Go to the `http://locahost:3000/debug` page to interact with your contract with a nice UI
- Iterate until you get the functionality you want in your contract
- Write tests for the contract in `packages/foundry/test`
- Create your custom UI using all the SE-2 components, hooks, and utilities.
- Deploy your Smart Contrac to a live network
- Deploy your UI (`yarn vercel` or `yarn ipfs`)
  - You can tweak which network the frontend is pointing (and some other configurations) in `scaffold.config.ts`

## Smart Contract UI interactions guidelines
SE-2 provides a set of hooks that facilitates contract interactions from the UI. It reads the contract data from `deployedContracts.ts` and `externalContracts.ts`, located in `packages/nextjs/contracts`.

### Reading data from a contract
Use the `useScaffoldReadContract` (`packages/nextjs/hooks/scaffold-eth/useScaffoldReadContract.ts`) hook. 

Example:
```typescript
const { data: someData } = useScaffoldReadContract({
  contractName: "YourContract",
  functionName: "functionName",
  args: [arg1, arg2], // optional
});
```

### Writing data to a contract
Use the `useScaffoldWriteContract` (`packages/nextjs/hooks/scaffold-eth/useScaffoldWriteContract.ts`) hook.
1. Initilize the hook with just the contract name
2. Call the `writeContractAsync` function.

Example:
```typescript
const { writeContractAsync: writeYourContractAsync } = useScaffoldWriteContract(
  { contractName: "YourContract" }
);
// Usage (this will send a write transaction to the contract)
await writeContractAsync({
  functionName: "functionName",
  args: [arg1, arg2], // optional
  value: parseEther("0.1"), // optional, for payable functions
});
```

Never use any other patterns for contract interaction. The hooks are:
- useScaffoldReadContract (for reading)
- useScaffoldWriteContract (for writing)

### Reading events from a contract

Use the `useScaffoldEventHistory` (`packages/nextjs/hooks/scaffold-eth/useScaffoldEventHistory.ts`) hook.

Example:

```typescript
const {
  data: events,
  isLoading,
  error,
} = useScaffoldEventHistory({
  contractName: "YourContract",
  eventName: "GreetingChange",
  watch: true, // optional, if true, the hook will watch for new events
});
```

The `data` property consists of an array of events and can be displayed as:

```jsx
<div>
  {events?.map((event) => (
    <div key={event.logIndex}>
      <p>{event.args.greetingSetter}</p>
      <p>{event.args.newGreeting}</p>
      <p>{event.args.premium}</p>
      <p>{event.args.value}</p>
    </div>
  ))}
</div>
```

### Other Hooks
SE-2 also provides other hooks to interact with blockchain data: `useScaffoldWatchContractEvent`, `useScaffoldEventHistory`, `useDeployedContractInfo`, `useScaffoldContract`, `useTransactor`. They live under `packages/nextjs/hooks/scaffold-eth`.
## Display Components guidelines
SE-2 provides a set of pre-built React components for common Ethereum use cases: 
- `Address`: Always use this when displaying an ETH address
- `AddressInput`: Always use this when users need to input an ETH address
- `Balance`: Display the ETH/USDC balance of a given address
- `EtherInput`: An extended number input with ETH/USD conversion.

They live under `packages/nextjs/components/scaffold-eth`.

Find the relevant information from the documentation and the codebase. Think step by step before answering the question.

## 📚 核心文档参考规则

### llms-full-scaffold.txt - 必读文档
- **重要性**: `llms-full-scaffold.txt` 是本项目开发中**至关重要**的参考文档
- **使用原则**: 多阅读、勤阅读，在任何 Scaffold-ETH 2 相关的开发问题上优先参考此文档
- **避免幻觉**: 在回答问题前，务必查阅该文档确认准确性，避免编造不存在的 API 或用法
- **深度查询**: 如有必要，使用 DeepWiki MCP 工具查询 Scaffold-ETH 2 官方仓库 (https://github.com/scaffold-eth/scaffold-eth-2) 获取最新信息

示例用法:
```
# 查询 Scaffold-ETH 相关问题
使用 mcp_mcp-deepwiki_deepwiki_fetch 查询:
url: "scaffold-eth/scaffold-eth-2"
```

## 🎭 前端开发与调试规则

### Playwright MCP - 真实环境调试
- **调试策略**: 在进行前端开发和调试时，必须结合 Playwright MCP 进行**真实浏览器环境**的测试
- **质量标准**: 使用浏览器自动化工具进行"拳拳到肉"的调试，直到一切**看起来完美**
- **工作流程**:
  1. 启动开发服务器 (`yarn start`)
  2. 使用 `browser_navigate` 访问页面
  3. 使用 `browser_snapshot` 获取页面结构
  4. 使用 `browser_click`、`browser_type` 等工具模拟用户操作
  5. 使用 `browser_console_messages` 查看前端错误
  6. 使用 `browser_take_screenshot` 验证视觉效果
  7. 迭代修复直到功能完美

### 调试优先级
1. **先看文档** - 查阅 `llms-full-scaffold.txt`
2. **再写代码** - 基于文档编写符合规范的代码
3. **实时测试** - 用 Playwright 在浏览器中验证
4. **持续优化** - 根据实际表现迭代改进

**记住**: 不要仅凭想象或记忆编写代码，始终基于文档和实际测试结果!

## 📋 项目文档管理规则

### ROADMAP.md - 工程路线图（实时更新）

**核心定位**: ROADMAP.md 是工程开发的**唯一真相来源**（Single Source of Truth），用于进度追踪和团队协作。

#### 维护原则
1. **实时更新**: 每完成一个任务，立即更新对应的 checkbox
2. **状态同步**: 及时更新阶段状态（⏳待开始 → 🚧进行中 → ✅已完成）
3. **问题记录**: 遇到阻塞立即标记 ⚠️ 并注明原因和解决方案
4. **时间记录**: 在更新日志中记录每次重要变更的时间和内容
5. **防止丢失**: ⚠️ **每隔一段时间就更新日志**，不要等到会话末尾才更新（会话可能意外中断导致进度丢失）

#### 更新工作流
```markdown
每完成一个任务时：
1. 在 ROADMAP.md 中标记任务为完成 [x]
2. 更新进度追踪表格（已完成数、进度百分比）
3. 如果完成一个阶段，更新阶段状态标记
4. 在"更新日志"中添加一条记录（日期 + 完成内容）
5. 如遇问题，在对应任务下添加问题说明和解决方案

格式示例：
### 2025-10-18 14:30
- ✅ 完成 Phase 1.1 Rust 环境安装
- ✅ 完成 Phase 1.1 Circom 编译器安装
- ⚠️ 问题：Circom 编译时内存不足 → 解决：增加 swap 空间
- 🎯 下一步：开始编写 wealth_proof.circom 电路
```

#### 信息对齐策略
- **每个会话开始前**: 快速扫描 ROADMAP.md 确认当前进度
- **每次任务切换**: 检查 ROADMAP.md 确保方向正确
- **遇到困难时**: 在 ROADMAP.md 中记录问题和尝试的解决方案
- **长时间中断后**: 通过 ROADMAP.md 快速恢复上下文

#### 计划调整规则
当需要调整计划时：
1. **评估影响**: 判断变更对后续阶段的影响
2. **更新任务列表**: 添加/删除/修改任务项
3. **调整时间估算**: 更新各阶段预计耗时
4. **记录变更原因**: 在更新日志中说明为何调整
5. **保留历史**: 不删除已完成的记录，确保可追溯性

#### 为后续开发者/Agent 留下的关键信息
- ✅ **决策记录**: 为什么选择某个技术方案
- ✅ **坑点警告**: 遇到的问题和解决方法
- ✅ **依赖关系**: 任务之间的前置条件
- ✅ **性能数据**: Gas 消耗、电路约束数等关键指标
- ✅ **测试结果**: 测试通过情况和覆盖率

### PRODUCT.md - 产品规格（设计权威）

**核心定位**: PRODUCT.md 是产品设计的**权威参考文档**，定义"做什么"而非"怎么做"。

#### 使用原则
1. **设计决策依据**: 任何功能实现前，先查阅 PRODUCT.md 确认需求
2. **技术选型参考**: 核心设计决策表是技术实现的指导
3. **边界明确**: MVP 范围明确标注，避免过度设计
4. **不可轻易修改**: 如需修改产品设计，需充分评估影响

#### 与 ROADMAP.md 的关系
```
PRODUCT.md (What - 做什么)
    ↓ 指导
ROADMAP.md (How - 怎么做、何时做)
    ↓ 驱动
实际代码实现
```

#### 何时参考 PRODUCT.md
- ❓ 不确定某个功能是否在 MVP 范围内
- ❓ 需要了解系统架构和组件关系
- ❓ 需要确认合约接口和数据结构
- ❓ 需要了解 ZK 电路的约束逻辑
- ❓ 需要查看技术栈和依赖关系

#### 如何引用 PRODUCT.md
```markdown
# 在代码注释或讨论中引用
参考 PRODUCT.md 第 232 行：钱包池大小固定为 32

# 在 ROADMAP.md 中关联
根据 PRODUCT.md 的合约架构设计，需要实现...
```

### 文档协作最佳实践

#### 上下文管理策略
1. **分层信息密度**
   - 高频信息：ROADMAP.md 当前阶段（每次必看）
   - 中频信息：PRODUCT.md 相关章节（按需查阅）
   - 低频信息：详细设计文档（深入开发时参考）

2. **关键信息永久化**
   必须记录在文档中的信息：
   - ✅ 重大技术决策及原因
   - ✅ 已知问题和解决方案
   - ✅ 依赖版本和兼容性说明
   - ✅ 部署配置和环境要求
   - ✅ 测试结果和性能指标
   
   可以省略的信息：
   - ❌ 临时调试日志
   - ❌ 中间过程的尝试（除非有学习价值）
   - ❌ 显而易见的代码注释

3. **链接而非重复**
   ```markdown
   ✅ 好的做法：
   详见 PRODUCT.md 的"ZK 电路设计"章节
   
   ❌ 避免：
   在 ROADMAP.md 中重复粘贴整个电路设计
   ```

4. **版本标记**
   ```markdown
   在重大变更时更新版本号：
   > **版本**: v0.3  
   > **最后更新**: 2025-10-18  
   > **更新内容**: 添加 Phase 2 详细任务
   ```

### 工作会话开始时的检查清单

每次开始工作时，按顺序检查：
1. ✅ 读取 ROADMAP.md 的"当前项目状态"和"下一步行动"
2. ✅ 检查进度追踪表，确认当前所处阶段
3. ✅ 查看最近的更新日志，了解上次进展
4. ✅ 如有疑问，参考 PRODUCT.md 确认需求
5. ✅ 开始工作前，标记任务状态为 🚧 进行中

### 工作会话结束时的更新清单

每次结束工作时，必须完成：
1. ✅ 更新 ROADMAP.md 中完成的任务 checkbox
2. ✅ 在更新日志中添加本次进展记录
3. ✅ 如遇到问题，记录问题和解决方案（或标记为待解决）
4. ✅ 更新"下一步行动"指向下一个任务
5. ✅ 提交代码并确保文档与代码同步

**核心原则**: 
- 📝 **文档先行**: 先更新文档，再写代码
- 🔄 **及时同步**: 完成即更新，不要拖延
- 🎯 **信息密度**: 记录决策和结果，省略过程细节
- 🔗 **建立链接**: 文档间相互引用，避免重复
- 📊 **可视化进度**: 用表格、emoji、进度条让进度一目了然

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TreapGoGo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
