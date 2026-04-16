## blackswan-contract

> Use Foundry. Keep imports compatible with the specified Aave libs.


## 1) Repository and Directory Layout

Use Foundry. Keep imports compatible with the specified Aave libs.

```
root/
  foundry.toml
  remappings.txt
  .github/workflows/ci.yml
  .prettierrc
  .solhint.json
  .gitignore
  README.md

  /contracts
    /core
      Core.sol
      CoreTypes.sol
    /storage
      PoolStorage.sol
      StorageCodec.sol
      SeriesStorage.sol
      QueueStorage.sol            // optional minimal queue state (or embed in PoolStorage)
    /interfaces
      IPool.sol
      IPoolAdmin.sol
      IOracle.sol
      ICToken.sol
      ILPToken.sol
    /tokens
      CToken1155.sol
      LPToken.sol
    /pool
      Pool.sol
      PoolAdmin.sol               // governor/ops functions
      PoolFactory.sol             // instantiate pools/products if we later need >1 product
    /oracle
      OracleAdapter.sol           // wraps external truth source to IOracle
      ProportionalModel.sol       // optional g(data)→ratio provider interface + mock
    /libs
      TokenIdLib.sol
      AccountingLib.sol           // non-core helpers that are not math (e.g. event checks)
      RBCLib.sol                  // RBC enforcement helpers (pure/view)
      SafeTransferLib.sol         // thin wrapper over OZ SafeERC20 for brevity
      Errors.sol
      Events.sol

  /script
    Deploy.s.sol
    CreateSeries.s.sol
    Seed.s.sol

  /test
    /unit
      Core_Quote.t.sol
      Pool_Buy.t.sol
      Pool_Settle.t.sol
      Pool_Mature.t.sol
      LP_DepositWithdraw.t.sol
      CT_Redeem.t.sol
      RBC_Guards.t.sol
      Pause_Cooldown.t.sol
      Activation_Gating.t.sol
      Decimals_6_18.t.sol
    /property
      Invariants.t.sol            // forge-std invariant framework
    /mocks
      MockERC20.sol
      MockOracle.sol
      MockProportionalModel.sol
      MockFailingERC20.sol        // to test transfer failures
```

---

## 2) External Dependencies and Remappings

* **Compiler**: Solidity 0.8.24 (same as Core). Enable via `foundry.toml`.
* **Libraries** (as requested in Core):

  ```solidity
  import {WadRayMath}     from "@aave/protocol/libraries/math/WadRayMath.sol";
  import {PercentageMath} from "@aave/protocol/libraries/math/PercentageMath.sol";
  import {MathUtils}      from "@aave/protocol/libraries/math/MathUtils.sol";
  import {Math}           from "@openzeppelin/contracts/utils/math/Math.sol";
  ```
* Also use:

  ```solidity
  import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
  import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
  import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
  import {ERC1155} from "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
  import {Pausable} from "@openzeppelin/contracts/security/Pausable.sol";
  import {ReentrancyGuard} from "@openzeppelin/contracts/security/ReentrancyGuard.sol";
  import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";
  ```
* `remappings.txt` example:

  ```
  @openzeppelin/=lib/openzeppelin-contracts/
  @aave/=lib/aave-v3-core/    // or the actual monorepo path that contains protocol/libraries/math/*
  ```

---

## 3) Contracts to Implement

### 3.1 Interfaces (`/contracts/interfaces`)

Implement the following minimal external APIs (function sets match previous design):

**`IPool.sol`**

```solidity
interface IPool {
  // Views
  function product() external view returns (PoolStorage.ProductConfig memory);
  function getSeries(bytes32 seriesId) external view returns (PoolStorage.SeriesState memory);
  function previewQuote(uint256 tenorDays) external view returns (uint256 pTWad, uint256 uWad);
  function utilization() external view returns (uint256 uWad);
  function nav() external view returns (uint256);
  function freeCapital() external view returns (uint256);
  function maxLiability() external view returns (uint256);

  // Primary market
  function buy(bytes32 seriesId, uint256 notional, address receiver)
    external returns (uint256 tokenId, uint256 premiumPaid);

  // Settlement / Maturity
  function settle(bytes32 seriesId) external returns (uint256 payoutRatioWad, uint256 payTotal);
  function mature(bytes32 seriesId) external returns (uint256 unlocked);
  function redeem(uint256 tokenId, uint256 amount) external returns (uint256 paid);

  // LP
  function deposit(uint256 amount, address receiver) external returns (uint256 shares);
  function withdraw(uint256 shares, address receiver) external returns (uint256 amountPaid);
}
```

