## effectstream

> └── multi-chain-token-transfer/

# Effectstream: Multi-chain Token Transfer Template

## Project Structure

```
/
└── templates/
    └── multi-chain-token-transfer/
        └── packages/
            ├── client/
            │   ├── batcher/                                # "@multi-chain-transfer/batcher"
            │   ├── database/                               # "@multi-chain-transfer/database"
            │   └── node/                                   # "@multi-chain-transfer/node"
            ├── frontend/                                   # "@multi-chain-transfer/frontend"
            └── shared/
                ├── contracts/
                │   ├── evm/                                # "@multi-chain-transfer/evm-contracts"
                │   └── midnight/                           # "@multi-chain-transfer/midnight-contracts"
                │       └── contract-eip-1155/              # "@multi-chain-transfer/midnight-contract-eip-1155"
                ├── custom-primitive-mct-erc1155/           # "@multi-chain-transfer/custom-primitive-mct-erc1155"
                └── data-types/                             # "@multi-chain-transfer/data-types"
```

---
> Source: [effectstream/effectstream](https://github.com/effectstream/effectstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
