## polyaave-contract-dev

> * **Liquidity pool** holds a single base asset (e.g., USDC). LP tokens are minted **1-for-1** with deposits, so the invariant `Σ lpBalances == pool.totalSupplied` always holds.

### Chapter 1 — Financial & Mathematical Specification

---

#### 1.1  Asset & Pool Model

* **Liquidity pool** holds a single base asset (e.g., USDC). LP tokens are minted **1-for-1** with deposits, so the invariant `Σ lpBalances == pool.totalSupplied` always holds.
* **Collateral** consists of ERC-1155/20 *prediction-market position* tokens whose live price is delivered by an oracle and collapses to a deterministic value at market maturity.
* **Debt stack** has two layers:

  1. **Principal** sourced from Aave V3 (variable-rate).
  2. **Spread** (protocol margin) tracked per-market through `variableBorrowIndex`.

#### 1.2  Canonical Data Types (Ray precision)

```solidity
struct PoolData   { uint256 totalSupplied; /* … */ }
struct MarketData { uint256 variableBorrowIndex; uint256 totalBorrowed; /* … */ }
struct UserPosition { uint256 collateralAmount; uint256 borrowAmount; uint256 scaledDebtBalance; }
```

All structs live in **`DataStruct.sol`**; storage contracts merely import them.

#### 1.3  Utilisation→Spread Function

`util = totalBorrowed / totalSupplied` (Ray).

```
if util ≤ U* :  rate = base + slope1 × util
else          :  rate = base + slope1 × U* + slope2 × (util − U*)
```

Governance tunes `{base, U*, slope1, slope2}`; the piece-wise curve protects LPs as liquidity tightens (see `CoreMath.calculateSpreadRate`, code reference).

#### 1.4  Multi-Level Debt Formulae

$$
\begin{aligned}
D_{m} &= D_{\text{protocol}}\;\frac{B_m}{\sum B_m} \\
P_{u} &= D_{m}\;\frac{b_u}{B_m} \\
S_{u} &= \bigl(\text{scaledDebt}_u \times \text{index}_m\bigr)\;-\;b_u \\
\text{Debt}_{u} &= P_{u}+S_{u}
\end{aligned}
$$

Exactly the same expressions are embedded in the README for auditors.

#### 1.5  Risk Limits & Global Invariants

| Invariant                                                                                          | Purpose                     |
| -------------------------------------------------------------------------------------------------- | --------------------------- |
| `Σ market.totalBorrowed == pool.totalBorrowedAllMarkets`                                           | accounting consistency      |
| `pool.totalBorrowedAllMarkets ≤ pool.totalSupplied`                                                | cannot over-lend base asset |
| **LTV** per borrower enforced before every borrow; breach opens liquidation (logic stubbed in V2). |                             |

---

### Chapter 2 — Technical & Architectural Specification

---

#### 2.1  Layered Design (Functional-Core / Imperative-Shell)

| Layer            | File               | Responsibility                                                     |
| ---------------- | ------------------ | ------------------------------------------------------------------ |
| **CoreMath**     | `CoreMath.sol`     | Pure, deterministic maths (no storage)                             |
| **Core**         | `Core.sol`         | Stateless façade: gathers structs, calls CoreMath, mutates storage |
| **StorageShell** | `StorageShell.sol` | Minimal storage + events; no business logic                        |

Because behaviour is defined **once** in CoreMath, upgrading Shell contracts or routing logic never changes protocol economics.

#### 2.2  State-Transition Pipeline

| Operation       | Key Steps                                                                                                 |
| --------------- | --------------------------------------------------------------------------------------------------------- |
| **Supply**      | `_processSupply` → `calculateLPTokensToMint` → update `totalSupplied` & LP balance                        |
| **Borrow**      | `_updateMarketIndices` → `_processBorrow` → oracle-price & LTV check → Aave borrow → scale Polynance debt |
| **Repay**       | `_processRepay` → compute debt (§1.4) → repay principal to Aave, pocket spread; update scaled balances    |
| **Resolution**  | `_processResolution` (TODO) redeems collateral, nets P/L, pays LPs (not yet merged).                      |
| **Liquidation** | `_processLiquidation` placeholder; enable only after implementation.                                      |

#### 2.3  Security Surface

* **Deterministic maths**: all critical calculations are `pure`; SafeCast and zero-division guards are mandatory.
* **Oracle risk**: stale/manipulated prices → under-collateralisation → mitigated with conservative LTV and multi-feed median.
* **Rate arbitrage**: governance tunes spread curve to ensure LP yield > Aave APR.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JinTanba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
