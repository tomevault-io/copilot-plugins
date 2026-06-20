## solana-payments-skill

> Integrate Solana SPL token payments (USDC/USDT) into the Toppio payment provider system. Use this skill when implementing the Solana payment provider, handling SPL token transfers, building sweep logic for Solana, or debugging Solana-related payment issues.


This skill guides implementation of a Solana SPL token payment provider for Toppio's existing chain-agnostic `PaymentProvider` interface. It covers wallet generation, balance checking, transfer detection, and sweeping — all adapted to Solana's unique account model.

## Solana vs EVM — Key Differences

Solana is NOT EVM-compatible. Do not use ethers.js or EVM patterns. Key differences:

| Concept | EVM (BSC/Polygon) | Solana |
|---------|-------------------|--------|
| Library | ethers.js v6 | @solana/web3.js + @solana/spl-token |
| Keypair | Random wallet (secp256k1) | Ed25519 keypair |
| Token balances | ERC-20 contract.balanceOf() | Associated Token Account (ATA) |
| Token transfer | contract.transfer() | createTransferInstruction() |
| Gas token | BNB / POL | SOL |
| Gas cost | ~$0.01-0.05 | ~$0.001 (base) + ~$0.40 (ATA creation) |
| Finality | ~3-15s | ~400ms (optimistic), ~30s (confirmed) |
| Block scanning | Transfer events via getLogs | getSignaturesForAddress + getParsedTransaction |

## Token Addresses (Mainnet)

```typescript
// USDC — Token Program (not Token-2022)
const USDC_MINT = 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v';

// USDT — Token Program (not Token-2022)
const USDT_MINT = 'Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB';

// Token Program address (both USDC and USDT use this, NOT Token-2022)
const TOKEN_PROGRAM_ID = 'TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA';
```

**CRITICAL**: USDC and USDT on Solana both use the original Token Program, NOT Token-2022. Using the wrong program produces invalid ATAs and failed transfers.

## Dependencies

```bash
npm install @solana/web3.js@1 @solana/spl-token bs58
```

Use `@solana/web3.js` v1 (stable). v2 is a full rewrite with different APIs. `bs58` is needed for encoding/decoding Solana keypairs (not built into web3.js).

## Provider Implementation

### File: `providers/payment/solana.ts`

Implement the `PaymentProvider` interface from `providers/payment/types.ts`:

```typescript
interface PaymentProvider {
  readonly chain: string;           // 'Solana'
  readonly chainId: string;         // 'solana'
  readonly token: string;           // 'USDC/USDT'
  readonly decimals: number;        // 6 (both USDC and USDT on Solana are 6 decimals)
  readonly gasToken: string;        // 'SOL'

  generateDepositAddress(): Promise<DepositAddress>;
  checkBalance(address: string): Promise<BalanceResult>;
  getIncomingTransfers(address: string, fromBlock?: number): Promise<TransferInfo[]>;
  sweep(fromPrivateKey: string, toAddress: string): Promise<SweepResult>;
  getExplorerUrl(txHash: string): string;
  getAddressExplorerUrl(address: string): string;
  isValidAddress(address: string): boolean;
  getMasterGasBalance(): Promise<string>;
}
```

### Wallet Generation

```typescript
import { Keypair } from '@solana/web3.js';
import bs58 from 'bs58';

async generateDepositAddress(): Promise<DepositAddress> {
  const keypair = Keypair.generate();
  return {
    address: keypair.publicKey.toBase58(),
    // Store secret key as base58 string (compatible with existing encrypt/decrypt)
    privateKey: bs58.encode(keypair.secretKey),
  };
}
```

Note: Solana secret keys are 64 bytes (includes the public key). Store the full secretKey, not just the seed.

### Address Validation

```typescript
import { PublicKey } from '@solana/web3.js';

isValidAddress(address: string): boolean {
  try {
    new PublicKey(address);
    return true;
  } catch {
    return false;
  }
}
```

### Balance Checking