**`IPoolAdmin.sol`**

```solidity
interface IPoolAdmin {
  function pause() external;
  function unpause() external;
  function setCooldown(uint64 untilTs) external;
  function setFees(uint256 feePremWad, uint256 feeMgmtAprWad) external;
  function setOracle(address oracle) external;
  function setRBCParams(bytes32 bucket, uint256 rhoWad, uint256 bufferWad) external;
  function createSeries(uint64 maturity, uint256 rMaxWad) external returns (bytes32 seriesId);
}
```

**`IOracle.sol`**

```solidity
interface IOracle {
  function eventConfirmed(bytes32 seriesId) external view returns (bool ok);
  function payoutRatio(bytes32 seriesId) external view returns (uint256 rWad, uint64 eventTime);
}
```

**`ICToken.sol`** (ERC1155 extension)

```solidity
interface ICToken {
  function mint(address to, uint256 id, uint256 amount) external;
  function burnFrom(address from, uint256 id, uint256 amount) external;
  function seriesOf(uint256 tokenId) external view returns (bytes32);
  function activationStartOf(uint256 tokenId) external view returns (uint64);
}
```

**`ILPToken.sol`** (ERC20 extension)

```solidity
interface ILPToken {
  function mint(address to, uint256 amount) external;
  function burn(address from, uint256 amount) external;
}
```

Add event and error definitions in a shared `Events.sol` and `Errors.sol` library for consistency.

---

### 3.2 Tokens (`/contracts/tokens`)

**`CToken1155.sol`**

* ERC1155 with:

  * `onlyPool` minter/burner
  * Metadata storage:

    * `mapping(uint256 => bytes32) private _seriesOf;`
    * `mapping(uint256 => uint64)  private _activationStart;`
  * `function mint(address to, uint256 id, uint256 amount) external onlyPool`
  * `function burnFrom(address from, uint256 id, uint256 amount) external onlyPool`
  * `function seriesOf(uint256 id) external view returns (bytes32)`
  * `function activationStartOf(uint256 id) external view returns (uint64)`
* No enumerable requirement. Redemption is pull‑based via Pool.
* Token ID is built off `TokenIdLib.ctTokenId(seriesId, activationStart)`.

**`LPToken.sol`**

* Plain ERC20 with 18 decimals.
* `onlyPool` mint/burn. No fee logic.
* Optional: EIP‑2612 permit for UX.

---

### 3.3 Libraries (`/contracts/libs`)

**`TokenIdLib.sol`** (as designed)

```solidity
library TokenIdLib {
  function seriesId(bytes32 productId, uint64 maturity) internal pure returns (bytes32) {
    return keccak256(abi.encodePacked(productId, maturity));
  }
  function ctTokenId(bytes32 seriesId, uint64 activationStart) internal pure returns (uint256) {
    return uint256(keccak256(abi.encodePacked(seriesId, activationStart)));
  }
}
```

**`RBCLib.sol`**

* `function requiredCapital(PoolStorage.RBC storage rbc) internal view returns (uint256)`
  Sum `rhoWad[bucket]*L_bucket / 1e18`, then multiply by `bufferWad`. Returns asset units by design: we already store `L_bucket` in asset units, `rhoWad` and `bufferWad` are WAD.
* View helper `checkSolvency(PoolStorage.Layout storage L) returns (bool ok)` comparing `cTotal` vs required capital.

**`AccountingLib.sol`**

* Thin helpers for NAV and U, just wrappers around Storage values where needed.
* Do not replicate Core math.

**`SafeTransferLib.sol`**

* Wrap OZ `SafeERC20` to reduce repetitive boilerplate in `Pool.sol`.

