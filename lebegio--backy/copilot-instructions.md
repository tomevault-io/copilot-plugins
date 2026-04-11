## backy

> Ethereum MEV backrunner. Single-file C++17 (`src/main.cpp`). Subscribes to local node mempool via IPC, detects pending DEX swaps, simulates them with a ghost wallet, identifies profitable backrun opportunities.

# backy_bot

Ethereum MEV backrunner. Single-file C++17 (`src/main.cpp`). Subscribes to local node mempool via IPC, detects pending DEX swaps, simulates them with a ghost wallet, identifies profitable backrun opportunities.

## Pipeline

hash → `eth_getTransactionByHash` → `TxFilter` → `TxSimulator` (debug_traceCall) → `detect_swap` → `SafetyChecker` → act

## Key classes (all in src/main.cpp)

| Class | Role |
|---|---|
| `IpcRpcClient` | Fresh Unix socket per JSON-RPC call |
| `MempoolSubscriber` | Persistent `eth_subscribe(newPendingTransactions)`, exponential backoff reconnect |
| `TxFilter` | Gates on: known DEX router + non-empty calldata |
| `GhostWallet` | `0xaaaa...` with unlimited balances via `stateOverrides` (WETH/USDC/USDT/DAI/WBTC + Permit2) |
| `TxSimulator` | Replaces `from` with ghost, calls `debug_traceCall callTracer withLog:true` |
| `StorageSlot` | Computes ERC-20 `_balances`/`_allowances` slots via keccak256 |
| `SwapInfo` | token_in/out, amounts, dex, price_before/after |
| `detect_swap()` | Largest net outflow = token_in, largest inflow = token_out |
| `SafetyChecker` | Guards before acting (liquidity etc.) |
| `Pricer` | AMM spot price for price_before |

## Watched DEXes

Uniswap V2/V3/Universal, SushiSwap, 1inch v5/v6

## Logging

Two streams: main logger (stats every 5k hashes, profitable hits) + swap_logger (one line per DEX tx: dex from to pair in out gas gasPrice price_before price_after slippage).

## Build

```bash
scripts/run.sh                           # cmake build + run
LOCAL_NODE_IPC=/path/to.ipc scripts/run.sh
```

C++17, CMake 3.15+. Deps: Boost 1.74+ headers, OpenSSL, nlohmann_json, zlib.
Binary: `build/backy_bot`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lebegio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lebegio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