```typescript
import { Connection, PublicKey } from '@solana/web3.js';
import { getAssociatedTokenAddress, getAccount, TokenAccountNotFoundError } from '@solana/spl-token';

async checkBalance(address: string): Promise<BalanceResult> {
  const connection = new Connection(this.rpcUrl);
  const owner = new PublicKey(address);
  let total = 0n;

  for (const mintStr of [USDC_MINT, USDT_MINT]) {
    const mint = new PublicKey(mintStr);
    const ata = await getAssociatedTokenAddress(mint, owner);
    try {
      const account = await getAccount(connection, ata);
      total += account.amount; // bigint, already in smallest units
    } catch (e) {
      if (e instanceof TokenAccountNotFoundError) continue; // ATA doesn't exist yet
      throw e;
    }
  }

  const balance = Number(total) / 10 ** this.decimals;
  return { balance, raw: total.toString() };
}
```

**Important**: ATAs may not exist for new deposit wallets. This is normal — the ATA gets created when the sender transfers tokens. Most wallets and dApps handle ATA creation automatically.

### Incoming Transfer Detection

Solana doesn't have event logs like EVM. Use signature history + parsed transactions:

```typescript
import { Connection, PublicKey, ParsedTransactionWithMeta } from '@solana/web3.js';

async getIncomingTransfers(address: string): Promise<TransferInfo[]> {
  const connection = new Connection(this.rpcUrl);
  const owner = new PublicKey(address);
  const transfers: TransferInfo[] = [];

  for (const mintStr of [USDC_MINT, USDT_MINT]) {
    const mint = new PublicKey(mintStr);
    const ata = await getAssociatedTokenAddress(mint, owner);

    // Get recent signatures for the ATA
    const signatures = await connection.getSignaturesForAddress(ata, {
      limit: 20,
    });

    for (const sig of signatures) {
      if (sig.err) continue; // skip failed txs

      const tx = await connection.getParsedTransaction(sig.signature, {
        maxSupportedTransactionVersion: 0,
      });
      if (!tx?.meta) continue;

      // Look for token transfers TO this ATA in the parsed instructions
      for (const ix of tx.transaction.message.instructions) {
        if (!('parsed' in ix)) continue;
        if (ix.parsed?.type === 'transferChecked' || ix.parsed?.type === 'transfer') {
          const info = ix.parsed.info;
          if (info.destination === ata.toBase58()) {
            const amount = ix.parsed.type === 'transferChecked'
              ? Number(info.tokenAmount.amount) / 10 ** this.decimals
              : Number(info.amount) / 10 ** this.decimals;

            transfers.push({
              hash: sig.signature,
              from: info.authority || info.source,
              amount,
            });
          }
        }
      }
    }
  }

  return transfers;
}
```

**Note**: The `fromBlock` parameter from the interface doesn't map directly to Solana. Use the `before`/`until` signature options or timestamp filtering if needed. For the watcher's 15s poll cycle, fetching recent signatures is sufficient.

### Sweep Logic

Sweeping on Solana differs fundamentally from EVM. No permit/approve pattern. Instead:

1. The deposit wallet signs a transfer instruction directly
2. Master wallet can pay for the transaction fee (as fee payer)
3. ATA creation on the master side may be needed