**`Errors.sol`**

* Mirror CoreTypes errors for Pool‑level reverts:

  * `error OnlyPool(); error OnlyGovernor(); error Cooldown(); error NotConfirmed(); error NotEligible(uint64 activationStart, uint64 eventTime); error ZeroAmount(); error BadOracle();` etc.

**`Events.sol`**

* Standardize:

  * `event Bought(address indexed buyer, bytes32 indexed seriesId, uint256 tokenId, uint256 notional, uint256 premium, uint256 uNewWad);`
  * `event Settled(bytes32 indexed seriesId, uint256 payoutRatioWad, uint256 payTotal, uint64 eventTime);`
  * `event Matured(bytes32 indexed seriesId, uint256 unlocked);`
  * `event Redeemed(address indexed account, uint256 indexed tokenId, uint256 amount, uint256 paid);`
  * `event Deposit(address indexed lp, uint256 amount, uint256 shares);`
  * `event Withdraw(address indexed lp, uint256 shares, uint256 amount, bool queued);`
  * `event Paused(); event Unpaused();`

---

### 3.4 Pool Orchestrator (`/contracts/pool/Pool.sol`)

Responsibilities:

* Gatekeeping (pause, cooldown, RBC checks)
* Snapshot storage → call `Core` → apply deltas with `StorageCodec`
* Execute side‑effects (token transfers, mint/burn) following CEI pattern
* Enforce invariant ordering

Key fields:

* Immutable addresses: asset (ERC20), CT token, LP token, treasury, oracle
* Roles: `GOVERNOR_ROLE`, `OPERATOR_ROLE`
* Reentrancy guard, pausable (mirror Storage.paused), but use Storage as the single source of truth for pause state to keep views consistent

Key functions:

**Views**

* `product()` returns `PoolStorage.ProductConfig`
* `getSeries(seriesId)` returns current `SeriesState`
* `previewQuote(tenorDays)` calls Core.quote(snapshot) and returns `(pTWad, uWad)`
* `utilization()`, `nav()`, `freeCapital()`, `maxLiability()`

**Primary**

* `buy(seriesId, notional, receiver)`

  1. `require(block.timestamp >= L.cooldownUntil)` to avoid selling in cooldown
  2. Build snapshots: `poolBytes`, `seriesBytes`, `productBytes`
  3. Call `Core.buy(...)`
  4. Effects:

     * Transfer `premium` from `msg.sender` to Pool
     * Transfer `feeToTreasury` to treasury
     * Apply deltas: `applyPoolDelta`, `applySeriesDelta`
     * Mint CT:

       * `tokenId = TokenIdLib.ctTokenId(seriesId, activationStart)`
       * `CToken1155.mint(receiver, tokenId, notional)`
       * Save mapping `seriesOf[tokenId]` and `activationStartOf[tokenId]` is handled inside token
  5. Emit `Bought`
  6. Return `(tokenId, premium)`

**Settlement**

* `settle(seriesId)`

  1. `require(oracle.eventConfirmed(seriesId))`
  2. `(r, eventTime) = oracle.payoutRatio(seriesId)`
  3. Build snapshots; call `Core.settle(...)` with `nowTs=block.timestamp`, `eventTime`, `r`
  4. Apply deltas (this sets `paused = true`, `cooldownUntil`)
  5. Emit `Settled`
  6. Return `(r, payTotal)`

**Maturity**

* `mature(seriesId)`

  1. Require `block.timestamp >= maturity`
  2. Call `Core.mature(...)`
  3. Apply deltas
  4. Emit `Matured`

**Redeem**

* `redeem(tokenId, amount)`

  1. Resolve `seriesId` and `activationStart` via CT
  2. Load series; ensure `series.status == Settled`
  3. Ensure `activationStart <= series.eventTime` (eligibility)
  4. Compute `pay = amount * series.payoutRatioWad / WAD` (downward rounding)
  5. Burn CT then transfer `pay` to `msg.sender` (pull model)
  6. Emit `Redeemed`

**LP**

