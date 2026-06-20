## obscura

> This skill is derived from building **Obscura** - a confidential RFQ dark pool on Zama FHEVM, deployed live on Sepolia at [obscuradex.vercel.app](https://obscuradex.vercel.app). 21-day sprint, 47 contract tests passing, full encrypted match-and-settle proven on-chain from a browser.

# Zama FHEVM Builder Skill - Confidential dApp Survival Kit

> Agent-usable playbook for shipping a confidential application on Zama FHEVM.
> Stack: Solidity FHEVM ops, OpenZeppelin ERC-7984, Vue 3 + Vite, Wagmi-Vue, viem, `@zama-fhe/relayer-sdk`.

---

## Context

This skill is derived from building **Obscura** - a confidential RFQ dark pool on Zama FHEVM, deployed live on Sepolia at [obscuradex.vercel.app](https://obscuradex.vercel.app). 21-day sprint, 47 contract tests passing, full encrypted match-and-settle proven on-chain from a browser.

Source repo: [github.com/unifyWeb3/obscura-monorepo](https://github.com/unifyWeb3/obscura-monorepo)

Every trap in this document came from a real bug, a real wasted hour, or a real production failure. The format is intentional: **Detection → Action → Why it works**, so an AI agent (or developer) can pattern-match against symptoms and apply the fix without re-deriving the underlying reasoning.

---

## Skill overview

This skill enables an agent or developer to:

1. Bootstrap a Vue 3 + Wagmi + relayer-sdk frontend that talks to FHEVM contracts on Sepolia
2. Encrypt user inputs client-side, submit them on-chain with ZK proofs, decrypt results via threshold KMS
3. Avoid the 23 documented integration traps that consume days of debugging time
4. Deploy to Vercel with the cross-origin isolation headers the relayer-sdk requires
5. Make confident architectural decisions (when to self-relay vs operator-relay, when to use plaintext vs ciphertext, how to expose state to the user)

---

## When to use this skill

Apply this skill when:

- Building any dApp where **user inputs need to stay private** during on-chain processing (RFQ, dark pools, sealed-bid auctions, private voting, confidential payments)
- The frontend stack is Vue 3 (with Wagmi-Vue) - most React-first FHEVM tutorials do NOT translate cleanly
- You're integrating `@zama-fhe/relayer-sdk` for browser-side encryption and threshold KMS public decryption
- You're deploying to Vercel or any static host where you control HTTP headers
- The contract uses ERC-7984 confidential token transfers as part of settlement

Do NOT use this skill for:

- Pure smart contract work without a frontend (skip the Wagmi/Vite traps)
- React-based dApps (the Vue-specific patterns will mislead you; the relayer-sdk traps still apply)
- FHEVM mainnet deployments (this skill targets Sepolia testnet - production has additional KMS configuration concerns)

---

## Core stack

| Layer               | Tool                                | Version         | Why this choice                                                    |
| ------------------- | ----------------------------------- | --------------- | ------------------------------------------------------------------ |
| Smart contracts     | Solidity + FHEVM Solidity SDK       | `0.11.x`        | Required for FHE ops                                               |
| Confidential tokens | OpenZeppelin Confidential Contracts | `0.4.x`         | ERC-7984 reference impl                                            |
| Contract testing    | Hardhat + FHEVM mock                | latest          | Mock skips KMS, runs in CI                                         |
| Frontend framework  | Vue 3.5                             | latest          | TypeScript-first, smaller bundles than React for crypto-heavy apps |
| Build tool          | Vite 8                              | latest          | Required for relayer-sdk WASM workers                              |
| Wallet & contracts  | `@wagmi/vue` + `viem`               | `0.5.x` / `2.x` | Vue-native bindings around viem                                    |
| Encryption + KMS    | `@zama-fhe/relayer-sdk`             | `0.4.x`         | Web entry - browser-side encryption + ZK proofs + threshold KMS    |
| Styling             | Tailwind CSS v4                     | latest          | Custom design tokens via `@theme`                                  |
| Deployment          | Vercel                              | latest          | Native Vite preset, configurable headers                           |

---

## Mental models

### How FHEVM actually behaves

1. **Plaintext never leaves the browser.** The relayer-sdk encrypts user inputs locally using the FHEVM public key + generates a ZK proof of the encryption. The contract receives ciphertext handles + the proof.

2. **The contract operates on ciphertext.** `euint64`, `euint32`, `ebool` are encrypted types. Comparisons (`FHE.gt`, `FHE.lt`, `FHE.eq`) return encrypted booleans. Conditional logic uses `FHE.select(cond, ifTrue, ifFalse)` - both branches always compute, only the result is selected obliviously.

3. **No actor sees the cleartexts during computation.** Not the operator, not the contract author, not any node operator. The encrypted handles are processed by the FHEVM coprocessor.

4. **Decryption is deliberate.** Cleartexts are revealed only when the contract explicitly calls `FHE.makePubliclyDecryptable(handle)` - typically as part of a settlement flow. The threshold KMS (multiple independent signers) then signs cleartexts so the contract can verify them.

5. **Decryption is verifiable.** When the user sends decrypted values back to the contract, `FHE.checkSignatures(handles, cleartexts, decryptionProof)` must pass before the contract acts on them. **This call must be the FIRST line of any settlement function** - otherwise an attacker can pass forged cleartexts.

### The lifecycle of an encrypted operation

```
Browser:                          Chain:                       KMS:
[plaintext]
    │
    ├─ relayer-sdk.encrypt()
    │  produces: handles[] + ZK proof
    │
    ▼
[ciphertext + proof]
    │
    ├─ writeContract(handles, proof)
    │
    ▼                              [ciphertext stored]
                                       │
                                       ├─ FHE ops happen
                                       │  (gt, select, etc.)
                                       │
                                       ├─ FHE.makePubliclyDecryptable(handle)
                                       │
                                       ▼
                                   [handle marked]──────────────►[KMS signs cleartexts]
                                                                      │
                                                                      ▼
[publicDecrypt(handles)]◄─────────────────────────────────────[cleartexts + signatures]
    │
    ├─ writeContract(matchId, cleartexts, decryptionProof)
    │
    ▼                              [FHE.checkSignatures ✓]
                                       │
                                       ├─ abi.decode(cleartexts, types)
                                       │
                                       └─ business logic with verified plaintext
```

---

## Build flow (deterministic)

Follow this order. Skipping steps creates compounding bugs.

### 1. Contracts first, frontend last

Write Solidity. Get tests passing on Hardhat FHEVM mock. Deploy + verify on Sepolia. Confirm a manual roundtrip works (Hardhat script that does encrypt → submit → match → release → decrypt → settle). **Only then** touch the frontend.

If the frontend hits a bug, you need to know whether it's a contract issue or an integration issue. Working contracts + working manual roundtrip means every frontend bug is purely an integration problem.

### 2. Frontend scaffold in this exact order

```bash
pnpm create vite@latest app -- --template vue-ts
cd app
pnpm add @wagmi/vue@^0.5 @wagmi/core@^3 @wagmi/connectors viem@^2
pnpm add @zama-fhe/relayer-sdk@^0.4
pnpm add @tanstack/vue-query
pnpm add -D tailwindcss@next @tailwindcss/vite vue-router
```

Then before writing any pages:

1. Configure Vite COOP/COEP headers + `optimizeDeps.exclude` (Trap 14)
2. Set up `tsconfig.app.json` paths block (Trap 21)
3. Wire WagmiPlugin BEFORE VueQueryPlugin in `main.ts` (Trap 11)
4. Create `useFhevm` composable (singleton SDK instance)
5. Test SDK boots - green dot before writing any submission UI

### 3. Encryption flow - the canonical pattern

```typescript
import { toHex } from "viem";
import { useFhevm } from "@/composables/useFhevm";

const { ensureReady } = useFhevm();
const instance = await ensureReady();

const encrypted = await instance
  .createEncryptedInput(contractAddress, userAddress)
  .add64(value1)
  .add64(value2)
  .add64(value3)
  .encrypt();

// CRITICAL: always wrap handles + proof in toHex()
await writeContractAsync({
  address: contractAddress,
  abi,
  functionName: "submitSomething",
  args: [
    toHex(encrypted.handles[0]),
    toHex(encrypted.handles[1]),
    toHex(encrypted.handles[2]),
    plaintextArg,
    toHex(encrypted.inputProof),
  ],
});
```

### 4. Decryption flow - also canonical

```typescript
const result = await instance.publicDecrypt([
  encryptedHandle1,  // 0x... bytes32 string from contract read
  encryptedHandle2,
  encryptedHandle3,
  encryptedHandle4,
]);

// SDK pre-encodes for FHE.checkSignatures - direct passthrough
await writeContractAsync({
  ...,
  functionName: "settleMatch",
  args: [
    matchId,
    result.abiEncodedClearValues,  // bytes for cleartexts arg
    result.decryptionProof,         // bytes for decryptionProof arg
  ],
});
```

### 5. Reading state - use `viem/actions`, not Wagmi-Vue hooks

```typescript
import { useClient } from "@wagmi/vue";
import { getLogs, readContract, waitForTransactionReceipt } from "viem/actions";
import { parseAbiItem } from "viem";

const client = useClient();
const eventDef = parseAbiItem(
  "event Something(uint256 indexed id, address indexed actor)",
);

const logs = await getLogs(client.value, {
  address: poolAddr,
  event: eventDef,
  fromBlock: DEPLOY_BLOCK,
  toBlock: "latest",
});
```

---

## Critical traps & resolutions

Each trap is structured **Detection → Action → Why it works**. Apply this format mentally when debugging - your symptom is the detection, the action is the resolution, the why prevents future recurrence.

---

### Trap 1 - Vite + relayer-sdk requires cross-origin isolation

**Detection:**

- SDK status stays "booting" forever
- Console: `This browser does not support threads` warning
- OR cryptic `Bad JSON` errors during `createInstance()`

**Action:**

In `vite.config.ts`:

```typescript
export default defineConfig({
  plugins: [vue(), tailwindcss()],
  server: {
    headers: {
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
  optimizeDeps: {
    exclude: ["@zama-fhe/relayer-sdk"],
  },
});
```

For Vercel production, additionally create `vercel.json`:

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "Cross-Origin-Opener-Policy", "value": "same-origin" },
        { "key": "Cross-Origin-Embedder-Policy", "value": "require-corp" }
      ]
    }
  ],
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

