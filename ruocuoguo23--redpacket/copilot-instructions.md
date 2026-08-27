## redpacket

> 本文件适用于仓库根目录及所有子目录。参与开发的 Agent 和开发者在修改代码、脚本、测试或文档前，应先阅读本文件与 `docs/DESIGN.md`。

# RedPacket 项目协作指南

本文件适用于仓库根目录及所有子目录。参与开发的 Agent 和开发者在修改代码、脚本、测试或文档前，应先阅读本文件与 `docs/DESIGN.md`。

## 项目定位

RedPacket 是一个基于 Solidity 0.8.20 和 Foundry 的红包合约：

- 通过 UUPS + ERC1967 Proxy 支持升级；
- 支持原生 ETH 与标准 ERC20 资产；
- 支持固定金额与随机金额两种领取模式；
- 随机模式使用 Chainlink VRF v2.5；
- 使用后端签名授权领取，签名绑定领取人、红包 ID 和链 ID；
- 红包默认 24 小时过期，过期后剩余资产退回创建者。

“支持多链”是指合约可部署到兼容 Solidity 0.8.20 的 EVM 网络。随机模式只有在目标网络提供 Chainlink VRF v2.5，或存在兼容的 VRF Coordinator 时才能正常工作。每条链必须单独配置 Coordinator、Key Hash、Subscription ID、确认数和回调 Gas。

## 目录与职责

- `src/RedPacket.sol`：核心合约、资产托管、领取、退款、签名验证和 VRF 回调。
- `test/RedPacket.t.sol`：核心单元测试。
- `test/TestToken.sol`：测试 ERC20。
- `script/deploy/`：部署、升级和更新签名人脚本。
- `script/send_and_claim/`：创建、领取、退款和转账示例脚本。
- `docs/DESIGN.md`：架构、状态流、跨链配置、安全边界和演进方案。
- `README.md`：面向使用者的快速开始说明。

## 常用命令

首次拉取仓库后初始化依赖：

```bash
git submodule update --init --recursive
```

开发前后运行：

```bash
forge fmt --check
forge build
forge test -vvv
```

需要分析 Gas 时运行：

```bash
forge test --gas-report
```

不得提交 `.env`、私钥、助记词、RPC 密钥、签名服务密钥或生产 Subscription 管理凭据。

## 修改原则

1. 保持修改范围最小，不顺带重构无关模块。
2. 修改外部接口、事件、存储布局、签名格式或随机数流程时，必须同步更新测试、`README.md` 和 `docs/DESIGN.md`。
3. 新增外部函数应补充清晰的 NatSpec，错误条件优先使用自定义错误；同一改动内保持风格一致。
4. 状态更新遵循 Checks-Effects-Interactions。涉及 ETH、ERC20 或外部 Coordinator 调用时，必须重新评估重入风险。
5. 不得通过降低断言、删除失败用例或跳过测试来使 CI 通过。
6. 部署脚本必须从环境变量读取链相关配置，不得硬编码生产地址。

## 必须维护的业务不变量

- `packetId` 全局唯一，已存在的红包不得覆盖。
- 创建时 `totalPackets > 0`，且不超过管理员配置上限。
- 以资产最小单位计算时，`totalAmount >= totalPackets`，保证每份理论上至少为 1 个最小单位。
- 同一地址对同一红包最多成功发起一次领取。
- 领取总额、剩余金额与退款金额不能超过创建时实际托管的资产。
- 最后一份红包领取全部剩余金额，不产生合约内尘埃余额。
- 过期红包不能新发起领取；退款不能重复执行。
- 固定模式不应请求 VRF；随机模式的金额只能由已登记请求对应的 VRF 回调决定。
- 领取签名当前固定为：

  ```text
  keccak256(abi.encodePacked(claimer, packetId, block.chainid))
  ```

  然后按 EIP-191 `personal_sign` 格式恢复签名地址。除非有迁移方案，不得直接修改该格式。

## Chainlink VRF 约束

- 当前代码使用 VRF v2.5 的 `VRFConsumerBaseV2Upgradeable` 与 `IVRFCoordinatorV2Plus`。
- Proxy 地址必须加入对应 Subscription 的 Consumer 列表，Subscription 必须有足够余额。
- `extraArgs` 当前为空，表示使用默认的 LINK 支付方式；若改为原生币支付，必须同步更新链配置、脚本与测试。
- VRF 是异步流程。新增或修改逻辑时，必须同时考虑：并发请求预留份数、回调乱序、重复或未知请求、回调失败、过期退款与待处理请求竞争、请求元数据清理。
- 回调函数不得依赖调用者输入，不应因非关键业务逻辑耗尽 Gas；回调 Gas 上限变更必须通过测试验证。
- 不得假设不同链拥有相同的 Coordinator、Key Hash、费用模型或确认数。

## UUPS 升级约束

- 只能在现有存储变量末尾追加新变量；不得重排、删除或改变已有变量类型。
- 新版本初始化逻辑使用 `reinitializer(version)`，不得重复调用初始 `initialize`。
- `_authorizeUpgrade` 必须保持严格权限控制。生产环境建议由多签或带延迟的治理合约持有 Owner 权限。
- 升级测试至少覆盖：旧状态保留、Owner 不变、实现地址更新、非 Owner 升级失败、新增状态正确初始化。
- 外部用户始终与 Proxy 地址交互；不得把实现合约地址当作业务入口。

## 资产与签名安全

- 当前记账假设 ERC20 转入金额等于 `_totalAmount`。手续费型、通缩型、重基型、ERC777 或带回调 Token 未经专门适配与测试不得宣称支持。
- ETH 使用哨兵地址 `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`，不得与零地址混用。
- 初始化和管理员配置应拒绝零地址与不合理参数。
- 签名私钥属于高权限后端密钥，必须使用 KMS/HSM 或等价方案托管；日志中不得输出私钥或完整敏感凭据。
- 签名服务必须校验红包状态、领取地址、链 ID、限流与业务授权，不能仅作为无条件签名接口。

## 测试要求

每个行为改动都要覆盖成功路径、失败路径与边界条件。至少关注：

- ETH 和标准 ERC20 的创建、领取、全部领完与退款；
- 固定模式的整数除法与最后一份余数；
- 随机模式的最小金额、最大金额、最后一份、乱序回调和多请求并发；
- 无效签名、错误链 ID、重复领取、不存在的红包和过期红包；
- 非标准 Token、转账失败、恶意接收合约与重入尝试；
- Subscription 余额不足、回调失败以及待处理请求恢复；
- Owner 配置、权限边界与 UUPS 升级。

测试命名应说明场景和期望，例如 `test_RevertWhen_ClaimSignatureIsInvalid`。修复安全问题时应先添加能复现问题的回归测试。

## 完成标准

任务完成前确认：

- 格式检查、构建与相关测试通过；
- 新增逻辑没有破坏上述业务不变量；
- 存储布局与升级兼容性已检查；
- 多链参数没有被硬编码；
- 文档反映真实实现，不把计划能力描述成现有能力；
- 已说明无法执行的验证以及原因。

当前实现已知风险和建议改造顺序见 `docs/DESIGN.md` 的“安全分析与已知限制”。涉及生产部署前，必须完成独立安全审计。

---
> Source: [ruocuoguo23/redpacket](https://github.com/ruocuoguo23/redpacket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