* `deposit(amount, receiver)`

  1. Require `amount > 0`
  2. Snapshot pool; call `Core.deposit(...)`
  3. Transfer in asset `amount`
  4. Apply pool delta
  5. Mint LP `mintShares` to `receiver`
  6. Emit `Deposit`

* `withdraw(shares, receiver)`

  1. Require `shares > 0`
  2. Snapshot; call `Core.withdraw(...)` with inputs `(shares, sTotal, nav, cFree)`
  3. Burn `burnShares` from `msg.sender`
  4. Apply pool delta
  5. Transfer `payAmount` to `receiver`
  6. If `queued`, enqueue the remainder in WithdrawQueue (see 3.5)
  7. Emit `Withdraw`

**Guards**

* Before selling (`buy`) also enforce RBC if enabled:

  * `require(RBCLib.checkSolvency(L))` prior to opening sales; otherwise pause.

**Admin**

* Expose admin actions in `PoolAdmin.sol` that mutate Storage fields (fees, oracle, RBC params, createSeries). Restrict with AccessControl.

---

### 3.5 Withdraw Queue

Implement a minimal FIFO queue library or inline in the Pool:

* Storage:

  * `struct Request { address lp; uint256 shares; uint256 navAmount; }`
  * `mapping(uint256 => Request) q; uint256 head; uint256 tail;`
* On `withdraw` when `queued == true`:

  * Store `(lp, remainingShares, remainingNAV)` after partial immediate payment and burn only the portion corresponding to paid amount. Alternatively, burn full `shares` upfront and mint an IOU ERC1155 representing queue position. MVP: keep simple and burn only the paid‑corresponding portion as Core suggested; enqueue unpaid portion in NAV terms and re‑price at settlement with a fairness rule documented.
* `settleQueue()` callable by anyone:

  * While `cFree > 0` and queue not empty, pay out requests FIFO up to `cFree`.
  * Emit an event per filled item.

Document how partial fills map to LP shares. MVP: keep invariant “shares burned equals NAV paid / current p” to avoid drift.

---

### 3.6 RBC (Correlation/Capital Buffer)

* Store per‑bucket `rhoWad` and aggregate `L_by_bucket` in `PoolStorage.RBC`.
* Update `L_by_bucket` only when applying `SeriesDelta.dCLocked`.
* `requiredCapital = sum_b( rho[b] * L_b ) * bufferWad / 1e36` (two WAD multiplications).
* If `cTotal < requiredCapital`, enforce:

  * pause sales and/or
  * strengthen withdraw behavior (queue everything)
* Expose read views for monitoring.

---

### 3.7 Oracle Adapter

`OracleAdapter.sol` conforms to `IOracle` and shields Pool from external feeds:

* For binary payouts, `payoutRatio` returns `(1e18, eventTime)` when condition is met.
* For proportional/capped, read external model `ProportionalModel.g(seriesId)` and cap by `rMax`.
* Trust boundaries:

  * Validate `eventTime != 0` and not in the future
  * Ensure monotonicity per series (only one settlement)
* Include a mock for tests.

---

### 3.8 Factory (optional for MVP)

If we foresee multiple products/pools:

* `PoolFactory.sol` that deploys a new Pool with product config and connects CT/LP tokens.
* Otherwise, MVP can deploy a single Pool via scripts.

---

## 4) Coding Standards

* Solidity: 0.8.24, SPDX identifiers, NatSpec for external/public functions.
* CEI pattern strictly in `Pool.sol`:

  * Checks: call Core (pure), validate
  * Effects: write Storage via `StorageCodec.apply*Delta`
  * Interactions: token transfers last (except `buy`, where premium must be transferred before mint to keep accounting consistent; still do it safely and never transfer after external mint calls)
* Use `custom errors` not revert strings.
* Use OZ `SafeERC20` and check return values.
* Rounding:

  * Prices/ratios: round down
  * Capacity (`U_new`) and locks: round up
  * Match Core exactly; do not re‑implement math in Pool.

---

## 5) Build & Tooling

**foundry.toml**

```toml
[profile.default]
solc_version = "0.8.24"
optimizer = true
optimizer_runs = 5000
libs = ["lib"]
remappings = ["@openzeppelin/=lib/openzeppelin-contracts/", "@aave/=lib/aave-v3-core/"]
```