**Why it works:**
The relayer-sdk uses `SharedArrayBuffer` for FHE math threading. Browsers refuse to expose `SharedArrayBuffer` unless the page is cross-origin-isolated, which requires both COOP and COEP headers. The `optimizeDeps.exclude` prevents Vite's pre-bundler from breaking the SDK's worker loading paths. Without both, the SDK can't initialize its worker pool.

---

### Trap 2 - Multiple browser wallet extensions fight over `window.ethereum`

**Detection:**

- Console errors: `Cannot redefine property: ethereum`, `MetaMask encountered an error setting the global Ethereum provider`, `Cannot redefine property: isZerion`
- Encryption succeeds but `writeContract` fails with `hex_._.replace is not a function` or similar string-method errors on what should be addresses
- User has multiple wallet extensions installed (Zerion + MetaMask + Coinbase Wallet + Rabby + Phantom etc.)

**Action:**
Direct the user to disable all wallet extensions except one in `chrome://extensions/`. Specifically for Brave: also set `brave://settings/wallet` "Default Ethereum wallet" to `Extensions (no fallback)`. After disabling, restart the Vite dev server and hard-refresh.

In production, add a defensive check on SDK boot:

```typescript
if (typeof window.ethereum !== "undefined") {
  const providers = (window.ethereum as any).providers;
  if (Array.isArray(providers) && providers.length > 1) {
    console.warn(
      "[Obscura] Multiple wallet extensions detected. May cause provider conflicts. Disable extras.",
    );
  }
}
```

