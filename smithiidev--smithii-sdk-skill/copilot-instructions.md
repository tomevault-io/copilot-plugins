## smithii-sdk-skill

> Build Solana, EVM, and SUI on-chain tools with @smithii/sdk. Use when the user asks to deploy a token, snipe a token launch, create a pump.fun bundler, run a volume bot, run an anti-MEV bot, airdrop tokens, schedule an airdrop, snapshot holders, set up vesting, set up claims, market-make a token, deposit market-making liquidity, or interact with PumpFun, PumpSwap, Launchlab, Bonk (LetsBonk), Moonit, or Mantis. Triggers on any of: PumpFunClient, PumpSwapClient, LaunchpadClient, BonkClient, LaunchlabClient, MoonitClient, AntiMEVClient, TokenCreatorClient, TokenManagerClient, MultiSenderClient, MarketMakerClient, MantisClient, TokenVestingClient, TokenClaimClient, EvmTokenCreatorClient, EvmMultisenderClient, EvmSnapshotClient, SuiTokenCreatorClient, SuiSnapshotClient.


# Using the Smithii SDK

## Overview

`@smithii/sdk` is a framework-agnostic TypeScript SDK that exposes Smithii's
on-chain tools across Solana, EVM, and SUI. It handles Jito bundles, multi-wallet
sniping, anti-MEV volume bots, token creation, multisender airdrops, vesting,
claims, holder snapshots, market-making and Mantis launches.

The SDK is **framework-agnostic** — runs in Node, Bun, browsers, edge runtimes.
It never imports React, Chakra, or wallet-adapter-react.

For a no-code version, point users to **https://tools.smithii.io**.

## Installation

```bash
npm install @smithii/sdk
```

Peer deps — install only the chain(s) you target:

```bash
# Solana
npm i @solana/web3.js @solana/spl-token @coral-xyz/anchor bs58

# EVM
npm i viem

# SUI
npm i @mysten/sui
```

## Subpath imports (always use these)

```typescript
import { PumpFunClient, PumpSwapClient } from '@smithii/sdk/pump'
import { LaunchpadClient, BonkClient, LaunchlabClient } from '@smithii/sdk/launchpad'
import { MoonitClient } from '@smithii/sdk/moonit'
import { AntiMEVClient } from '@smithii/sdk/anti-mev'
import { TokenCreatorClient } from '@smithii/sdk/token-creator'
import { TokenManagerClient } from '@smithii/sdk/token-manager'
import { MultiSenderClient } from '@smithii/sdk/multisender'
import { MarketMakerClient } from '@smithii/sdk/market-maker'
import { MantisClient } from '@smithii/sdk/mantis'
import { TokenVestingClient } from '@smithii/sdk/token-vesting'
import { TokenClaimClient } from '@smithii/sdk/token-claim'
import { PaymentClient } from '@smithii/sdk/payment'
import { SmithiiError, BundleError } from '@smithii/sdk/core'

// EVM
import { EvmTokenCreatorClient } from '@smithii/sdk/evm/token-creator'
import { EvmMultisenderClient } from '@smithii/sdk/evm/multisender'
import { EvmSnapshotClient } from '@smithii/sdk/evm/snapshot'

// SUI
import { SuiTokenCreatorClient } from '@smithii/sdk/sui/token-creator'
import { SuiSnapshotClient } from '@smithii/sdk/sui/snapshot'
```

**Never** import from the root barrel `@smithii/sdk` — it pulls in every chain's
peer deps. Always go through a subpath.

## Signer setup (Solana)

The SDK accepts any object that satisfies its `Signer` interface. Two common
sources:

```typescript
// Option A — from a private key (server / bot)
import { Keypair } from '@solana/web3.js'
import bs58 from 'bs58'
const keypair = Keypair.fromSecretKey(bs58.decode(process.env.PRIVATE_KEY!))
const signer = {
  publicKey: keypair.publicKey,
  signTransaction:     async (tx) => { tx.sign([keypair]); return tx },
  signAllTransactions: async (txs) => { txs.forEach((t) => t.sign([keypair])); return txs },
}

// Option B — from wallet-adapter (browser / dApp)
import { useWallet } from '@solana/wallet-adapter-react'
const { wallet } = useWallet()
const signer = wallet  // already implements the Signer interface
```