```typescript
import {
  Connection, Keypair, PublicKey, Transaction, sendAndConfirmTransaction,
  SystemProgram, LAMPORTS_PER_SOL,
} from '@solana/web3.js';
import {
  getAssociatedTokenAddress, createAssociatedTokenAccountInstruction,
  createTransferInstruction, getAccount, TokenAccountNotFoundError,
} from '@solana/spl-token';

async sweep(fromPrivateKey: string, toAddress: string): Promise<SweepResult> {
  const connection = new Connection(this.rpcUrl);
  const fromKeypair = Keypair.fromSecretKey(bs58.decode(fromPrivateKey));
  const toPublicKey = new PublicKey(toAddress);

  let totalSwept = 0;
  let lastTxHash = '';

  for (const mintStr of [USDC_MINT, USDT_MINT]) {
    const mint = new PublicKey(mintStr);

    // Source ATA (deposit wallet)
    const sourceAta = await getAssociatedTokenAddress(mint, fromKeypair.publicKey);

    // Check balance
    let balance: bigint;
    try {
      const account = await getAccount(connection, sourceAta);
      balance = account.amount;
    } catch (e) {
      if (e instanceof TokenAccountNotFoundError) continue;
      throw e;
    }
    if (balance === 0n) continue;

    // Destination ATA (master wallet)
    const destAta = await getAssociatedTokenAddress(mint, toPublicKey);

    const tx = new Transaction();

    // Create destination ATA if it doesn't exist
    try {
      await getAccount(connection, destAta);
    } catch (e) {
      if (e instanceof TokenAccountNotFoundError) {
        tx.add(
          createAssociatedTokenAccountInstruction(
            fromKeypair.publicKey, // payer (needs SOL)
            destAta,
            toPublicKey,
            mint,
          )
        );
      } else {
        throw e;
      }
    }

    // Transfer all tokens
    tx.add(
      createTransferInstruction(
        sourceAta,
        destAta,
        fromKeypair.publicKey, // authority (signer)
        balance,
      )
    );

    // Deposit wallet needs SOL for tx fees
    // Check if it has enough, if not fund from master
    const solBalance = await connection.getBalance(fromKeypair.publicKey);
    const estimatedFee = 10_000; // ~0.00001 SOL, generous estimate

    if (solBalance < estimatedFee) {
      // Fund deposit wallet with SOL from master
      const masterKeypair = Keypair.fromSecretKey(
        bs58.decode(process.env.SOLANA_MASTER_PRIVATE_KEY!)
      );
      const fundTx = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: masterKeypair.publicKey,
          toPubkey: fromKeypair.publicKey,
          lamports: 50_000, // ~$0.007, enough for several txs
        })
      );
      await sendAndConfirmTransaction(connection, fundTx, [masterKeypair]);
    }

    const txHash = await sendAndConfirmTransaction(connection, tx, [fromKeypair]);
    lastTxHash = txHash;
    totalSwept += Number(balance) / 10 ** this.decimals;
  }

  if (!lastTxHash) throw new Error('No tokens to sweep');
  return { txHash: lastTxHash, amount: totalSwept };
}
```

**Alternative sweep approach — Master as fee payer** (saves a funding step):

```typescript
// Master wallet pays the fee, deposit wallet only signs the transfer
const masterKeypair = Keypair.fromSecretKey(
  bs58.decode(process.env.SOLANA_MASTER_PRIVATE_KEY!)
);

const tx = new Transaction();
tx.feePayer = masterKeypair.publicKey; // Master pays fees

tx.add(
  createTransferInstruction(
    sourceAta,
    destAta,
    fromKeypair.publicKey, // authority
    balance,
  )
);

// Both must sign: master (fee payer) + deposit wallet (token authority)
const txHash = await sendAndConfirmTransaction(connection, tx, [masterKeypair, fromKeypair]);
```

This is the **preferred approach** — it eliminates the SOL funding step entirely. The master wallet pays ~$0.001 in SOL per sweep, and the deposit wallet never needs SOL.

### Explorer URLs

```typescript
getExplorerUrl(txHash: string): string {
  return `https://solscan.io/tx/${txHash}`;
}

getAddressExplorerUrl(address: string): string {
  return `https://solscan.io/account/${address}`;
}
```

### Master Gas Balance

```typescript
async getMasterGasBalance(): Promise<string> {
  const connection = new Connection(this.rpcUrl);
  const masterAddress = new PublicKey(process.env.SOLANA_MASTER_WALLET_ADDRESS!);
  const balance = await connection.getBalance(masterAddress);
  return (balance / LAMPORTS_PER_SOL).toFixed(4);
}
```

## Registration

Add to `providers/payment/index.ts`:

```typescript
import { SolanaPaymentProvider } from './solana';