**Why it works:**
Each wallet extension races to set `window.ethereum` at page load. The second one fails because the property is non-configurable after the first setter. The relayer-sdk gets a corrupted provider reference and downstream calls explode on string operations against malformed wallet state. There's no programmatic fix - multi-extension is fundamentally broken for SDK consumers.

---

### Trap 3 - Uint8Array vs hex string at the relayer-sdk → viem boundary

**Detection:**

- Encryption succeeds (handles array populated, inputProof present)
- `writeContractAsync` immediately fails with `hex_.replace is not a function` or `Cannot read properties of undefined (reading 'replace')`
- Logging the handles shows `Uint8Array(32)` not `0x...` strings

**Action:**

Always wrap handles and inputProof in `toHex()` from viem before passing to `writeContractAsync`:

```typescript
import { toHex } from "viem";

await writeContractAsync({
  ...,
  args: [
    toHex(encrypted.handles[0]),  // not encrypted.handles[0]
    toHex(encrypted.handles[1]),
    toHex(encrypted.inputProof),  // not encrypted.inputProof
  ],
});
```

**Why it works:**
The relayer-sdk returns handles as `Uint8Array(32)` and `inputProof` as `Uint8Array(N)`. Viem's ABI encoder for `bytes32` and `bytes` types calls `.replace()` on the input expecting a hex string. `Uint8Array` has no `.replace` method. `toHex()` converts cleanly.

This is the most common production-blocking bug in any relayer-sdk → Wagmi/viem integration. Apply it preemptively.

---

### Trap 4 - Wagmi-Vue API ≠ Wagmi React API

**Detection:**

- Compile errors: `Module '@wagmi/vue' has no exported member 'usePublicClient'`
- React Wagmi tutorials don't work copy-pasted
- `publicClient.value.getLogs is not a function`

**Action:**

In Wagmi-Vue, use `useClient` (not `usePublicClient`). The returned client is a **plain viem `Client`** without public actions attached. Read methods must be imported from `viem/actions`:

```typescript
import { useClient } from "@wagmi/vue";  // NOT usePublicClient
import { getLogs, readContract, waitForTransactionReceipt } from "viem/actions";

const client = useClient();
const logs = await getLogs(client.value, { ... });  // not client.value.getLogs(...)
```

**Why it works:**
React Wagmi attaches public actions to the returned client by convention. Vue Wagmi returns the base viem `Client` to keep the API surface small and tree-shakable. The `viem/actions` exports are first-class, equally typed, and intentional.

---

### Trap 5 - Public RPC blocks wide `eth_getLogs` queries

**Detection:**

- Composable that reads event history fails with "RPC Request failed"
- Default Wagmi-Vue config uses public Sepolia RPC
- The `fromBlock → toBlock` range is more than ~1000 blocks

**Action:**

Use a real RPC provider. Add to `.env.local`:

```bash
VITE_INFURA_API_KEY=your_infura_key_here
```

In `wagmi.ts`:

```typescript
import { http, createConfig } from "@wagmi/vue";
import { sepolia } from "@wagmi/vue/chains";

const INFURA_KEY = import.meta.env.VITE_INFURA_API_KEY as string | undefined;

const sepoliaTransport = INFURA_KEY
  ? http(`https://sepolia.infura.io/v3/${INFURA_KEY}`)
  : http();

