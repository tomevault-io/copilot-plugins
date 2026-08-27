## infrastructure

> 本文件只放**本仓可直接执行的命令**与**不知道就会当场写错的硬约束**。跨仓概念、基类能力细节、决策背景、

# CLAUDE.md

本文件只放**本仓可直接执行的命令**与**不知道就会当场写错的硬约束**。跨仓概念、基类能力细节、决策背景、
现状数字一律在统一知识库，不在这里重复（重复必漂移，`check-facts` 的 forbidden 断言会拦）。

> **统一知识库（SSOT）**：router [../workspace-knowledgebase/CLAUDE.md](../workspace-knowledgebase/CLAUDE.md) ·
> AI 加载协议 [../workspace-knowledgebase/AGENTS.md](../workspace-knowledgebase/AGENTS.md) ·
> 导航 [../workspace-knowledgebase/MAP.md](../workspace-knowledgebase/MAP.md) ·
> **本仓画像** [../workspace-knowledgebase/80-repos/infrastructure/index.md](../workspace-knowledgebase/80-repos/infrastructure/index.md) ·
> **基类能力**（Entropy / PrizePool / AccessControl / SalesVault / SelfPermit）
> [../workspace-knowledgebase/20-architecture/contract-capabilities.md](../workspace-knowledgebase/20-architecture/contract-capabilities.md)。

## 项目一句话

GreatLottoGroup 的**共享基础合约库**：GLC 稳定币锚定 + ERC4626 销售金库 + 渠道分润托管 + 6 个抽象基类。
Hardhat + Foundry + solc 0.8.36（viaIR，本仓不开 optimizer）。

> ⚠️ **本仓是接口契约源**：下游 ScratchCard / GreatLottoCore 经 `node_modules` symlink 直接消费本仓源码
> （不走 publish），改对外接口会立刻打穿它们。跨仓变更**必须本仓先定稿**。

## 常用命令

```bash
# ⚠️ 首次 / CI 先装 forge-std（lib/ 已 gitignore）
forge install foundry-rs/forge-std     # 断网回退：git clone --depth 1 https://github.com/foundry-rs/forge-std lib/forge-std

forge test                              # 全部（单测 + invariant）；= pnpm test
forge test --match-path test/foundry/PrizePoolBase.t.sol
forge test --gas-report                 # = pnpm gas
forge coverage --report summary         # = pnpm coverage（本仓无链上 SVG，无需 --ir-minimum）
pnpm test:scripts                       # skills/ 里纯函数的 node:test 单测

pnpm exec hardhat compile               # 编译 + 产出 ABI（供下游与 interface 消费）
pnpm exec hardhat clean

pnpm docs / pnpm docs:serve / pnpm docs:lint    # forge doc + check-natspec；本仓是基类文档权威源
```

### 本地一键部署 + 跨仓同步（skill `deploy-local-and-sync`，已软链到 `.claude/skills/`）

脚本本体在 [skills/deploy-local-and-sync/](skills/deploy-local-and-sync/)（唯一事实源；`scripts/` 只留运行期日志）。
命令均以 `infrastructure/` 为 cwd：

```bash
bash skills/deploy-local-and-sync/deploy-local.sh    # 清旧 → 起 node → 部署三仓 → 回填地址 → 同步 ABI
node skills/deploy-local-and-sync/sync-addresses.mjs --network <net> [--write] [--yes] [--only sc,core,interface]
node skills/deploy-local-and-sync/sync-abi.mjs [--network <net>] [--write]   # 前置：对应仓已 compile
```

加链 / 加合约 / 加 ABI 映射见该 skill 的 `SKILL.md`；流程见
[../workspace-knowledgebase/50-runbooks/local-deploy-and-sync.md](../workspace-knowledgebase/50-runbooks/local-deploy-and-sync.md)。

## 硬约束（不知道就会当场写错）

- **`contracts/test/` 只保留 `GreatLottoCoinTest`**，它被下游 ScratchCard（`InfraImports.sol`）与
  GreatLottoCore（`GreatLottoCoinTest2.sol`）**跨仓 import** → 是发布给下游的合约，**必须留在原路径**。
  删 / 迁 `contracts/test/*.sol` 前必须 grep 下游仓的 `@greatlotto/infrastructure/contracts/test/`。
- ⚠️ `ignition/modules/infrastructure.js` 当前部署的是 **`*Test` 版本**（`GreatLottoCoinTest`），
  **主网部署前必须切换为生产合约**。
