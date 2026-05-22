## nuva-evm-contracts

> This document outlines the architectural patterns, security standards, and operational workflows established for the Nuva Finance EVM contracts.

# Dedicated Vault Router & Nuva Vault

This document outlines the architectural patterns, security standards, and operational workflows established for the Nuva Finance EVM contracts.

## 🏗 Architecture & Design Patterns

### 1. Multi-Hop Deposit Flow
The `DedicatedVaultRouter` implements a three-layer staking flow:
*   **Hop 1**: Asset (e.g., USDC) -> `AssetVault` shares.
*   **Hop 2**: `AssetVault` shares -> `StakingVault` shares.
*   **Hop 3**: `StakingVault` shares -> `NuvaVault` shares.
*   **Mechanism**: Uses `forceApprove` and standard `IERC4626.deposit` calls. Signature-based AML verification is mandatory for every entry.

### 2. Asynchronous Redemptions (Disposable Proxies)
To isolate accounting and handle asynchronous lock periods, redemptions use EIP-1167 Minimal Proxy clones:
*   **Workflow**: User calls `requestRedeem` -> Router clones `RedemptionProxy` -> Proxy unwinds shares back to the underlying `AssetVault` and initiates a request.
*   **Cleaning**: Admin/Keeper sweeps assets from proxies back to users once the async period ends.

### 3. Permissions & Access Control
*   **Ownable2Step**: Used for "Heavy" administrative tasks (upgrades, updating AML signers, setting master proxy implementations).
*   **AccessControl**: Used for "Operational" tasks.
    *   `KEEPER_ROLE`: Authorized to call `sweepRedemptions`. This should be held by automated bot wallets.
    *   `DEFAULT_ADMIN_ROLE`: Held by the `Owner` to manage `KEEPER_ROLE` assignments.

### 4. NuvaVault (Production ERC4626)
*   **Upgradeability**: UUPS pattern.
*   **Inflation Protection**: Implements a `_decimalsOffset` of **12** to prevent share manipulation attacks.
*   **Pausability**: The `Owner` can pause deposits/withdrawals in emergencies.

## 🛠 Development Workflow

### Verification Mandate
After any contract modification, the following verification suite **MUST** be executed:

1.  **Linter**: Ensure NatSpec compliance and style.
    ```bash
    npm run lint:sol
    ```
2.  **Tests**: Confirm logic integrity.
    ```bash
    npx hardhat test test/DedicatedVaultRouter.test.js
    ```

### Professional Standards
*   **NatSpec**: All public/external functions, events, and state variables must have full `@notice`, `@param`, and `@return` tags.
*   **Specific Imports**: Avoid global imports. Use `{ Symbol } from "path"`.
*   **Storage Safety**: Maintain a `uint256[50] private __gap` (or adjusted for used slots) in all upgradeable contracts.
*   **Initialization**: Use `_unchained` initializers in the `initialize` function to prevent double-initialization in multiple inheritance.

---
> Source: [ProvLabs/nuva-evm-contracts](https://github.com/ProvLabs/nuva-evm-contracts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