**CI (`.github/workflows/ci.yml`)**

* Steps: `forge fmt --check`, `forge build`, `forge test -vvv`, `forge coverage`, `slither .` (if available)
* Fail if coverage < 95% lines/branches on contracts we own.

**Pre‑commit**

* `solhint`, `prettier-plugin-solidity`, `forge fmt`

---

## 6) Testing Strategy

Use Foundry. Include unit, scenario, and property/invariant tests.

### 6.1 Unit & Scenario Tests

1. **Core\_Quote.t.sol**

   * Zero utilization → `p30 = base`
   * Near saturation `U → U_max-ε` → `p30 ≤ p30Max`
   * Tenor scaling linear sanity

2. **Pool\_Buy.t.sol**

   * Succeeds when `cFree >= lockAmt` and `U_new < U_max`
   * Fails when `U_new ≥ U_max` (capacity exceeded)
   * Fails when `activationDelay` would cross maturity
   * Fee accounting: `treasury` receives fee, `cTotal += poolTake`, `cFree -= lockAmt + ...`

3. **Activation\_Gating.t.sol**

   * `redeem` in settled series rejects CT with `activationStart > eventTime`
   * Accepts when `activationStart <= eventTime`

4. **Pool\_Settle.t.sol**

   * Binary: `payoutRatio=1e18` pays all locked
   * Proportional: pays `r*N`, returns leftover to `cFree`
   * Post‑settlement: `paused=true`, `cooldownUntil` set

5. **Pool\_Mature.t.sol**

   * No event: `cLocked → cFree`, `L_total` decremented; `cTotal` unchanged

6. **LP\_DepositWithdraw\.t.sol**

   * First depositor `shares=amount`
   * Later deposit `shares = amount*S_total/NAV`
   * Withdraw pays min(NAV share, `cFree`), queues remainder, burns shares proportionally

7. **RBC\_Guards.t.sol**

   * Set rho>0, verify `requiredCapital` view
   * If `cTotal < required`, sales are blocked (or Pool is paused)

8. **Pause\_Cooldown.t.sol**

   * `pause()` blocks `buy`
   * During cooldown, `buy` blocked, `deposit` allowed, `redeem` allowed

9. **Decimals\_6\_18.t.sol**

   * Run identical flows with 6‑dec and 18‑dec assets

### 6.2 Property/Invariant Tests (`/test/property/Invariants.t.sol`)

Express as always‑true properties after each action:

* `∀j: C_locked[j] >= rMax[j]*N_j` (by design equal; test equality)
* `L_total == Σ_j C_locked[j]`
* `C_total == C_free + Σ_j C_locked[j]`
* After `buy`: `U_new < U_max`
* `withdrawable ≤ C_free`
* `post‑settle: paused == true` and `cooldownUntil > now`
* Never overpay: total redeemed per series ≤ `payTotal`
* Monotonicity: once `series.settled` or `matured`, cannot transition back to active

Use fuzzing: vary `notional`, `tenor`, times, decimals, fee rates, `k`, `S*`. Include corner cases: `U≈0`, `U≈U_max`, `p30≈p30Max`, tiny notionals, huge notionals, fee=0, fee>0.

---

## 7) Deployment & Runbooks

**Deploy.s.sol**

* Deploy CT and LP tokens
* Initialize `PoolStorage.ProductConfig`:

  * basePrice30dWad, scarcityK, S\*, U\_max, P30\_max, feePremWad
  * activationDelay, cooldownAfterPayout
  * addresses: asset, oracle, treasury, ctToken, lpToken
* Set initial RBC params (optional)
* Grant roles: GOVERNOR to multisig, OPERATOR to ops wallet, POOL role to Pool in tokens

**CreateSeries.s.sol**

* For each tenor in config, compute `seriesId = keccak(productId, maturity)` via `TokenIdLib`, call `PoolAdmin.createSeries(maturity, rMaxWad)`

**Seed.s.sol**

* Optional: seed `cFree` via `deposit`

**Runbook**

