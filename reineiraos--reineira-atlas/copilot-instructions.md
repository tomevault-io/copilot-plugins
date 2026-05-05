## 02-protocol-policy

> Use when discussing insurance policies, risk scores, disputes, underwriter, coverage, pools, liquidity, LP staking, IUnderwriterPolicy, FHE encryption


# Protocol — Underwriter Policies

> **Read before acting:**
> - `.claude/docs/product/PROTOCOL_INTEGRATION.md`
> - `.claude/docs/strategy/TOKENOMICS.md`

Guide builders in designing IUnderwriterPolicy contracts that evaluate risk and judge disputes using FHE encryption. All return values MUST be FHE-encrypted.

## The Interface

```solidity
interface IUnderwriterPolicy {
    function onPolicySet(uint256 coverageId, bytes calldata data) external;
    function evaluateRisk(uint256 escrowId, bytes calldata riskProof)
        external returns (euint64 riskScore);
    function judge(uint256 coverageId, bytes calldata disputeProof)
        external returns (ebool valid);
}
```

## FHE Pattern (MUST Follow Exactly)

```solidity
import { FHE, euint64, ebool } from "@fhenixprotocol/cofhe-contracts/FHE.sol";

function evaluateRisk(uint256, bytes calldata) external returns (euint64) {
    uint64 score = 500; // 5% premium
    euint64 encrypted = FHE.asEuint64(score);
    FHE.allowThis(encrypted);
    FHE.allow(encrypted, msg.sender);
    return encrypted;
}
```

**Critical:** Without `FHE.allowThis()` + `FHE.allow()`, the value is unusable.

## Risk Score Scale

| Score (bps) | Premium | Use Case                           |
| ----------- | ------- | ---------------------------------- |
| 100         | 1%      | Low-risk, established counterparty |
| 300         | 3%      | Standard risk                      |
| 500         | 5%      | Elevated risk                      |
| 1000        | 10%     | High-risk, new counterparty        |

## Design Playbook

1. **Define risk factors** — what makes a transaction risky?
2. **Design scoring model** — how factors map to 0-10000 bps
3. **Define dispute evidence** — what proof validates a claim?
4. **Design judge logic** — how to evaluate dispute proof
5. **Set boundaries** — min/max premiums, dispute windows
6. **Consider pool economics** — premiums must exceed expected claims

## Checklist

- [ ] FHE pattern followed exactly (allowThis + allow)
- [ ] Risk score uses 0-10000 bps scale
- [ ] ERC-165 supportsInterface implemented
- [ ] Judge logic has clear evidence requirements
- [ ] Pool economics are sustainable

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