const providers: Record<string, () => PaymentProvider> = {
  bsc: () => new BnbPaymentProvider(),
  polygon: () => new PolygonPaymentProvider(),
  solana: () => new SolanaPaymentProvider(),
};
```

## Environment Variables

```env
# Solana
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
SOLANA_MASTER_WALLET_ADDRESS=<base58 public key>
SOLANA_MASTER_PRIVATE_KEY=<base58 encoded secret key>
```

**RPC Note**: The default public RPC (`api.mainnet-beta.solana.com`) has strict rate limits. For production, use a dedicated RPC from Helius, QuickNode, or Alchemy.

## Configuration Constants

```typescript
const LOW_SOL_THRESHOLD = 0.05;  // SOL — warn in admin dashboard
const DECIMALS = 6;               // Both USDC and USDT are 6 decimals on Solana
const SIGNATURE_LIMIT = 20;       // Max recent signatures to check
```

## Gotchas and Edge Cases

1. **ATA may not exist**: New deposit wallets have no ATAs. Most sending wallets/dApps create the recipient's ATA during transfer. If the sender doesn't, the transfer fails on their end (not ours).

2. **Decimals are 6**: Unlike BSC where USDC has 18 decimals, Solana USDC/USDT both use 6 decimals (matching their real-world value: 1_000_000 = $1.00).

3. **No permit/approve pattern**: Solana uses direct authority-based transfers. The owner of tokens signs the transfer instruction directly. No need for approve + transferFrom flows.

4. **Fee payer separation**: Unlike EVM where msg.sender pays gas, Solana transactions have an explicit `feePayer` field. Use the master wallet as fee payer to avoid funding deposit wallets with SOL.

5. **Transaction confirmation**: Use `confirmed` commitment for balance checks and `finalized` for sweep confirmations:
   ```typescript
   const connection = new Connection(rpcUrl, 'confirmed');
   // For sweeps, wait for finalized:
   await sendAndConfirmTransaction(connection, tx, signers, { commitment: 'finalized' });
   ```

6. **RPC rate limits**: Public Solana RPC has aggressive rate limits. Batch requests carefully. For the watcher's per-order checks, consider using `getMultipleAccountsInfo` to batch ATA lookups.

7. **Rent exemption**: Token accounts require ~0.00203928 SOL for rent exemption. This is paid during ATA creation. When the master wallet creates ATAs, budget for this cost.

8. **Private key format**: Solana uses 64-byte Ed25519 keys (not 32-byte seeds). Store as base58. The existing AES-256-GCM encryption in `lib/crypto.ts` works fine — it encrypts arbitrary strings.

9. **No block numbers**: Solana uses slot numbers, not block numbers. The `fromBlock` parameter in `getIncomingTransfers` should be ignored or adapted to use `before`/`until` signature cursors.

10. **Transaction versioning**: Always pass `maxSupportedTransactionVersion: 0` when fetching parsed transactions, otherwise versioned transactions return null.

## Watcher Integration Notes

The existing watcher in `scripts/watcher.ts` loops per chain. The Solana provider will be picked up automatically once registered. However:

- **Poll timing**: Solana's ~400ms block time means 15s polling is fine; payments will be detected quickly.
- **Transfer detection**: Unlike EVM event scanning, Solana uses `getSignaturesForAddress` which returns the most recent signatures. No need to track block numbers.
- **Sweep timing**: Sweeps complete in ~1-2s on Solana (vs 3-15s on EVM). The existing retry logic with 3 max attempts works well.

## Testing Checklist

Before going live:

- [ ] Generate keypair and verify address format (base58, 32-44 chars)
- [ ] Encrypt/decrypt private key through `lib/crypto.ts` roundtrip
- [ ] Check balance returns 0 for empty wallet
- [ ] Send USDC on devnet, verify `checkBalance` detects it
- [ ] Verify `getIncomingTransfers` returns correct hash, sender, amount
- [ ] Sweep tokens to master wallet with master as fee payer
- [ ] Verify explorer URLs resolve correctly
- [ ] Test with both USDC and USDT mints
- [ ] Confirm watcher detects and processes Solana orders end-to-end
- [ ] Verify admin dashboard shows SOL gas balance

## Payment Verification

Three approaches to verify incoming payments, from simplest to most robust:

### 1. Balance Query (simplest — used by `checkBalance`)

```typescript
// Already covered above. Use getAccount() on the ATA.
// Limitation: only shows net balance, not individual transfers.
```

### 2. WebSocket Subscription (real-time monitoring)

For lower-latency detection than 15s polling, subscribe to ATA account changes:

```typescript
import { createSolanaRpcSubscriptions } from '@solana/kit';