Never pass a private key string directly — wrap it in a `Signer`-shaped object.

For EVM, pass `viem.WalletClient`. For SUI, pass an object that implements
`SuiSigner` (`{ address, signAndExecute }`).

---

## Solana — Pump.fun bundler

Create a token on pump.fun and snipe it with up to **16 wallets** atomically
in the same block via Jito. The flow is **two-step**: upload metadata first,
then create+snipe with the returned `metadata` object plus a fresh `mintKeypair`.

```typescript
import { Connection, Keypair } from '@solana/web3.js'
import { PumpFunClient } from '@smithii/sdk/pump'

const client = new PumpFunClient({
  connection: new Connection(process.env.RPC_URL!),
  signer,
  jito: { uuid: process.env.JITO_UUID! },
  proxyUrl: process.env.PROXY_URL,
})

// 1. Upload metadata (image + socials) to pump.fun's IPFS endpoint
const metadata = await client.uploadMetadata({
  name: 'My Token',
  symbol: 'MTK',
  description: 'My token description',
  file: imageFile,                   // Blob | File
  twitter: 'https://x.com/mytoken',  // optional, may be null
  telegram: null,
  website: null,
})

// 2. Atomic create + dev buy + sniper bundle
const mintKeypair = Keypair.generate()
const result = await client.createAndSnipeToken({
  mintKeypair,
  metadata,
  devAmount: 0.5,                    // SOL the dev wallet buys
  buyers: [
    { pk: 'base58-priv-key-1', amount: 0.1 },
    { pk: 'base58-priv-key-2', amount: 0.2 },
    // up to 16 wallets
  ],
  isCashbackCoin: false,             // true → CREATE_V2 + Token-2022
  isTokenPregenerated: false,        // true charges 2x fee (vanity mint)
})

console.log('Mint:', mintKeypair.publicKey.toBase58())
console.log(result.createTxSignature, result.bundleIds, result.paymentSignature)
```

> The mint pubkey is `mintKeypair.publicKey` — the result object exposes
> `createTxSignature`, `buyerTxSignatures`, `bundleIds`, `paymentSignature`.

### Bundle buy / sell on existing token

Up to **25 wallets** per call, parallel `privKeys[]` and `amounts[]`. Atomic —
all succeed or all fail.

```typescript
import { PublicKey } from '@solana/web3.js'

const buyResult = await client.bundleSellBuy({
  mint: new PublicKey('TokenMintAddress'),  // PublicKey, not string
  action: 'buy',                             // 'buy' | 'sell'
  pool: 'pump',                              // 'pump' | 'pump-amm' | 'launchlab' | 'bonk'
  privKeys: ['wallet-1-pk', 'wallet-2-pk'],
  amounts:  [0.1, 0.15],                     // SOL per wallet, parallel array
})
```

For **graduated tokens (PumpSwap AMM)**, use `PumpSwapClient` (alias of
`PumpFunClient`) and pass `pool: 'pump-amm'`.

---

## Solana — Launchlab / Bonk bundler

Same API, different platform. Max **4 buyers** (Jito limit: 1 create + 4 buyers).
The metadata `uri` is pre-uploaded by the caller — `LaunchpadClient` does not
expose an upload helper.

```typescript
import { LaunchlabClient, BonkClient } from '@smithii/sdk/launchpad'

const launchlab = new LaunchlabClient({
  connection,
  signer,
  jito: { uuid: process.env.JITO_UUID! },
})

// LetsBonk.fun — same constructor, BONK_PLATFORM_ID baked in
const bonk = new BonkClient({
  connection,
  signer,
  jito: { uuid: process.env.JITO_UUID! },
})

const result = await launchlab.createAndSnipe({
  mintKeypair: Keypair.generate(),
  name: 'My Token',
  symbol: 'MTK',
  uri: 'https://your-cdn/metadata.json',     // pre-uploaded URI
  devAmount: 0.5,
  buyers: [
    { pk: 'pk-1', amount: 0.1 },
    // up to 4
  ],
  isTokenPregenerated: false,
})
console.log(result.mint, result.bundleId)    // mint is already a base58 string
```

---

## Solana — Moonit bundler

Max **3 buyers** on creation, **4 wallets** on bundle swap. Token metadata is
nested under `mint`; the icon is a base64 data string (not a File/Blob).