* Settlement flow: ops runs a bot that watches oracle; when confirmed, calls `Pool.settle(seriesId)`
* Queue servicing: ops can call `settleQueue()` periodically or anyone can call and take a keeper tip if we add incentives (not in MVP)

---

## 8) Security Checklist

* **Reentrancy**: NonReentrant on state‑changing external functions in Pool. No external calls between Storage delta application and token/accounting critical sections. Pull payments for CT redemption.
* **Pause/Cooldown**: `buy` must check both `paused` and `cooldownUntil`.
* **Oracle Trust**: validate `eventTime != 0`, and reject multiple settlements. Prevent oracle from returning `r > rMax`.
* **Rounding**: Use the exact same rounding as Core when recomputing simple amounts in Pool (e.g., `redeem` pay computation). Prefer to pass through Core results when possible.
* **Access Control**: Tokens mint/burn restricted to Pool. Admin functions restricted by roles.
* **Approve/Transfer**: Use SafeERC20. Support fee‑on‑transfer tokens explicitly as unsupported for MVP; assert `pre/post` balances if needed.
* **DoS by large holder set**: No per‑holder loops. CT redemption is per holder (pull model).
* **Storage Layout**: Do not change storage structs without a migration plan. If you later introduce proxies, freeze layout.

---

## 9) Gas & Engineering Hygiene

* Cache storage pointers (e.g., `PoolStorage.Layout storage L = PoolStorage.layout();`)
* Pack variables where possible (`uint64` times together)
* Use `unchecked` in tight increments only when proven safe
* Emit events last after state is finalized
* Prefer `memory` over `calldata` only where needed
* Avoid string concatenations on‑chain

---

## 10) Acceptance Criteria

The build is “done” when:

1. All unit and scenario tests pass with ≥ 95% coverage
2. All invariants in property tests hold under fuzzing (≥ 100k runs per seed in CI)
3. `forge script` deployments succeed on a local fork with:

   * Buy → Mature → Buy → Settle → Redeem flows verified on a 6‑dec asset
   * RBC guard prevents sales when required capital exceeds total capital
4. Slither reports no high/medium issues (false positives documented)
5. Gas snapshots recorded; hot paths (`buy`, `redeem`, `deposit`, `withdraw`) are within budget (< 350k each in typical flows for 6‑dec asset on L2; document actuals)

---

## 11) Implementation Order (Sprint Plan)

**Sprint 1**

* Tokens (CT, LP)
* PoolStorage/StorageCodec/SeriesStorage final polish
* Interfaces, Events, Errors
* Pool “read‑only” views
* Unit tests for tokens and storage encoding

**Sprint 2**

* Pool primary market (`buy`), `deposit`, `withdraw` (without queue)
* Basic tests for buy/deposit/withdraw
* Decimals tests

**Sprint 3**

* Settlement (`settle`), maturity (`mature`), redeem
* Oracle adapter and mocks
* Tests for activation gating, settle/redeem/mature

**Sprint 4**

* Withdraw queue MVP
* RBC checks
* Pause/cooldown logic
* Property tests, fuzzing, gas snapshots

**Sprint 5**

* CI, scripts, docs, runbooks
* Dry‑run deployments on a public testnet fork

---

## 12) Documentation

* Update `README.md` with:

  * High‑level architecture diagram
  * Math specification cross‑reference (from the paper)
  * Contract list and responsibilities
  * Deployment instructions
  * Admin runbooks (how to pause, set fees, create series, handle settlements)
  * Security model and known limitations (e.g., proportional model oracle trust, queue fairness)

---

## 13) Notes on Future Extensions (not MVP)

* Exact exponential time scaling for `P_T` (replace linear with `1 - (1 - P30)^(T/30)` using `ln/exp` approximation with safe bounds)
* Reinsurance pools with split locks (primary vs reinsurer) and mirrored Core calls
* Permit2 for premium payments
* Subgraph indexing for off‑chain analytics

---

If you implement exactly the above, you will have a production‑grade MVP that respects the invariants (purchase‑time lock, capacity guard, free‑capital withdrawals) and is auditable with clear separation between pure Core and imperative orchestration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JinTanba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