async function watchPayments(tokenAccountAddress: string, onPayment: (amount: bigint) => void) {
  const rpcSubscriptions = createSolanaRpcSubscriptions('wss://api.mainnet-beta.solana.com');
  const abortController = new AbortController();

  const subscription = await rpcSubscriptions
    .accountNotifications(tokenAccountAddress, {
      commitment: 'confirmed',
      encoding: 'base64',
    })
    .subscribe({ abortSignal: abortController.signal });

  let previousBalance = 0n;
  for await (const notification of subscription) {
    // Parse token account data to extract balance
    // Compare with previous balance to detect incoming payment
    const currentBalance = /* parse from notification.value.data */;
    if (currentBalance > previousBalance) {
      onPayment(currentBalance - previousBalance);
    }
    previousBalance = currentBalance;
  }
}
```

**Note**: WebSocket subscriptions are optional. The existing 15s watcher poll cycle works fine for Toppio's use case. Consider WebSocket only if faster detection is needed.

### 3. Transaction History with Pre/Post Balances (most accurate)

Parse `preTokenBalances` and `postTokenBalances` from transaction metadata for precise per-transaction amounts:

```typescript
async function getRecentPayments(ataAddress: string, mintAddress: string, limit = 100) {
  const signatures = await connection.getSignaturesForAddress(new PublicKey(ataAddress), { limit });
  const payments = [];

  for (const sig of signatures) {
    const tx = await connection.getTransaction(sig.signature, { maxSupportedTransactionVersion: 0 });
    if (!tx?.meta?.preTokenBalances || !tx?.meta?.postTokenBalances) continue;

    const accountKeys = tx.transaction.message.accountKeys;
    const ataIndex = accountKeys.findIndex(key => key.toBase58() === ataAddress);
    if (ataIndex === -1) continue;

    const pre = tx.meta.preTokenBalances.find(b => b.accountIndex === ataIndex && b.mint === mintAddress);
    const post = tx.meta.postTokenBalances.find(b => b.accountIndex === ataIndex && b.mint === mintAddress);

    const preAmount = BigInt(pre?.uiTokenAmount.amount ?? '0');
    const postAmount = BigInt(post?.uiTokenAmount.amount ?? '0');
    const diff = postAmount - preAmount;

    if (diff > 0n) {
      payments.push({
        signature: sig.signature,
        timestamp: tx.blockTime,
        amount: diff,
        type: 'incoming',
      });
    }
  }
  return payments;
}
```

This is more reliable than parsing instructions because it accounts for complex multi-instruction transactions.

## Commitment Levels

| Level | Latency | Safety | Use For |
|-------|---------|--------|---------|
| `processed` | ~400ms | Can be dropped during forks | UI feedback only — never for business logic |
| `confirmed` | ~1-2s | Supermajority voted | Balance checks, payment detection |
| `finalized` | ~13s | Irreversible | Sweep confirmations, high-value operations |

**Toppio mapping**:
- `checkBalance()` → `confirmed`
- `getIncomingTransfers()` → `confirmed`
- `sweep()` → `finalized` (wait for irreversibility before marking complete)

## Transaction Confirmation & Retry

### Blockhash Management

Solana transactions reference a recent blockhash and expire after ~60-90 seconds (~150 blocks):

```typescript
const { blockhash, lastValidBlockHeight } = await connection.getLatestBlockhash('confirmed');
tx.recentBlockhash = blockhash;

// Send with manual retry control
const signature = await connection.sendTransaction(tx, signers, {
  maxRetries: 0,        // We handle retries ourselves
  skipPreflight: false,  // Keep preflight during dev, can disable in prod
});