export const config = createConfig({
  chains: [sepolia],
  transports: { [sepolia.id]: sepoliaTransport },
  connectors: [injected(), metaMask()],
});
```

**Why it works:**
Public RPCs rate-limit aggressively and reject `eth_getLogs` over wide block ranges to protect themselves. Infura, Alchemy, QuickNode etc. handle 100k+ block scans without issue. Any production dApp doing event-log queries needs a real RPC URL.

---

### Trap 6 - Browser wallet extensions resolved (`hex_.replace` deep-stack errors)

**Detection:**

- After fixing Traps 2 + 3, encryption succeeds AND wallet provider is clean BUT a deep-stack `hex_.replace` error still fires
- The error trace points into the SDK internals, not your code

**Action:**

Verify the `mac/contract address` argument to `createEncryptedInput()` is a valid `0x...` string (not a `Uint8Array`, not an `address` object). If you're getting the address from `useAccount()`, it's already a string - but if you're constructing it from `ADDRS[chainId.value]`, ensure the address strings in your contracts module are quoted hex strings, not raw bytes.

**Why it works:**
The SDK calls `.replace()` internally on the contract + user address args to normalize them. If those strings are malformed, the same cryptic error fires. Always pass plain `0x...` hex strings; never pass viem `Address` types blindly through other transforms.

---

### Trap 7 - `setOperator` takes `uint48`, not `uint64`

**Detection:**

- Looking at OpenZeppelin Confidential Contracts source for `ERC7984.sol`, the function is `setOperator(address operator, uint48 until)`
- Tutorials and older docs may show `uint64`

**Action:**

When calling `setOperator` from the frontend or in tests, the expiry param's type is `uint48`. JavaScript bigint coercion handles this fine - `2^48 - 1 = 281474976710655` seconds since epoch is year 8923+, plenty of headroom.

```typescript
const expiry = BigInt(Math.floor(Date.now() / 1000) + 30 * 24 * 3600); // 30 days
await writeContractAsync({
  address: tokenAddr,
  abi: erc7984Abi,
  functionName: "setOperator",
  args: [poolAddr, expiry],
});
```

**Why it works:**
`uint48` is enough for centuries of timestamps. The narrower type saves storage. Viem ABI encoder respects the ABI's declared type - passing a value within range works regardless of whether you typed it as `uint48` or `uint64` in your application code.

---

### Trap 8 - `publicDecrypt` ↔ `settleMatch` mapping is direct

**Detection:**

- Documentation is sparse on how `PublicDecryptResults` maps to a contract function expecting `bytes cleartexts, bytes decryptionProof`
- Temptation to manually `abi.encode` the cleartexts in JavaScript

**Action:**

**Don't manually encode anything.** The SDK does it for you. Map fields 1-to-1:

```typescript
const result = await instance.publicDecrypt([handle1, handle2, handle3, handle4]);

await writeContractAsync({
  ...,
  functionName: "settleMatch",
  args: [
    matchId,
    result.abiEncodedClearValues,  // already abi.encoded bytes - direct passthrough
    result.decryptionProof,         // KMS signatures + threshold metadata bundle
  ],
});
```

**Why it works:**
`PublicDecryptResults.abiEncodedClearValues` is already `abi.encode`d in the order the handles were passed. Inside the contract, `FHE.checkSignatures(handlesList, cleartexts, decryptionProof)` verifies the KMS signed those cleartexts for those handles. After verification, you `abi.decode(cleartexts, (types...))` in the same order. The handle order, the encoding order, and the decode order must all match - but the SDK handles the encoding for you.

---

### Trap 9 - `FHE.checkSignatures` MUST be the first operation in settlement

**Detection:**

- Writing a `settleMatch` function and tempted to do "cheap" checks first (e.g., `if (m.settled) revert; if (!m.released) revert;`)

**Action:**

Order the function body as:

```solidity
function settleMatch(uint256 matchId, bytes cleartexts, bytes decryptionProof) external {
    MatchResult storage m = matches[matchId];

    // Pre-flight: just enough to know we have a real match
    if (m.createdAt == 0) revert InvalidMatchId();
    if (!m.released) revert MatchNotReleased();
    if (m.settled) revert MatchAlreadySettled();

    // ═════════════════════════════════════════════════════════════
    //  SECURITY-CRITICAL: FHE.checkSignatures FIRST
    // ═════════════════════════════════════════════════════════════
    bytes32[] memory handlesList = new bytes32[](4);
    handlesList[0] = FHE.toBytes32(m.encFillPrice);
    handlesList[1] = FHE.toBytes32(m.encFillSize);
    handlesList[2] = FHE.toBytes32(m.encHasMatch);
    handlesList[3] = FHE.toBytes32(m.encWinnerIdx);

    FHE.checkSignatures(handlesList, cleartexts, decryptionProof);
    // From this line onward, cleartexts are AUTHENTIC.

    // Now decode and run business logic
    (uint64 fillPrice, uint64 fillSize, bool hasMatch, uint64 winnerIdx) =
        abi.decode(cleartexts, (uint64, uint64, bool, uint64));

    // ...
}
```

**Why it works:**
`checkSignatures` verifies that the KMS signed _these specific cleartexts_ for _these specific handles_. If you decode and act on cleartexts before this check, an attacker can pass arbitrary fake cleartexts. The pre-flight checks (`createdAt`, `released`, `settled`) are about state validity - they don't authenticate the cleartexts. The signature check is the actual authentication.

---

### Trap 10 - Use the SDK's `web` entry, not `bundle`

**Detection:**

- TypeScript can't find expected exports
- `SepoliaConfig`, `createInstance`, `initSDK` resolve to wrong types
- Bundler complains about mixed Node/browser deps

**Action:**

Always import from `@zama-fhe/relayer-sdk/web`:

```typescript
import {
  initSDK,
  createInstance,
  SepoliaConfig,
} from "@zama-fhe/relayer-sdk/web";
```

NOT from `@zama-fhe/relayer-sdk` (root) or `@zama-fhe/relayer-sdk/bundle`.

**Why it works:**
The package has multiple entry points for different runtimes. The `web` entry is browser-targeted with proper WASM + worker loading. Other entries assume Node or bundled environments and break in Vite.

---

### Trap 11 - Plugin order: WagmiPlugin BEFORE VueQueryPlugin

**Detection:**

- `useReadContract` and other Wagmi composables throw `Cannot read 'connectors' of undefined`
- Wagmi composables don't work despite WagmiPlugin being registered

**Action:**

In `main.ts`:

```typescript
import { createApp } from "vue";
import { VueQueryPlugin } from "@tanstack/vue-query";
import { WagmiPlugin } from "@wagmi/vue";
import { config } from "@/lib/wagmi";
import { router } from "@/lib/router";
import App from "@/App.vue";

