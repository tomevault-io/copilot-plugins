## raydium-sdk-swap-example-typescript

> This repository contains a TypeScript example for performing token swaps on Solana using the Raydium SDK. When working on the project interactively with an agent (e.g. the Codex CLI) please follow the guidelines below for safe development and testing.

# AGENTS Guidelines for This Repository

This repository contains a TypeScript example for performing token swaps on Solana using the Raydium SDK. When working on the project interactively with an agent (e.g. the Codex CLI) please follow the guidelines below for safe development and testing.

## 1. Use Simulation Mode for Testing

* **Always start with simulation** (`executeSwap: false` in swapConfig).
* **Test swaps thoroughly** before executing real transactions.
* **Do _not_ execute real swaps** during agent development sessions.
* **Monitor gas costs** even in simulation mode.

## 2. Keep Dependencies in Sync

If you update dependencies:

1. Use yarn to manage packages: `yarn add <package>`.
2. Update all packages: `yarn upgrade`.
3. Verify compatibility with Raydium SDK.
4. Test with latest Solana web3.js versions.

## 3. Environment Configuration

Create a `.env` file with required settings:

```env
RPC_URL=YOUR_CHAINSTACK_RPC_URL
WALLET_PRIVATE_KEY=YOUR_TEST_WALLET_PRIVATE_KEY
```

Never commit `.env` to version control.

## 4. Development Workflow

Follow this sequence for safe development:

1. **Download liquidity data**: Get latest pool information
   ```bash
   wget https://api.raydium.io/v2/sdk/liquidity/mainnet.json
   ```

2. **Configure swap**: Edit `src/swapConfig.ts`
   ```typescript
   executeSwap: false  // Always false for testing
   ```

3. **Trim liquidity data**: Optimize for your token pair
   ```bash
   ts-node src/trimMainnet.ts
   ```

4. **Test swap**: Run simulation first
   ```bash
   yarn swap
   ```

## 5. Swap Configuration

Key parameters in `src/swapConfig.ts`:

| Parameter | Purpose | Safe Default |
| --------- | ------- | ------------ |
| `executeSwap` | Execute vs simulate | `false` |
| `tokenAAmount` | Amount to swap | `0.01` |
| `maxLamports` | Max fee allowed | `1000000` |
| `maxRetries` | Retry attempts | `10` |

## 6. Data Management

### Liquidity File Optimization
The full `mainnet.json` is ~500MB. Always trim it:

```bash
# Download fresh data
wget https://api.raydium.io/v2/sdk/liquidity/mainnet.json

# Trim to your tokens
ts-node src/trimMainnet.ts

# Use trimmed file (< 1MB)
```

## 7. Token Address Management

Common token addresses:
* **SOL**: `So11111111111111111111111111111111111111112`
* **USDC**: `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`
* **USDT**: `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB`

Always verify addresses before swapping.

## 8. Testing Checklist

Before executing real swaps:

- [ ] Simulation successful
- [ ] Correct token addresses
- [ ] Appropriate slippage settings
- [ ] Sufficient SOL for fees
- [ ] Liquidity data is current
- [ ] RPC endpoint responsive
- [ ] Private key secure

## 9. Error Handling

Common issues and solutions:

**"Liquidity pool not found"**
- Update mainnet.json
- Verify token pair exists
- Check trimmed file contains pair

**"Transaction simulation failed"**
- Check wallet balance
- Verify token amounts
- Increase maxLamports

**"RPC request failed"**
- Check RPC endpoint
- Verify rate limits
- Use different RPC if needed

## 10. Performance Optimization

* Use trimmed liquidity files (loads in < 1s vs minutes).
* Cache pool information when possible.
* Implement retry logic for RPC calls.
* Use versioned transactions for efficiency.

## 11. Security Best Practices

* **Never commit private keys**.
* **Use test wallets** for development.
* **Start with small amounts** (0.01 SOL).
* **Always simulate first**.
* **Verify token addresses** from official sources.
* **Monitor slippage** to avoid MEV attacks.

## 12. Common Development Tasks

### Add New Token Pair
1. Find official token addresses
2. Update `swapConfig.ts`
3. Download fresh liquidity data
4. Trim for new pair
5. Test with simulation

### Debug Failed Swap
1. Check simulation output
2. Verify liquidity exists
3. Confirm wallet balances
4. Review transaction logs
5. Adjust maxLamports if needed

## 13. Useful Commands Recap

| Command | Purpose |
| ------- | ------- |
| `yarn` | Install dependencies |
| `yarn swap` | Run swap (simulation or real) |
| `ts-node src/trimMainnet.ts` | Trim liquidity data |
| `wget [liquidity URL]` | Download pool data |

## 14. RPC Configuration

For Chainstack nodes:
1. Sign up at https://console.chainstack.com
2. Deploy Solana node
3. Use HTTP endpoint for swaps
4. Monitor rate limits

## 15. Troubleshooting

### Common Issues

**"Module not found"**
```bash
yarn
```

**"Large file warning"**
- Never commit mainnet.json
- Always use trimmed version
- Add to .gitignore

**"Insufficient lamports"**
- Top up SOL for fees
- Reduce swap amount
- Increase maxLamports

---

Following these practices ensures safe token swap development, prevents loss of funds, and maintains efficient execution. Always simulate before executing real swaps and use test amounts during development.

---
> Source: [chainstacklabs/raydium-sdk-swap-example-typescript](https://github.com/chainstacklabs/raydium-sdk-swap-example-typescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