// Poll for confirmation with expiry awareness
const confirmation = await connection.confirmTransaction({
  signature,
  blockhash,
  lastValidBlockHeight,
}, 'finalized');

if (confirmation.value.err) {
  throw new Error(`Transaction failed: ${JSON.stringify(confirmation.value.err)}`);
}
```

### Priority Fees

During network congestion, add priority fees to ensure inclusion:

```typescript
import { ComputeBudgetProgram } from '@solana/web3.js';

// Add compute budget instructions BEFORE transfer instructions
tx.add(
  ComputeBudgetProgram.setComputeUnitLimit({ units: 200_000 }),
  ComputeBudgetProgram.setComputeUnitPrice({ microLamports: 50_000 }), // ~$0.001-0.005
);
```

Normal cost: ~$0.001. During congestion: $0.01-0.05. Use RPC provider priority fee APIs (Helius, QuickNode) for dynamic pricing.

### Error Handling

```typescript
// Retryable errors — rebuild with fresh blockhash and retry
const RETRYABLE_ERRORS = [
  'BlockhashNotFound',
  'BlockheightExceeded',
  // Network timeouts
];

// Non-retryable errors — surface to user/admin
const FATAL_ERRORS = [
  'InsufficientFundsForFee',  // Master wallet needs more SOL
  'InsufficientFunds',         // Token balance insufficient
  'AccountNotFound',           // ATA doesn't exist (create it)
];
```

## Security Checklist

1. **Hardcode mint addresses**: Never accept mint addresses dynamically. Spoofed tokens with identical names are common on Solana.
2. **Server-side verification only**: Never trust client-side "payment confirmed" signals.
3. **Idempotency**: Store processed transaction signatures to prevent double-fulfillment from watcher re-runs.
4. **Validate token program**: Verify the ATA's owner program matches `TOKEN_PROGRAM_ID`. A token account owned by Token-2022 would indicate a spoofed token.
5. **Separate hot/cold wallets**: The master wallet (hot) should hold minimal SOL for gas. Sweep to a cold wallet periodically.
6. **RPC URL as secret**: Treat RPC endpoints as credentials. Never expose in frontend code.
7. **Distinct devnet/mainnet keys**: Never reuse keypairs across environments.

## Production RPC

The public `api.mainnet-beta.solana.com` is rate-limited with no SLA. For production:

| Provider | Feature |
|----------|---------|
| **Helius** | Priority fee API, webhooks, enhanced RPCs |
| **QuickNode** | Multi-chain, add-ons for Solana-specific features |
| **Triton** | Yellowstone gRPC streaming, low-latency |
| **Alchemy** | Enterprise SLA, multi-region |

Implement multi-provider failover for reliability:

```typescript
const RPC_URLS = [
  process.env.SOLANA_RPC_URL!,           // Primary
  process.env.SOLANA_RPC_URL_FALLBACK!,  // Fallback
];
```

## Indexing (High Volume)

For basic Toppio order volume, RPC polling via `getSignaturesForAddress` is sufficient. If volume grows significantly, consider:

- **Yellowstone gRPC**: Real-time streaming directly from validators, sub-100ms latency. Available through Helius, Triton, QuickNode.
- **Webhooks**: Most RPC providers offer webhook notifications for account changes — eliminates polling entirely.
- **Carbon/Vixen**: Rust frameworks for building custom indexing pipelines (overkill for current Toppio scale).

## Devnet Testing

For testing, use Solana devnet with test tokens:

```typescript
// Devnet USDC (Circle test mint)
const DEVNET_USDC = '4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU';

// Devnet RPC
const DEVNET_RPC = 'https://api.devnet.solana.com';

// Get free SOL for testing
// solana airdrop 2 <address> --url devnet
```

Devnet faucets for test tokens: Circle (USDC), QuickNode, Solana CLI (`solana airdrop`).

---
> Source: [sepivip/solana-payments-skill](https://github.com/sepivip/solana-payments-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