```typescript
import { MoonitClient } from '@smithii/sdk/moonit'
import { CurveType, MigrationDex } from '@heliofi/launchpad-common'

const client = new MoonitClient({
  connection,
  signer,
  jito: { uuid: process.env.JITO_UUID! },
})

// Create and snipe
const result = await client.createAndSnipe({
  mint: {
    name: 'My Token',
    symbol: 'MTK',
    icon: 'data:image/png;base64,…',          // base64 string, not File
    description: 'Description',
    curveType:    CurveType.CONSTANT_PRODUCT_V1,
    migrationDex: MigrationDex.RAYDIUM,
    links: [
      { url: 'https://x.com/mytoken',  label: 'twitter' },
      { url: 'https://t.me/mytoken',   label: 'telegram' },
    ],
  },
  devTokenAmount: 1_000_000n,                  // atomic-unit token amount, NOT SOL
  buyers: [
    { pk: 'pk-1', amount: 0.1 },               // amount IS in SOL here
    // up to 3
  ],
})

// Bundle buy/sell on existing token
const swap = await client.bundleSwap({
  mintAddress: 'TokenMintAddress',             // string, not PublicKey
  direction: 'BUY',                            // UPPERCASE: 'BUY' | 'SELL'
  buyers: [
    { pk: 'pk-1', amount: 0.1 },
    { pk: 'pk-2', amount: 0.2 },
    // up to 4
  ],
})
```

---

## Solana — Anti-MEV volume bot

Sandwichproof buy+sell bundles. The backend orchestrates the bundles —
this is *not* a client-side bundle.

```typescript
import { AntiMEVClient } from '@smithii/sdk/anti-mev'

const client = new AntiMEVClient({
  connection,
  signer,
  apiBaseUrl: process.env.BOTS_API_URL!,
  variant: 'pump',                             // 'pump' | 'pumpswap' | 'printr'
})

// Single-wallet — backend builds deposit tx, user signs, SDK polls confirmation
const single = await client.runSingle({
  tokenAddress: 'MintAddress',
  antiMEVUses: 10,                             // number of buy+sell cycles
  amount: { mode: 'fixed', fixedAmount: 0.05 }, // or { mode: 'random', randomMin, randomMax }
  delay:  { mode: 'fixed', fixedDelay: 2 },     // seconds (NOT ms)
  randomize: true,                              // optional
})
console.log(single.depositSignature)

// Multiple-wallet — pure backend orchestration with private keys
const multi = await client.runMultiple({
  tokenAddress: 'MintAddress',
  antiMEVUses: 5,
  privateKeys:    ['pk-1', 'pk-2'],
  privateAmounts: [0.05, 0.10],                 // parallel array, must match privateKeys.length
  randomMinDelay: 1,
  randomMaxDelay: 5,
})

// History
const history = await client.getHistory(signer.publicKey.toBase58())
```

Fee: **0.025 SOL per run per wallet**.

---

## Solana — Token Creator

Create an SPL token with airdrop + revokes in a single transaction.
TX size limit: **1232 bytes** — for airdrops > ~30 wallets, use
`MultiSenderClient` instead.

```typescript
import { TokenCreatorClient } from '@smithii/sdk/token-creator'

const client = new TokenCreatorClient({
  connection,
  signer,
  backend: { baseUrl: process.env.BOTS_API_URL! },
})

const result = await client.createToken({
  metadata: {
    name: 'My Token',
    symbol: 'MTK',
    description: 'Description',
    decimals: 6,
    supply: 1_000_000_000,
    socials: {
      website:  'https://mytoken.com',
      twitter:  'https://x.com/mytoken',
      telegram: 'https://t.me/mytoken',
    },
  },
  image: imageFile,                            // Blob | File
  isMutable: true,                             // true = LOCKED on-chain (SDK inverts)
  revokeMintAuth:   true,                      // NOT 'revokeMint'
  revokeFreezeAuth: true,                      // NOT 'revokeFreeze'
  walletDistributions: [                       // optional airdrop, percentages (sum ≤ 100)
    { address: 'wallet-1', percentage: 50 },
    { address: 'wallet-2', percentage: 30 },
  ],
})

console.log(result.mint.toBase58(), result.metadataUri, result.estimatedFeesSol)
```