- `GreatLottoCoin` 支持的稳定币由**构造参数**传入（`ignition/parameters/<network>.json` 的 `supportedTokens`；
  测试网 / 本地留空 `[]`），不再硬编码在源码里。Foundry 测试用 6 位 `MockERC20Permit` 显式传入。
- Foundry 测试约定：命名 `test_<func>_<场景>`；事件用接口限定 `emit IXxx.Event(...)`；带参 custom error 的
  `vm.expectRevert` **必须**给完整 `abi.encodeWithSelector(.., args)`；`PARTNER_CONTRACT_ROLE` 授给测试合约自身；
  GLC 余额用 `deal`。目录 `test/foundry/{base,mocks,harness,invariant}/`。
- NatSpec 正文**不能写 `@greatlotto/...`**（solc 当非法标签报错）；**不用 `@defi-wonderland/natspec-smells`**
  （它自行重编译且不开 viaIR，会 stack-too-deep）。

## 合约骨架

| 合约 | 符号 | 用途 |
|---|---|---|
| `GreatLottoCoin` | GLC | ERC20，与稳定币 1:1 锚定（当前仅 USDC，白名单可扩展）；`mint()` 仅 `PARTNER_CONTRACT_ROLE` |
| `SalesVault` | GLSV | ERC4626 销售利润金库；分润 `safeTransfer` 注入抬高 `totalAssets`（不动 `totalSupply`）使份额增值 |
| `SalesChannel` | — | 渠道注册表 + 分润托管账本（pull payment，渠道方自提） |

基类（`contracts/base/`）：`AccessControlPartnerContract` · `EntropyConsumerBase` · `PrizePoolBase` ·
`NoDelegateCall` · `DeadLine` · `SelfPermit`。**各基类的能力、构造签名、治理 setter、分润 pipeline 细节
见 KB 的 contract-capabilities.md**（唯一权威，别在本文件重述）。

## 部署

命令模式与三个坑（IGN1002 / apiKey 必须单串 / OP 系 IGN410）见
[../workspace-knowledgebase/50-runbooks/deploy-verify.md](../workspace-knowledgebase/50-runbooks/deploy-verify.md)。
参数走 `ignition/parameters/<network>.json`（`owner` / `supportedTokens` 不读 `.env`）。

```bash
pnpm exec hardhat ignition deploy ignition/modules/infrastructure.js \
  --network <localhost|baseSepolia|arbitrumSepolia|optimismSepolia|unichainSepolia|base|arbitrum|optimism|unichain> \
  --parameters ignition/parameters/<network>.json --reset --verify
```

### `.env`

`ALCHEMY_API_KEY`（所有 RPC 与默认 fork）· `DEPLOY_ACCOUNT_PRIVATE_KEY` ·
`ETHERSCAN_API_KEY`（Etherscan V2 单串 key 通吃四链；旧的 `BASESCAN_API_KEY` / `ARBISCAN_API_KEY` 已停用）。

## 流程（三道门 + 归档）

`brainstorming` → `openspec-propose` → `writing-plans`（产物落 `openspec/changes/<id>/`）→
**`/flow-review-spec`（方案门）** → 实现 → **`/flow-ship`（实现绿 + 代码门 + 安全门，本仓 `/security-review` 必跑）**
→ **`/flow-archive`（KB 沉淀 → `openspec archive --yes` → 收口）**。

- **怎么操作**（命令 / 门产物模板 / 报错→怎么修）：[../workspace-knowledgebase/40-workflows/how-to.md](../workspace-knowledgebase/40-workflows/how-to.md)
- 门的判定口径与 loop 终止条件：[../workspace-knowledgebase/40-workflows/review-gates.md](../workspace-knowledgebase/40-workflows/review-gates.md)
- **本仓改接口 = 跨仓变更**：按 [../workspace-knowledgebase/40-workflows/cross-repo-protocol.md](../workspace-knowledgebase/40-workflows/cross-repo-protocol.md) 建协调文档、本仓先定稿。
- 改了合约却没建 change 时 `commit-msg` 钩子会提示（非阻断；无行为变化在 message 里写 `[exempt]`）；归档一律
  `openspec archive <id> --yes`，不要手动 `mv`。
- **只在明确要求时 commit。**

---
> Source: [GreatLottoGroup/infrastructure](https://github.com/GreatLottoGroup/infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
