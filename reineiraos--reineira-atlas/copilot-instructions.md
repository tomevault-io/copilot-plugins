## 01-protocol-resolver

> Use when discussing condition resolvers, escrow release logic, isConditionMet, onConditionSet, zkTLS, Reclaim proofs, oracles, Chainlink, UMA, verification sources


# Protocol — Condition Resolvers

> **Read before acting:** `.claude/docs/product/PROTOCOL_INTEGRATION.md`

Guide builders in designing IConditionResolver contracts that control when escrows release funds.

## The Interface

```solidity
interface IConditionResolver {
    function isConditionMet(uint256 escrowId) external view returns (bool);
    function onConditionSet(uint256 escrowId, bytes calldata data) external;
}
```

## Verification Sources

| Source            | Best For                                   |
| ----------------- | ------------------------------------------ |
| **Reclaim zkTLS** | Payment proofs (PayPal, Stripe, bank APIs) |
| **Chainlink**     | Price thresholds, market data              |
| **UMA Oracle**    | Binary/numeric outcome resolution          |
| **Multi-sig**     | N-of-M human approval                      |
| **Time lock**     | Deadline-based release                     |

## Design Playbook

1. **Identify the condition** — what must be true for funds to release?
2. **Select verification source** — which of the above fits?
3. **Design the storage** — what does onConditionSet need to store?
4. **Define the check** — what does isConditionMet evaluate?
5. **Security review** — replay protection, input validation, gas bounds
6. **Spec the output** — resolver name, struct layout, data encoding

## Storage Pattern

```solidity
struct Config { /* fields from resolverData */ }
mapping(uint256 => Config) public configs;
mapping(uint256 => bool) public fulfilled;
mapping(bytes32 => bool) public usedProofs;

function onConditionSet(uint256 escrowId, bytes calldata data) external {
    Config memory config = abi.decode(data, (Config));
    configs[escrowId] = config;
}

function isConditionMet(uint256 escrowId) external view returns (bool) {
    return fulfilled[escrowId];
}
```

## Security Checklist

- [ ] `isConditionMet` is `view` — no state changes
- [ ] `isConditionMet` gas < 50k — no unbounded loops
- [ ] `onConditionSet` validates all inputs
- [ ] Replay protection: `usedProofs[keccak256(proof)]`
- [ ] One escrow ID → one condition state (no overwriting)
- [ ] External calls use known addresses, not user-supplied
- [ ] Oracle data freshness validated
- [ ] ERC-165 `supportsInterface` implemented
- [ ] Events emitted on state changes

## Implementation Path

1. Design spec in Atlas
2. Implement in `reineira-code/contracts/resolvers/`
3. Test in `reineira-code/test/resolvers/`
4. Deploy: `npx hardhat run scripts/deploy.ts --network arbitrumSepolia`
5. Attach: `sdk.escrow.build().condition(resolverAddress, resolverData)`

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