createApp(App)
  .use(WagmiPlugin, { config }) // FIRST
  .use(VueQueryPlugin, {}) // AFTER WagmiPlugin
  .use(router)
  .mount("#app");
```

**Why it works:**
WagmiPlugin uses Vue Query internally for all reactive read state. If VueQueryPlugin loads first with no Wagmi context, Wagmi's internal queries get registered against the wrong query client.

---

### Trap 12 - Vue 3.5 + Vite 8 chokes on self-closing tags

**Detection:**

- Build error: `RolldownError: Invalid end tag` pointing at a line 20-50 lines after the actual broken tag
- The cited line looks correct
- File contains `<span />`, `<div />`, `<a />` etc.

**Action:**

Always use explicit closing tags:

```vue
<!-- WRONG -->
<span class="dot" />
<div class="spacer" />

<!-- RIGHT -->
<span class="dot"></span>
<div class="spacer"></div>
```

**Why it works:**
Vue 3 SFC + Vite's rolldown parser doesn't treat HTML elements as void/self-closing the way XHTML does. Self-closing only works for true void elements (`<input>`, `<br>`, `<hr>`, `<img>`). For everything else, the parser keeps consuming content until it finds a matching close tag - so the error fires far from the actual problem.

---

### Trap 13 - Tailwind v4: Google Fonts `@import` must be FIRST

**Detection:**

- Custom fonts don't load
- Console: `@import` ignored
- `style.css` has Google Fonts import after `@import "tailwindcss"`

**Action:**

```css
/* style.css - LINE 1 */
@import url("https://fonts.googleapis.com/css2?family=...");

/* THEN tailwind */
@import "tailwindcss";

@theme {
  /* tokens */
}
```

**Why it works:**
CSS spec mandates that all `@import` statements appear before any other rules. Tailwind v4's `@import "tailwindcss"` injects `@layer` and other directives that count as "rules." Browsers ignore `@import` statements that appear after rules.

---

### Trap 14 - `v-for` binding scope breaks in vue-tsc

**Detection:**

- TypeScript error: `Property 'item' does not exist on type ...` inside a v-for block
- Dev mode works fine, only build fails
- The v-for is on a wrapper element with v-else or other sibling attributes
- Format-on-save moved the `v-for` attribute to a different element

**Action:**

Put `v-for` directly on the looped element, never on a wrapper:

```vue
<!-- WRONG: v-for on wrapper, breaks vue-tsc binding -->
<div v-else v-for="item in items" :key="item.id">
  <a :href="item.url">{{ item.label }}</a>
</div>

<!-- RIGHT -->
<div v-else>
  <a v-for="item in items" :key="item.id" :href="item.url">{{ item.label }}</a>