> **`isMutable` is inverted on-chain.** Pass `true` to LOCK metadata
> (the SDK sends `!isMutable` to UMI's `createV1`). Charges a `RevokeMutability`
> fee when locking. Front-parity behavior — see token-creator/client.ts.

### Update metadata

```typescript
import { PublicKey } from '@solana/web3.js'

const updated = await client.updateMetadata({
  mint: new PublicKey('MintAddress'),
  metadata: {
    name: 'New Name',
    symbol: 'NEW',
    description: 'Updated',
  },
  image:  newImageFile,                        // optional; pass existingImageUri to reuse
  isMutable: true,                             // literal here (NOT inverted on update)
  chargeRevokeMutability: false,               // bill fee when transitioning mutable → immutable
})
```

---

## Solana — Token Manager

Post-launch operations on existing tokens.

```typescript
import { PublicKey } from '@solana/web3.js'
import { TOKEN_PROGRAM_ID } from '@solana/spl-token'
import { TokenManagerClient } from '@smithii/sdk/token-manager'

const client = new TokenManagerClient({ connection, signer })
const mint = new PublicKey('MintAddress')

// Revoke mint or freeze authority
await client.revokeAuthority({
  token: { mint, decimals: 6, tokenProgram: TOKEN_PROGRAM_ID },
  type: 'mint',                                // 'mint' | 'freeze'
})

// Increase supply (UI amount, SDK multiplies by 10**decimals)
await client.increaseSupply({
  token: { mint, decimals: 6, tokenProgram: TOKEN_PROGRAM_ID },
  amount: 100_000,
})

// Lock metadata (immutable). Requires the current metadata snapshot.
await client.makeImmutable({
  token: { mint, decimals: 6, tokenProgram: TOKEN_PROGRAM_ID },
  metadata: {
    name:        'My Token',
    symbol:      'MTK',
    uri:         'https://your-cdn/metadata.json',
    description: 'Description',
    updateAuthority: signer.publicKey,         // null/undefined → SDK uses createV1
  },
})

// Snapshot SPL token holders (skips USDC/USDT/WSOL via SNAPSHOT_BLOCKED_MINTS)
const { holders, signature } = await client.snapshotTokenHolders({
  token: { mint, decimals: 6, tokenProgram: TOKEN_PROGRAM_ID },
})

// Snapshot NFT collection holders (requires Helius DAS-enabled RPC)
const nftSnap = await client.snapshotNftCollectionHolders({
  creatorAddress: 'CollectionCreatorAddress',
  rpcUrl: process.env.HELIUS_RPC_URL!,
  pageSize: 1000,                              // optional, default 1000
})
console.log(nftSnap.mints, nftSnap.owners, nftSnap.signature)
```

---

## Solana — Multisender

For airdrops > ~30 wallets. Batches into multiple txs automatically.

```typescript
import { PublicKey } from '@solana/web3.js'
import { TOKEN_PROGRAM_ID } from '@solana/spl-token'
import { MultiSenderClient } from '@smithii/sdk/multisender'

const client = new MultiSenderClient({
  connection,
  signer,
  backend: { baseUrl: process.env.BOTS_API_URL! },
  lookupTable: process.env.LUT_ADDRESS
    ? new PublicKey(process.env.LUT_ADDRESS)
    : undefined,
})

// Send now
const result = await client.sendAirdrop({
  token: {                                     // null = native SOL
    mint: new PublicKey('MintAddress'),
    decimals: 6,
    tokenProgram: TOKEN_PROGRAM_ID,
  },
  amountMode: 'same',                          // 'same' | 'different'
  defaultAmount: 100,                          // when 'same' — NOT 'amount'
  wallets: [
    { wallet: 'recipient-1' },                 // field is 'wallet' NOT 'address'
    { wallet: 'recipient-2' },
    // when amountMode='different': { wallet: '...', amount: 50 }
  ],
  saveDetailsToBackend: true,                  // optional, default true
})
console.log(result.totalRecipients, result.failedWallets)

// Schedule (SPL only, no native SOL)
const scheduled = await client.scheduleAirdrop({
  /* same SendAirdropArgs */
  scheduledAt: new Date(Date.now() + 60 * 60 * 1000),
} as never)
console.log(scheduled.airdropId)

// History
const history = await client.getHistory(signer.publicKey)
```

---

## Solana — Vesting & Claim

```typescript
import { PublicKey } from '@solana/web3.js'
import { TokenVestingClient } from '@smithii/sdk/token-vesting'
import { TokenClaimClient }   from '@smithii/sdk/token-claim'

// === Vesting ===
const vesting = new TokenVestingClient({ connection, signer })

const init = await vesting.initVesting({
  vesting: {
    tokenMint: { pubKey: 'MintAddress', decimals: 6 },
    totalAmount: 1_000_000,
    lockingEndDate: new Date('2026-06-30T00:00:00Z'),
    periods: [
      { endDate: new Date('2026-05-30T00:00:00Z'), percentage: 50 },
      { endDate: new Date('2026-06-30T00:00:00Z'), percentage: 50 },
      // percentages must sum to 100
    ],
    receivers: [],                             // empty = public; non-empty → merkle-gated
  },
})
console.log(init.vestingPda.toBase58())

await vesting.claimVesting({
  vestingAuthority: signer.publicKey,          // who created the vesting
  mint: new PublicKey('MintAddress'),
  receivers: [],                               // same list passed at init
})

// === Token Claim ===
const claim = new TokenClaimClient({ connection, signer })

const initClaim = await claim.initialize({
  info: {
    tokenMint: { pubKey: 'MintAddress', decimals: 6 },
    quantityPerWallet: 100,
    receivers: ['receiver-1', 'receiver-2'],   // MUST be non-empty
    endDate: new Date('2026-06-30T00:00:00Z'),
  },
})

await claim.claim({
  claimAuthority: signer.publicKey,
  mint: new PublicKey('MintAddress'),
  receivers: ['receiver-1', 'receiver-2'],
})
```

---

## Solana — Market Maker

```typescript
import { LAMPORTS_PER_SOL } from '@solana/web3.js'
import { MarketMakerClient } from '@smithii/sdk/market-maker'

const client = new MarketMakerClient({ connection, signer })

const tradable = await client.isTokenTradable('MintAddress')
if (tradable) {
  const { signature } = await client.deposit({
    depositLamports: 0.5 * LAMPORTS_PER_SOL,   // SOL deposited to the bot's vault
    feeSol: 0.025,                              // SOL fee paid to Smithii
  })
  console.log(signature)
}
```

---

## Solana — Mantis (launchpad)

```typescript
import { PublicKey } from '@solana/web3.js'
import { MantisClient, PaymentMethod } from '@smithii/sdk/mantis'

const client = new MantisClient({ connection, signer })

// Initialize a launch (signer is overridden as authority)
const launch = await client.initialize({
  launch: {
    mint: new PublicKey('MintAddress'),
    mintDecimals: 6,
    hardcap: 100,                              // UI units of the payment currency
    softcap: 50,
    paymentMethod: PaymentMethod.SOL,
    whitelistPhase: {
      price: 0.001,
      startDate: Math.floor(Date.now() / 1000),
      endDate:   Math.floor(Date.now() / 1000) + 86_400,
      minAmount: 0.01,
      maxAmount: 1,
    },
    whitelistLimit: 100,
    publicPhase: {
      price: 0.002,
      startDate: Math.floor(Date.now() / 1000) + 86_400,
      endDate:   Math.floor(Date.now() / 1000) + 86_400 * 2,
      minAmount: 0.01,
      maxAmount: 5,
    },
  },
})

// Buy with SOL / USDC / USDT
await client.buy({
  launchAuthority: signer.publicKey,           // who initialized the launch
  mint: new PublicKey('MintAddress'),
  paymentMethod: PaymentMethod.USDC,
  amount: 100,
})

// Claim, withdraw, edit, getLaunchState, getUserLaunchState — see api-reference.md
```

---

## Solana — Payment (read-only)

Read user's plan, referrals, and accumulated fees from the on-chain
payment program.

```typescript
import { PaymentClient, PlanEnum } from '@smithii/sdk/payment'

const client = new PaymentClient({ connection })

const tools = await client.getTools()
console.log(tools.prices, tools.accumulatedFees)

const referral = await client.getReferral(userPubkey)
console.log(referral.referrer, referral.totalAmount, referral.alreadyUsed)

const plan = await client.getPlan(userPubkey)
// plan.plan is PlanEnum (numeric) | null
if (plan.plan === PlanEnum.OGPass) { /* … */ }
```

---

## EVM — Token Creator

Supports Base, Ethereum, BSC, Polygon, Arbitrum, Avalanche, Blast.

The factory takes a pre-compiled ERC20 template `bytecode` plus the constructor
args. Bytecode is shipped separately (download from your build artifacts) — the
SDK does not bundle it because it would balloon every install.

```typescript
import { createWalletClient, http, type Hex } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { base } from 'viem/chains'
import { EvmTokenCreatorClient } from '@smithii/sdk/evm/token-creator'
import { EvmRegistry } from '@smithii/sdk/evm/core'

const account = privateKeyToAccount(process.env.PRIVATE_KEY as Hex)
const walletClient = createWalletClient({ account, chain: base, transport: http() })

const client = new EvmTokenCreatorClient({ walletClient })
const registry = new EvmRegistry()
const antiBot   = registry.getOrThrow(base.id, 'antiBot').address!
const antiWhale = registry.getOrThrow(base.id, 'antiWhale').address!

const { tokenAddress, hash } = await client.deployErc20({
  projectId: '0x…32-byte hex',                 // bytes32, REQUIRED — from indexer
  bytecode:  '0x…compiled-template…',          // Hex, REQUIRED — your template build
  tokenInfo: {
    name:   'My Token',
    symbol: 'MTK',
    decimals: 18,
    supply: 1_000_000,                         // UI amount; SDK scales by 10**decimals
    tax: 5,                                    // 0–20 (%)
    taxReceiver: account.address,
    airdrop: false,
  },
  antiBot,                                     // module ADDRESS, not a boolean
  antiWhale,
  exempt: [],                                  // wallets exempt from tax/limits
})
console.log(tokenAddress, hash)
```

## EVM — Multisender

```typescript
import { EvmMultisenderClient } from '@smithii/sdk/evm/multisender'

const client = new EvmMultisenderClient({ walletClient })

const { hash, totalAmount, totalFee } = await client.airdrop({
  token:     '0xTokenAddress',
  projectId: '0x…32-byte hex',
  wallets: [
    { address: '0x…' },
    { address: '0x…' },
  ],
  amount: 100,                                 // number, NOT bigint (SDK scales via parseEther)
  amountMode: 'same',                          // 'same' | 'different'
})
```

## EVM — Snapshot

Pays via the Smithii payments contract (`payService`) before scanning Transfer
events — same flow as the front. `projectId` is required.

```typescript
import { createPublicClient, http } from 'viem'
import { base } from 'viem/chains'
import { EvmSnapshotClient } from '@smithii/sdk/evm/snapshot'

const publicClient = createPublicClient({ chain: base, transport: http() })

const client = new EvmSnapshotClient({ walletClient, publicClient })

const erc20 = await client.snapshotErc20Holders({
  token: '0xTokenAddress',
  projectId: '0x…32-byte hex',                 // REQUIRED
  // optional: fromBlock, toBlock, chunkSize (default 2500)
})
console.log(erc20.holders, erc20.paymentHash)

const erc721 = await client.snapshotErc721Holders({
  collection: '0xCollectionAddress',
  projectId: '0x…32-byte hex',                 // REQUIRED
  // optional: chunkSize (default 30000)
})
```

---

## SUI — Token Creator

Single call covers fee + publish + configure. The fee tx (default 7.5 SUI)
runs first; if it fails, publish never runs.

```typescript
import { SuiTokenCreatorClient } from '@smithii/sdk/sui/token-creator'

const client = new SuiTokenCreatorClient({
  signer:  yourSuiSigner,                      // implements SuiSigner
  network: 'mainnet',                          // 'mainnet' | 'testnet' | 'devnet' | raw RPC URL
})

const result = await client.deploy({
  config: {
    name: 'My Token',                          // ≤ 32 ASCII chars
    symbol: 'MTK',                             // 2-8 ASCII chars
    decimals: 6,                               // 0-12
    description: 'Description',                // ≤ 320 ASCII chars
    iconUrl: 'https://…',                      // ≤ 320 ASCII chars; '' = no icon
    initialSupply: 1_000_000n,                 // bigint, atomic units NOT scaled by SDK
    isMintable: false,                         // false = burns the TreasuryCap
  },
})

console.log(
  result.coinType,                             // '0xpkg::module::SYMBOL'
  result.coinAddress,                          // CoinMetadata object id
  result.treasuryAddress,                      // TreasuryCap object id (or burned)
  result.digest,                               // publish digest
  result.paymentDigest,                        // fee tx digest
)
```

> The minted supply goes to `signer.address` automatically. Use the
> `feeMist` arg to override the fee amount; `skipPayment` is dev-only.

## SUI — Snapshot

Backend-driven. The `save*` endpoints validate a payment digest server-side —
see your backend's payment policy.

```typescript
import { SuiSnapshotClient } from '@smithii/sdk/sui/snapshot'

const client = new SuiSnapshotClient({ apiBaseUrl: process.env.BOTS_API_URL! })

const tokenHolders = await client.getTokenHolders(
  '0xfa7ac3951fdca…::BLUB::BLUB',              // coin type tag
  'BLUB',
)
const nftHolders = await client.getNftCollectionHolders('collection-slug')

// Persist snapshot on the backend (requires payment digest)
const saved = await client.saveTokenSnapshot({
  accountAddress: '0xUserWallet',
  token: { name: 'BLUB', coinType: '0x…::BLUB::BLUB', symbol: 'BLUB', imageUrl: '' },
  payment: { digest: 'tx-digest', amount: 0.1, rpcUrl: 'https://…' },
})
```

---

## Error handling

All SDK methods throw subclasses of `SmithiiError`:

```typescript
import {
  SmithiiError,
  BundleError,
  TransactionFailedError,
  ValidationError,
} from '@smithii/sdk/core'

try {
  const result = await client.createAndSnipeToken({ /* … */ })
} catch (err) {
  if (err instanceof BundleError) {
    console.error('Bundle failed', err.bundleId, err.message)
  } else if (err instanceof TransactionFailedError) {
    console.error('TX failed', err.signature, err.onChainError)
  } else if (err instanceof ValidationError) {
    console.error('Invalid input', err.message)
  } else if (err instanceof SmithiiError) {
    console.error('SDK error', err.message)
  } else {
    throw err
  }
}
```

| Class | Thrown when |
|-------|-------------|
| `ConfigError` | Missing/invalid constructor config |
| `ValidationError` | Invalid argument at API boundary |
| `HttpError` | Backend HTTP failure (`status`, `url`, `body`) |
| `RpcError` | Solana RPC failure |
| `BundleError` | Jito bundle failed (`bundleId?`) |
| `TransactionFailedError` | On-chain tx failed (`signature`, `onChainError`) |
| `TransactionTimeoutError` | Confirmation timeout (`signature`, `attempts`) |

---

## On-chain invariants — never bypass

- **Payment is enforced by the SDK.** Single-tx tools bundle it atomically;
  bundler tools (Pump / Launchpad / Moonit) send it after the bundle confirms.
  Never call payment manually.
- **Pump.fun token math:** `tokenAmount = floor(solAmount * 35_707_140 * 1e6)`,
  `maxSolCost = solAmount * 1.1 * 1e9`.
- **Slippage values are deliberate:** Pump buys 10%, Pump sells 0% min-out,
  Moonit first buy 25% / swaps 10%, Raydium launchpad creation 99.99%.
- **Jito limits:** max 5 txs per bundle, 500_000 lamports tip on first tx,
  compute price 200_000 microLamports.
- **Wallet caps per call:** Pump 16/25, Launchpad 4, Moonit 3/4 — derived from
  Jito's 5-tx-per-bundle limit.
- **TokenCreator `isMutable` is inverted on-chain at create time.** SDK forwards
  `!isMutable` to UMI's `createV1`. Charges `RevokeMutability` when locking.
  The update path is *not* inverted (literal semantic).
- **TX size cap:** 1232 bytes. `TokenCreatorClient` validates upfront — large
  airdrops fail; use `MultiSenderClient`.

Full invariants reference in [`resources/invariants.md`](./resources/invariants.md).

---

## Environment variables

The SDK never reads `process.env` — all config is passed via constructor.
A typical app sets:

| Variable | Required | Purpose |
|----------|----------|---------|
| `RPC_URL` | Yes (Solana) | Solana RPC endpoint |
| `PRIVATE_KEY` | Yes (server) | Wallet secret key (base58 for Solana, 0x-hex for EVM) |
| `JITO_UUID` | Yes (bundlers) | Jito Block Engine auth token |
| `PROXY_URL` | Optional | Smithii proxy for Jito calls (defaults to canonical) |
| `BOTS_API_URL` | Yes (anti-mev, multisender, token-creator, market-maker, sui) | Smithii backend |
| `LUT_ADDRESS` | Optional | Default Address Lookup Table (multisender) |
| `HELIUS_RPC_URL` | If snapshotting NFTs on Solana | DAS-enabled RPC |

---

## When to use which client

| User wants… | Use |
|---|---|
| Pump.fun token + snipe (≤16 wallets) | `PumpFunClient.createAndSnipeToken` |
| Buy/sell pump.fun token from many wallets (≤25) | `PumpFunClient.bundleSellBuy` |
| Same on a graduated PumpSwap token | `PumpSwapClient` (alias) with `pool: 'pump-amm'` |
| Raydium LaunchLab launch (≤4 wallets) | `LaunchlabClient.createAndSnipe` |
| LetsBonk.fun launch (≤4 wallets) | `BonkClient.createAndSnipe` |
| Moonit launch (≤3 wallets) | `MoonitClient.createAndSnipe` |
| Sandwichproof volume bot | `AntiMEVClient` |
| SPL token + airdrop + revokes (≤30 wallets) | `TokenCreatorClient.createToken` |
| Airdrop > 30 wallets | `MultiSenderClient.sendAirdrop` |
| Schedule airdrop for later | `MultiSenderClient.scheduleAirdrop` |
| Revoke authority / lock metadata | `TokenManagerClient` |
| Holder snapshot | `TokenManagerClient.snapshotTokenHolders` |
| NFT collection snapshot | `TokenManagerClient.snapshotNftCollectionHolders` |
| Vesting schedule | `TokenVestingClient` |
| Claim flow | `TokenClaimClient` |
| Market making | `MarketMakerClient.deposit` |
| Smithii's own launchpad | `MantisClient` |
| User's plan/referrals/fees | `PaymentClient` (read-only) |
| EVM token | `EvmTokenCreatorClient` |
| EVM airdrop | `EvmMultisenderClient` |
| EVM holder snapshot | `EvmSnapshotClient` |
| SUI token | `SuiTokenCreatorClient` |
| SUI snapshot | `SuiSnapshotClient` |

## Common mistakes

- ❌ Importing from `@smithii/sdk` (root barrel) — always use a subpath
- ❌ Passing a private key string as `signer` without wrapping it in a `Signer`-shaped object
- ❌ Calling `pump.createAndSnipeToken` with flat metadata fields — first call `uploadMetadata`, then pass `mintKeypair` + the returned `metadata` object
- ❌ Passing strings where the SDK wants `PublicKey` (e.g. `bundleSellBuy.mint`, `revokeAuthority.token.mint`)
- ❌ Using `'buy' | 'sell'` for Moonit — Moonit's `direction` is uppercase: `'BUY' | 'SELL'`
- ❌ Calling a payment instruction manually — every client charges it internally
- ❌ Retrying a failed bundler call without re-fetching state — bonding curve may have moved
- ❌ Treating `bundleSellBuy` like a swap loop — it's atomic across all wallets
- ❌ Using `TokenCreatorClient` for airdrops > 30 wallets — TX size is capped at 1232 bytes
- ❌ Double-inverting `isMutable` for `TokenCreatorClient.createToken` — the SDK already inverts it at create time (the update path is literal)
- ❌ Forgetting `projectId` on EVM `airdrop` / `snapshot*` / `deployErc20` — the on-chain payments contract requires it

## Additional resources

- Full API surface: [`resources/api-reference.md`](./resources/api-reference.md)
- Invariants reference: [`resources/invariants.md`](./resources/invariants.md)

## Links

- **Smithii**: https://smithii.io
- **Smithii Tools (no-code)**: https://tools.smithii.io
- **SDK on npm**: https://www.npmjs.com/package/@smithii/sdk
- **SDK GitHub**: https://github.com/SmithiiDev/smithii-tools-sdk
- **Discord**: https://discord.gg/3AFfGDfmk7
- **Twitter / X**: https://x.com/Smithii_io

---
> Source: [SmithiiDev/smithii-sdk-skill](https://github.com/SmithiiDev/smithii-sdk-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