</div>
```

**Why it works:**
Vue 3.5 + vue-tsc 3.0 narrowing has trouble inferring the v-for variable when the looped element has multiple v-\* directives or sibling structural attributes. Putting v-for on the actual repeating element keeps the type binding scope clean.

---

### Trap 15 - `tsconfig.app.json` needs explicit `paths` for `@/*`

**Detection:**

- Build fails on every `import ... from "@/..."` with `Cannot find module`
- Dev mode works fine (Vite resolves the alias)
- Only `vue-tsc -b` complains during `pnpm build`

**Action:**

In `tsconfig.app.json`, add a `paths` block to `compilerOptions`:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

(Don't add `baseUrl` - it's deprecated in TS 7.0. Relative paths inside `paths` work without it.)

**Why it works:**
Vite's `resolve.alias` config is for runtime resolution. It does NOT teach TypeScript about path aliases. `vue-tsc` (the strict build checker) needs the alias declared in `tsconfig` independently. Without this, dev runs fine but `pnpm build` fails.

---

### Trap 16 - Vercel Production Overrides are sticky from CLI deploys

**Detection:**

- First deployed via `vercel` CLI, then linked GitHub repo
- New commits trigger builds but use OLD config (e.g., wrong build command)
- Project Settings show all overrides off, but Production still uses them

**Action:**

Push any commit to trigger a fresh deploy that ignores Production Overrides. The Project Settings become canonical only after a non-CLI deploy lands.

For best practice, never deploy with `vercel` CLI as the first deploy. Set up the GitHub integration first, push a commit, let auto-deploy create the project. CLI deploys for ad-hoc previews are fine; just not for the canonical first deploy.

**Why it works:**
The "Production Overrides" section of Vercel's Project Settings shows what was used in the most recent production deploy. Vercel preserves the CLI-passed flags as overrides "just in case." A fresh git-driven deploy uses Project Settings instead.

---

### Trap 17 - Vercel + monorepo needs root `package.json`

**Detection:**

- Vercel Root Directory set to `packages/app`
- Build fails with `vite: command not found` (exit code 127)
- Trying `cd ../../ && pnpm install` fails with `ERR_PNPM_NO_PKG_MANIFEST`

**Action:**

Always have a root `package.json` + `pnpm-workspace.yaml` even if the monorepo is just one package:

```json
// package.json (root)
{
  "name": "your-monorepo",
  "private": true,
  "scripts": {
    "dev": "pnpm --filter app dev",
    "build": "pnpm --filter app build"
  },
  "packageManager": "pnpm@9.0.0"
}
```

```yaml
# pnpm-workspace.yaml
packages:
  - "packages/*"
```

Then in Vercel, leave Root Directory at `packages/app`, and let the default Install Command run. Pnpm will detect the workspace and install correctly.

**Why it works:**
Pnpm needs `package.json` + `pnpm-workspace.yaml` at the root to recognize a monorepo. Without it, Vercel can't traverse up from the package root to install hoisted deps. With it, the standard install command works.

---

### Trap 18 - `useConnectors().length === 0` is statically impossible

**Detection:**

- TypeScript error: `This comparison appears to be unintentional because the types '2' and '0' have no overlap`
- The comparison is inside a `v-if="connectors.length === 0"` empty-state check

**Action:**

Remove the empty-state branch. With Wagmi configured for at least one connector, `connectors.length` is statically narrowed to a positive number. The empty state can never render. Delete the dead branch:

```vue
<!-- DELETE THIS BLOCK -->
<div v-if="connectors.length === 0">No wallets detected</div>
```

**Why it works:**
Wagmi-Vue types `connectors` as a tuple of however many connectors you configured. TS narrows `.length` to that exact number. Comparisons against impossible values fail strict type checking. The block was dead code anyway - connectors are statically known.

---

### Trap 19 - `ORDER_MAX_LIFETIME = 300` (IOC for taker orders)

**Detection:**

- Building a UI showing taker order status
- Need to determine "expired" without contract help

**Action:**

The contract uses `ORDER_MAX_LIFETIME = 300` (seconds = 5 minutes). Compute expiration on the client:

```typescript
const submittedAt = BigInt(order.submittedAt);
const ORDER_LIFETIME_SEC = 300n;
const nowSec = BigInt(Math.floor(Date.now() / 1000));
const isExpired = !order.filled && submittedAt + ORDER_LIFETIME_SEC < nowSec;
```

**Why it works:**
IOC = Immediate-or-Cancel. Stale orders are pruned on the client side because the contract doesn't have a sweeper. The `filled` flag is the canonical "matched" state; expiry is a UX layer.

---

### Trap 20 - Swap UX requires explicit max-price; midpoint pricing is sensitive to limit value

**Detection:**

- Building a swap-style interface on top of an RFQ matching engine
- Tempted to set `limitPrice = SOME_LARGE_NUMBER` as a "market buy at any price"
- Settled fillPrice comes back at half of (ask + sentinel) - way above market

**Action:**

Always require the user to specify a max price. Default it to a reasonable value (e.g., 5% above current ask). Never use a sentinel:

```vue
<input
  v-model.number="maxPrice"
  type="number"
  min="1"
  placeholder="Max cUSDC per cETH"
/>
<p>Won't fill above this. Final price = midpoint of ask + your max.</p>
```

**Why it works:**
Midpoint pricing = `(ask + limit) / 2`. This is fair when both sides are real prices. But if `limit` is a sentinel (`100000`), the midpoint shoots to the moon and the user effectively pays 50× market. The right swap UX is to ask for a max-acceptable explicitly, framing it as slippage tolerance.

---

### Trap 21 - The relayer-sdk's `KMS responds in 5-30s` reality

**Detection:**

- Calling `publicDecrypt()` and the promise hangs longer than expected
- No error, just waiting
- Network tab shows pending request to KMS endpoint

**Action:**

Set realistic loading state expectations in your UI:

```vue
<button>
  <span v-if="stage === 'decrypting'">Decrypting (KMS, ~10s)…</span>
</button>
```

Don't add timeouts shorter than 60s on the decrypt promise. Don't show the user spinners with "<5s" expectations.

**Why it works:**
The KMS is a threshold signing committee. A `publicDecrypt` call requires multiple independent signers to coordinate, sign, and respond. On a healthy network this typically takes 5-15s. Under load it can take 30s+. Optimistic UI (assuming sub-second responses) creates a worse user experience than honest "waiting for KMS" framing.

---

### Trap 22 - `publicDecrypt` requires `FHE.makePubliclyDecryptable` on-chain first

**Detection:**

- Calling `publicDecrypt(handles)` returns an error like "not_ready_for_decryption" or "not_allowed_on_host_acl"
- The handles are valid bytes32 from a contract read

**Action:**

The contract must have called `FHE.makePubliclyDecryptable(handle)` for each handle before the browser can publicly decrypt them. Typically this happens in a "release" function:

```solidity
function releaseMatchForSettlement(uint256 matchId) external {
    MatchResult storage m = matches[matchId];
    if (m.released) revert AlreadyReleased();

    FHE.makePubliclyDecryptable(m.encFillPrice);
    FHE.makePubliclyDecryptable(m.encFillSize);
    FHE.makePubliclyDecryptable(m.encHasMatch);
    FHE.makePubliclyDecryptable(m.encWinnerIdx);

    m.released = true;
    emit MatchReleasedForSettlement(matchId);
}
```

In the frontend, the workflow is two distinct transactions:

1. Call `releaseMatchForSettlement` first
2. THEN call `publicDecrypt` from the SDK
3. THEN call `settleMatch` with the decrypted values

**Why it works:**
The default ACL on encrypted handles allows decryption only by parties the contract explicitly grants access to. `makePubliclyDecryptable` adds a public ACL entry. Without it, the KMS will refuse to sign cleartexts because the requesting party (the browser) isn't authorized.

---

### Trap 23 - `useReadContract` doesn't auto-refetch after `writeContract`

**Detection:**

- User submits a tx, it confirms, but the UI doesn't reflect new state
- Manually refreshing the page shows correct state

**Action:**

After any state-changing `writeContract`, manually call `refetch()` on dependent reads. Use a `watch` on `isTxConfirmed`:

```typescript
const { data: state, refetch } = useReadContract({ ... });
const { isSuccess: isTxConfirmed } = useWaitForTransactionReceipt({ hash });

watch(isTxConfirmed, (confirmed) => {
  if (confirmed) {
    void refetch();  // pull fresh state
  }
});
```

**Why it works:**
`useReadContract` caches reads aggressively (TanStack Query). Cache invalidation isn't automatic across tx boundaries. Explicitly refetching after confirmed txs is the canonical pattern.

---

## Decision rules

When in doubt, follow these:

| Condition                                               | Decision                                                                                                  |
| ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Need to encrypt user input client-side                  | `instance.createEncryptedInput(contractAddr, userAddr).addN(value).encrypt()`                             |
| Passing handles to a contract function                  | Wrap in `toHex()` from viem                                                                               |
| Reading event logs                                      | Import `getLogs` from `viem/actions`, call as `getLogs(client.value, params)`                             |
| Reading contract state via Wagmi composable             | `useReadContract` with `query: { enabled: ... }` for conditional fetching                                 |
| Reading contract state imperatively                     | `readContract` from `viem/actions`                                                                        |
| Waiting for tx receipt                                  | `useWaitForTransactionReceipt` (reactive) OR `waitForTransactionReceipt` from `viem/actions` (imperative) |
| Triggering decryption                                   | Two-tx flow: contract `release` first, then SDK `publicDecrypt`, then contract `settle`                   |
| User has multiple wallet extensions                     | Detect on boot, warn the user, don't try to fix programmatically                                          |
| TS path alias breaks build                              | Add `paths` block to `tsconfig.app.json`                                                                  |
| Vite + relayer-sdk warnings about threading             | COOP/COEP headers + `optimizeDeps.exclude`                                                                |
| Vercel deploy uses wrong build command after CLI deploy | Push any commit to trigger a fresh git-driven deploy                                                      |
| Component types narrow incorrectly in v-for             | Move `v-for` to the looped element directly, not the wrapper                                              |
| Build error "Invalid end tag"                           | Search for self-closing tags on non-void elements                                                         |
| Need a swap UX over an RFQ engine                       | Require explicit max-price input; never use a sentinel                                                    |
| Decryption hangs more than 30s                          | Probably KMS load, not a bug. Surface honest loading state                                                |
| Need to read all events globally                        | Parallel `getLogs` calls, one per event signature, then merge + sort by block desc                        |

---

## Deployment playbook (Vercel)

Step-by-step for getting a working production deploy:

1. **Verify the build works locally first**

```bash
   pnpm build
   # confirm dist/ populated, no TS errors
```

2. **Create root `package.json` + `pnpm-workspace.yaml`** (Trap 17)

3. **Create `packages/app/vercel.json`** (Trap 1)

```json
{
  "buildCommand": "pnpm build",
  "outputDirectory": "dist",
  "framework": "vite",
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "Cross-Origin-Opener-Policy", "value": "same-origin" },
        { "key": "Cross-Origin-Embedder-Policy", "value": "require-corp" }
      ]
    }
  ],
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

4. **Connect Vercel to the GitHub repo first**, NOT via CLI (Trap 16)
   - Vercel UI → Add New Project → Import Git Repository
   - Set Root Directory: `packages/app`
   - Framework Preset: Vite
   - Don't override Build Command, Install Command, or Output Directory

5. **Add environment variables in Vercel UI** before first deploy
   - `VITE_INFURA_API_KEY` (Production, Preview, Development - all 3)

6. **Push a commit to trigger first deploy.** Watch the build log. If it fails, never use Build Command overrides as a workaround - fix the underlying TS/lint issues. Overrides become tech debt.

7. **Verify SPA routing works** by visiting `/swap`, `/match/0` directly (not via in-app navigation). If you get 404s, the rewrites in `vercel.json` aren't being picked up - push another commit.

---

## Expected outcomes

If you've applied this skill correctly, you should be able to:

✅ Encrypt a user's input in the browser, get back valid handles + ZK proof in 3-8 seconds

✅ Submit those handles to a contract and have the tx confirm on Sepolia in 15-30 seconds

✅ Read encrypted state from the contract as `0x...` bytes32 strings

✅ Trigger `FHE.makePubliclyDecryptable` from the contract, then call `publicDecrypt` from the SDK, get back `abiEncodedClearValues + decryptionProof` in 5-30 seconds

✅ Pass those values directly to `settleMatch` (or your equivalent) and have the contract verify them via `FHE.checkSignatures` and act on the cleartexts

✅ Read all historical events with parallel `getLogs` calls and render an activity feed

✅ Deploy to Vercel with cross-origin isolation working in production

✅ Run the entire flow end-to-end from a browser, on Sepolia, with no off-chain operator

If any of those don't work: re-read the relevant trap above. The fix is documented.

---

## Anti-patterns (DON'T do these)

❌ Don't trust React-first Wagmi tutorials when working in Vue (Trap 4)

❌ Don't use `usePublicClient` in Vue (it doesn't exist)

❌ Don't call `publicClient.value.getLogs(...)` (use `getLogs(client.value, ...)` from `viem/actions`)

❌ Don't pass `Uint8Array` directly to `writeContract` for `bytes32`/`bytes` args (use `toHex`)

❌ Don't use the bundle entry of relayer-sdk (use `/web`)

❌ Don't deploy via `vercel` CLI as the first deploy (set up GitHub integration first)

❌ Don't override Vercel's build command as a workaround - fix the TS errors

❌ Don't use sentinel values as limit prices in midpoint-priced systems

❌ Don't decode cleartexts before `FHE.checkSignatures` in settlement functions

❌ Don't add timeouts < 60s on `publicDecrypt` calls

❌ Don't put `v-for` on wrappers that also have v-else or other directives

❌ Don't use self-closing tags for non-void elements in Vue 3 SFCs

❌ Don't trust `connectors.length === 0` checks - Wagmi-Vue narrows the type

❌ Don't rely on auto-refetch after `writeContract` confirms - manually refetch

---

## Reference: minimal working example

The smallest possible working setup demonstrating encryption + decryption.

`useFhevm.ts`:

```typescript
import { ref } from "vue";
import {
  initSDK,
  createInstance,
  SepoliaConfig,
  type FhevmInstance,
} from "@zama-fhe/relayer-sdk/web";

let instance: FhevmInstance | null = null;
let bootPromise: Promise<FhevmInstance> | null = null;

const status = ref<"idle" | "booting" | "ready" | "error">("idle");
const error = ref<string | null>(null);

export function useFhevm() {
  async function ensureReady(): Promise<FhevmInstance> {
    if (instance) return instance;
    if (bootPromise) return bootPromise;

    status.value = "booting";
    bootPromise = (async () => {
      try {
        await initSDK();
        instance = await createInstance({
          ...SepoliaConfig,
          network: (window as any).ethereum,
        });
        status.value = "ready";
        return instance;
      } catch (err: any) {
        status.value = "error";
        error.value = err?.message ?? String(err);
        bootPromise = null;
        throw err;
      }
    })();
    return bootPromise;
  }

  return { ensureReady, status, error };
}
```

Submission:

```typescript
const instance = await ensureReady();
const enc = await instance
  .createEncryptedInput(contractAddr, userAddr)
  .add64(value1)
  .add64(value2)
  .encrypt();

await writeContractAsync({
  address: contractAddr,
  abi,
  functionName: "submit",
  args: [toHex(enc.handles[0]), toHex(enc.handles[1]), toHex(enc.inputProof)],
});
```

Decryption + settlement:

```typescript
const result = await instance.publicDecrypt([handle1, handle2]);
await writeContractAsync({
  address: contractAddr,
  abi,
  functionName: "settle",
  args: [matchId, result.abiEncodedClearValues, result.decryptionProof],
});
```

---

## Credits

Compiled by [@unifyWeb3](https://twitter.com/unifyWeb3) from the 21-day Obscura sprint. Source: [github.com/unifyWeb3/obscura-monorepo](https://github.com/unifyWeb3/obscura-monorepo).

Built on [Zama FHEVM](https://docs.zama.ai/protocol). Submitted to the Zama Developer Program - Season 2 Builder Track.

This skill is a living document. PRs welcome via the source repo.

---
> Source: [unifyWeb3/obscura](https://github.com/unifyWeb3/obscura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
