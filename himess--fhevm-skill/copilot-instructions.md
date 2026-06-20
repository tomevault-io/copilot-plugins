## fhevm-skill

> >-


# FHEVM Development Skill

> Build confidential smart contracts with Fully Homomorphic Encryption on EVM chains using the Zama Protocol.

## Installation

Copy the `fhevm-skill/` directory to your AI tool's skills folder:
- **Claude Code**: `~/.claude/skills/fhevm/`
- **Cursor**: `.cursor/skills/fhevm/`
- **Windsurf**: `.windsurf/skills/fhevm/`
- **Copilot**: `.github/skills/fhevm/`

## Architecture

FHEVM uses a **coprocessor model** with 4 components:

```
User (browser)                          Coprocessor (FHE engine)
  │ encrypt via Relayer SDK                  │ executes actual FHE math
  ▼                                          ▲
Contract (Host Chain, EVM)  ──handles──►     │
  │ stores handles (bytes32)                 │
  │ symbolic execution only                  │
  ▼                                          │
ACL Contract ◄──── who can read what ──────► │
  │                                          │
  ▼                                          │
Gateway Chain ──── orchestrates ──────► KMS (Key Management Service)
                   decryption            │ threshold decryption
                                         │ multi-party computation
                                         │ no single party has the full key
```

**How it works:**

1. **Host Chain** (Ethereum/Sepolia): Your contracts live here. FHE operations are **symbolic** — when you call `FHE.add(a, b)`, the chain produces a new handle but the real encrypted computation happens asynchronously in the coprocessor.

2. **Coprocessor**: Rust-based engine that executes the actual FHE math offchain. Receives operations from the host chain, processes ciphertexts, returns result handles.

3. **ACL Contract**: On-chain registry tracking who can access each encrypted handle. Every `FHE.allow()`, `FHE.allowThis()`, and `FHE.allowTransient()` writes to this contract.

4. **KMS (Key Management Service)**: Handles decryption via **threshold multi-party computation** — the FHE secret key is split across multiple KMS nodes. No single node can decrypt alone. When `FHE.makePubliclyDecryptable()` is called, the KMS nodes cooperatively produce a decryption proof that can be verified on-chain via `FHE.checkSignatures()`.

5. **Gateway Chain** (chainId 10901 for Sepolia): Orchestrates communication between the host chain and KMS for decryption requests.

6. **Relayer SDK** (`@zama-fhe/relayer-sdk`): Frontend library that handles client-side encryption (ZK proof generation) and coordinates decryption requests with the KMS via the Gateway.

**Key implications:**
- You NEVER see plaintext in contract logic (except at system boundaries like wrap/unwrap)
- All branching on encrypted values uses `FHE.select()`, not `if/else`
- Decryption is **asynchronous** — mark a value as decryptable, then verify the KMS proof separately
- The FHE key is **never held by a single entity** — threshold security via KMS

## Picking the Right SDK Layer

Solidity has one current library (`@fhevm/solidity`); the off-chain stack has two layers, both first-class but for different jobs.

### Solidity (on-chain)

| | OLD (deprecated) | CURRENT (use this) |
|---|---|---|
| **Package** | `fhevm` v0.5-0.6 | `@fhevm/solidity` v0.11+ |
| **Library** | `TFHE` | `FHE` |
| **Import** | `import "fhevm/lib/TFHE.sol"` | `import {FHE, euint64} from "@fhevm/solidity/lib/FHE.sol"` |
| **Input type** | `einput` | `externalEuint64` (typed per width) |
| **Input parse** | `TFHE.asEuint64(einput, proof)` | `FHE.fromExternal(externalEuint64, proof)` |
| **Config** | `SepoliaZamaFHEVMConfig` | `ZamaEthereumConfig` |
| **Decryption** | `Gateway.requestDecryption()` | `FHE.makePubliclyDecryptable()` + `FHE.checkSignatures()` |

> **Warning**: `fhevm-contracts` was **archived in 2025** and used the OLD `TFHE` library. It has been replaced by `@openzeppelin/confidential-contracts` which uses the new `FHE` library. Use the new package for all development.

### Off-chain (tests + frontends)

Two layers cooperate — pick by the job, not by "which is newer". The high-level layer is built on top of the foundational layer; they aren't substitutes.

| Layer | Package | Use it for | Don't use for |
|---|---|---|---|
| **Foundational SDK** | `@zama-fhe/relayer-sdk@0.4.1` (EXACT pin) | Every Hardhat test (the plugin imports it internally), server-side scripts, frontends for non-token contracts (voting / auction / AMM / vault UIs), manual encryption pipelines | Token UIs where the high-level layer is more ergonomic |
| **High-level Token API** | `@zama-fhe/sdk@3.x` + `@zama-fhe/react-sdk@3.x` | ERC-7984 token UIs in the browser (balances, transfers, wraps, operator setup), React apps wanting `useConfidentialBalance` / `useConfidentialTransfer` hooks | Hardhat tests, non-token contracts, server-side encryption |
| **Deprecated** | `fhevmjs` | Never. Migration table below. | — |

**Rule of thumb:** The foundational SDK is mandatory at the test layer (the hardhat-plugin pins it). Add the high-level Token API on top whenever you build a token UI. They're independent and can coexist in one app.

See **[references/sdk-v3-guide.md](references/sdk-v3-guide.md)** and **[references/react-sdk-guide.md](references/react-sdk-guide.md)** for the Token API. See **[references/frontend-integration.md](references/frontend-integration.md)** for the foundational SDK (which every test relies on).

### Migrating from `fhevmjs` (deprecated)

If you have existing code using `fhevmjs`, migrate via this map. **Do NOT mix `fhevmjs` and the new SDK in the same project** — they target different protocol versions and will produce incompatible handles.

| `fhevmjs` (deprecated) | CURRENT (use one of these instead) |
|---|---|
| `import { createInstance } from "fhevmjs"` | **Hardhat tests:** `import { createInstance, SepoliaConfig } from "@zama-fhe/relayer-sdk/web"` (foundational SDK) |
| `import { initFhevm } from "fhevmjs"` | **Browser/React token UI:** `import { ZamaSDK, RelayerWeb } from "@zama-fhe/sdk"` (high-level Token API) |
| `await createInstance({ chainId, publicKey })` | `await createInstance({ ...SepoliaConfig, network: provider })` — addresses come from the config |
| `instance.encrypt8/16/32/64(value)` | `instance.createEncryptedInput(addr, user).add8/16/32/64(value).encrypt()` |
| `instance.generatePublicKey({ verifyingContract })` + manual reencrypt | Foundational: `instance.userDecrypt(handles, ...)`. Token UIs: `useConfidentialBalance` hook |
| `instance.decrypt(contractAddr, ciphertext)` | `instance.publicDecrypt([handle])` (only after `FHE.makePubliclyDecryptable` on-chain) |

**Migration checklist:**
1. `npm uninstall fhevmjs && npm install --save-exact @zama-fhe/relayer-sdk@0.4.1` (foundational; mandatory for tests). For new browser token UIs, additionally `npm install @zama-fhe/sdk@3.x @zama-fhe/react-sdk@3.x`.
2. Replace every `import { ... } from "fhevmjs"` with the right new import (see table above).
3. Update Solidity contracts: replace `einput` with `externalEuintXX`, `TFHE.*` with `FHE.*`. See the Solidity migration table at the top of this section.
4. Replace `Gateway.requestDecryption(...)` with `FHE.makePubliclyDecryptable(...)` + `FHE.checkSignatures(...)`.
5. Run `scripts/validate-fhevm.sh` against `contracts/` to catch any leftover legacy patterns.
6. Run `npx hardhat test` — every test must still pass.

### Self-Correction Table

If you just generated code containing any of these, STOP and fix:

| If you wrote... | You meant... |
|---|---|
| `TFHE.asEuint64(input, proof)` | `FHE.fromExternal(encInput, proof)` — where `encInput` has type `externalEuint64`. (`externalEuint64` is the **parameter type**, not a value: `function deposit(externalEuint64 encInput, bytes calldata proof) { euint64 amount = FHE.fromExternal(encInput, proof); ... }`) |
| `einput` parameter type | `externalEuint64` (or `externalEuintN` / `externalEbool` / `externalEaddress`) |
| `Gateway.requestDecryption()` | `FHE.makePubliclyDecryptable()` + `checkSignatures` |
| `import "fhevm/lib/TFHE.sol"` | `import {FHE} from "@fhevm/solidity/lib/FHE.sol"` |
| `SepoliaZamaFHEVMConfig` | `ZamaEthereumConfig` |
| `fhevmjs` package | `@zama-fhe/sdk` (browser/React) or `@zama-fhe/relayer-sdk` (Hardhat tests) |
| `import { createInstance } from "fhevmjs"` | `import { ZamaSDK, RelayerWeb } from "@zama-fhe/sdk"` |
| `instance.createEncryptedInput(...).add64(n).encrypt()` in app code | `await token.confidentialTransfer(to, amount)` via `sdk.createToken(addr)` |
| Hand-rolled EIP-712 for user decryption in React | `useConfidentialBalance({ tokenAddress })` from `@zama-fhe/react-sdk` |
| `<ZamaProvider>` outside `<QueryClientProvider>` | Provider order: `WagmiProvider` → `QueryClientProvider` → `ZamaProvider` |
| `FHE.decrypt(value)` in Solidity | No in-contract decrypt. Use Relayer SDK off-chain |
| `ebytes64` or `eint8` | These types don't exist. Use `euint64` or `ebool` |
| `FHE.div(a, encryptedB)` | Divisor must be plaintext: `FHE.div(a, uint64(b))` |
| `FHE.safeAdd()` / `safeSub()` | Don't exist. All arithmetic wraps silently |
| `npm install hardhat` (gets v3) | Use `npm install hardhat@^2.22.0` — FHEVM plugin requires Hardhat 2 |
| `npm install hardhat-deploy` + `import "hardhat-deploy"` in config | Crashes with `TypeError: Cannot read 'JsonRpcSigner' of undefined` (zksync-web3 × ethers v6). Drop the import unless you actually need deploy scripts. Templates ship with it commented out. |
| `npm install @nomicfoundation/hardhat-ethers` (gets v4) | Use `@nomicfoundation/hardhat-ethers@^3.1.3` — v4 requires Hardhat 3 |
| `npm install @nomicfoundation/hardhat-verify` (gets v3) | Use `@nomicfoundation/hardhat-verify@^2.0.0` — v3 silently requires Hardhat 3 and the smoke compile fails with `Cannot find module '.../hardhat/config'`. Same Hardhat 2/3 split as `hardhat-ethers`. |
| `npm install @typechain/hardhat` (alone) | Crashes with `Couldn't find ethers-v6`. Always pair with `@typechain/ethers-v6@^0.5.1` AND bare `typechain@^8.3.2` (peer dep, not auto-installed) — otherwise compile fails with `HH801: Plugin @typechain/hardhat requires the following dependencies to be installed: typechain`. |
| `@zama-fhe/relayer-sdk@^0.4.1` (caret) | Use `@zama-fhe/relayer-sdk@0.4.1` (exact). Caret resolves to 0.4.3 → plugin 0.4.2 hard-fails with "Invalid relayer-sdk version. Expecting 0.4.1." |
| `npm install @zama-fhe/relayer-sdk@0.4.1` without `--save-exact` | Silently writes `"^0.4.1"` to `package.json` despite the explicit version. Always pin with `npm install --save-exact @zama-fhe/relayer-sdk@0.4.1` so the caret never sneaks back in. |
| `abi.decode(cleartexts, (uint64))` *(in the on-chain `revealResults`-style callback after `publicDecrypt + checkSignatures`)* | SDK encodes EVERY cleartext as `uint256`: use `abi.decode(cleartexts, (uint256))` then cast down. **Note:** this applies to public-decrypt cleartexts decoded on-chain. The off-chain `userDecrypt` flow returns typed values directly through the SDK — no `abi.decode` needed there. |
| `FHE.randEuint64(100)` | upperBound must be power of 2: `FHE.randEuint64(128)` then `FHE.rem(r, 100)` |
| `expect(call).to.be.revertedWithCustomError(c, "MyError")` for a tx whose function signature includes `externalEuint*` | Plugin wraps the revert as `HardhatFhevmError: Fhevm assertion failed.` and chai never sees the custom-error selector. The wrapping is keyed on the **function signature** (any function whose ABI takes `externalEuint*` gets every revert wrapped), NOT on whether `FHE.fromExternal` was reached — modifier-guarded reverts (`whenNotPaused`, `onlyOwner`) get wrapped too as long as the signature has `externalEuint*`. Always use `try { await call; } catch (e) { expect(e.message).to.match(/MyError\|Fhevm/); }` on any such function. Functions whose signature has no `externalEuint*` (pure plaintext / view) are unaffected — chai matcher works there. See `references/testing-guide.md`. |
| `BigInt(handleStr)` to check "is this slot uninitialised?" written as `handleStr === 0n` | `confidentialBalanceOf` returns the handle as a `bytes32` **string** in ethers v6, not a BigInt. Comparing a string to a BigInt always returns false. Use `BigInt(handle) === 0n` (cast first). |
| Sequential FHE-state-mutating txs on Sepolia with no pacing | Coprocessor's view of newly-created handles can lag the host chain by a block or two. Two back-to-back mints/transfers from the same wallet may revert the second with `ERC7984ZeroBalance` even though the first succeeded. Add `await new Promise(r => setTimeout(r, 6000))` between FHE-state-mutating txs in your Sepolia harness, or use a retry loop. Mock-mode is unaffected. |
| Submitting an FHE-state-mutating tx on Sepolia with `gasLimit` left to ethers' `estimateGas` | Default estimation under-estimates by 100–1000 gas on calls that perform ≥3 sequential FHE ops (one stress-test agent OOG'd at `gasUsed=489117 / gasLimit=489947` — 830 gas of headroom). Always set explicit `gasLimit: 2_000_000n` on FHE-state-mutating Sepolia txs (well within the 20 M HCU per-tx budget). `templates/onchain-e2e.ts` exposes a `sepoliaTxOpts()` helper that packages this. |
| Submitting a Sepolia tx with default `maxPriorityFeePerGas` (1 gwei) | The public mempool silently drops 1-gwei-tip txs on busy days — `eth_getTransactionByHash` returns `null` while the wallet's nonce is unchanged, leaving you stuck. Bump the tip to ≥3 gwei: `const fd = await provider.getFeeData(); maxPriorityFeePerGas = fd.maxPriorityFeePerGas + ethers.parseUnits("3", "gwei")`. Same `templates/onchain-e2e.ts:sepoliaTxOpts()` packages this. |

## Agent Workflow

**CRITICAL REMINDERS (read before writing ANY code):**
- ERC-7984 tokens use `confidentialTransfer`/`confidentialBalanceOf` — NOT `transfer`/`balanceOf`
- Use Hardhat 2 (`^2.28.4`) — NOT Hardhat 3
- Call `FHE.allowThis()` + `FHE.allow()` after EVERY FHE operation that stores a value
- Functions receiving encrypted handles from other contracts need `FHE.isSenderAllowed()` check
- Encrypted operations NEVER revert — they silently return 0

**When a user asks to create a new FHEVM project:**

1. **Scaffold**: Clone `fhevm-hardhat-template` or create Hardhat project with FHEVM deps
2. **Configure**: Set `evmVersion: "cancun"` in hardhat.config.ts (use templates/hardhat.config.ts)
3. **Write contract**: Use templates/ as starting points. Always use the NEW FHE library
4. **Apply ACL pattern**: After every FHE operation that stores a value: `allowThis` + `allow`
5. **Write tests**: Use templates/test-template.ts as boilerplate. Test silent failures
6. **Validate**: Run `scripts/validate-fhevm.sh` against the contracts directory
7. **Deploy**: `npx hardhat run scripts/deploy.ts --network sepolia` then verify (use `templates/deploy-template.ts` — plain ethers, no `hardhat-deploy`)

**When a user asks to add FHE to an existing contract:**

1. Add `@fhevm/solidity` dependency
2. Inherit `ZamaEthereumConfig` — **mandatory**. This abstract contract's
   constructor calls `FHE.setCoprocessor(...)` with the canonical Ethereum
   mainnet + Sepolia coprocessor address. **Without it, every `FHE.*` call
   reverts** with "coprocessor not initialized" (the coprocessor address is
   `address(0)` by default). One contract, one inheritance — `ZamaEthereumConfig`
   covers both Sepolia and Mainnet because they share the same coprocessor
   topology. Do NOT look for a `SepoliaConfig` Solidity contract; only the
   `ZamaEthereumConfig` abstract base exists in `@fhevm/solidity@0.11.x`.
   *(There IS a JS export named `SepoliaConfig` in `@zama-fhe/relayer-sdk/{web,node}` — that's the off-chain SDK config object passed to `createInstance`, a different layer. See `references/frontend-integration.md`.)*
3. Replace plaintext state variables with encrypted types (`uint256 balance` → `euint64 balance`)
4. Replace `if/require` conditions with `FHE.select` patterns
5. Add ACL calls after every state mutation
6. Run `scripts/validate-fhevm.sh`

## Quick Start

> **Node.js compatibility:** Hardhat 2 officially supports **Node 18 / 20 / 22**. Node 24+ prints `WARNING: ... not supported` but generally works. Use `nvm use 20` or `nvm use 22` if you hit unexplained Hardhat errors. Drop `tsconfig.json` from `templates/tsconfig.json` into your project root to avoid `TS5011 rootDir` errors during `npx hardhat test`.

### 1. Project Setup

```bash
git clone https://github.com/zama-ai/fhevm-hardhat-template.git my-fhevm-project
cd my-fhevm-project && npm install
# Template does NOT include OpenZeppelin. Install for Ownable2Step, ReentrancyGuard, etc.:
npm install @openzeppelin/contracts@^5.6.1
# For ERC-7984 tokens, also install:
npm install @openzeppelin/confidential-contracts@^0.4.0
```

Key deps: `@fhevm/solidity` ^0.11.1, `@fhevm/hardhat-plugin` ^0.4.2, `@zama-fhe/relayer-sdk` **EXACT 0.4.1** (no caret), `@typechain/hardhat` + `@typechain/ethers-v6`.

**CRITICAL**: Use **Hardhat 2** (^2.22.0), NOT Hardhat 3. The `@fhevm/hardhat-plugin` is incompatible with Hardhat 3. If setting up from scratch instead of cloning the template:

```bash
# Two-step install. Step 1 uses --save-exact so that @zama-fhe/relayer-sdk
# lands as exactly "0.4.1" in package.json (default `npm install x@0.4.1`
# silently writes "^0.4.1" which can later resolve to 0.4.3 → plugin breaks).
npm install --save-dev --save-exact --legacy-peer-deps \
  @zama-fhe/relayer-sdk@0.4.1

# Step 2: everything else (carets are fine here).
npm install --save-dev --legacy-peer-deps \
  hardhat@^2.28.4 \
  @fhevm/solidity@^0.11.1 @fhevm/hardhat-plugin@^0.4.2 @fhevm/mock-utils@^0.4.2 \
  encrypted-types@^0.0.4 \
  @nomicfoundation/hardhat-chai-matchers@^2.1.0 @nomicfoundation/hardhat-ethers@^3.1.3 \
  @nomicfoundation/hardhat-verify@^2.0.0 \
  @typechain/hardhat @typechain/ethers-v6@^0.5.1 typechain@^8.3.2 \
  ethers@^6.16.0 \
  @openzeppelin/contracts@^5.6.1 @openzeppelin/confidential-contracts@^0.4.0

# Smoke test BEFORE writing any contracts — catches the most common install issues.
npx hardhat init    # if no project yet, otherwise skip
echo > contracts/Smoke.sol  # any minimal .sol file
npx hardhat compile  # MUST exit 0
```

> **Critical version pins** (other versions silently break the toolchain):
> - `@zama-fhe/relayer-sdk@0.4.1` — **exact, NOT caret**. Use `--save-exact` (step 1 above) or your `package.json` will say `^0.4.1` and a future `npm install` may resolve `0.4.3`, after which the plugin hard-fails: `Invalid @zama-fhe/relayer-sdk version. Expecting 0.4.1. Got 0.4.3 instead`.
> - `@nomicfoundation/hardhat-verify@^2.0.0` — **the 2-series pin matters**. Without it, `npm install` resolves to `3.0.17+` whose `peerDep hardhat: ^3.4.0` silently requires Hardhat 3, breaking the smoke compile with `Error: Cannot find module '.../hardhat/config' ... Did you mean to import "hardhat/config.js"?`. Same Hardhat 2/3 split trap as `hardhat-ethers`.
> - `@typechain/ethers-v6@^0.5.1` AND `typechain@^8.3.2` — both required. `@typechain/hardhat` has `typechain` as a peer dep that npm does not auto-install. Without `@typechain/ethers-v6`: `Couldn't find ethers-v6`. Without bare `typechain`: `HH801: Plugin @typechain/hardhat requires the following dependencies to be installed: typechain`.
> - `encrypted-types@^0.0.4` — transitive peer dep surfaced explicitly so newer plugin versions resolve cleanly. You don't import it directly; safe to keep, no direct usage in your code.
> - `hardhat-deploy` is **NOT installed by default**. It transitively pulls `zksync-web3@0.14.4`, which crashes on ethers v6 at module load with `Cannot read 'JsonRpcSigner' of undefined`. Only install it if you need named-account deployment scripts, and pin a version compatible with your ethers major.
> - **Smoke test:** run `npx hardhat compile` in an empty project right after install. If this fails, fix the install before writing any contracts.

Config must set `evmVersion: "cancun"` and `viaIR: true` (avoids stack-too-deep in complex FHE contracts).

#### ERC-7984 operator setup (replaces ERC-20 `approve`)

If your contract calls `confidentialTransferFrom(user, …)`, the user must FIRST authorize your contract as an operator on the token (one tx, separate from the actual call):

```solidity
// User-side, separate transaction:
token.setOperator(myContract, uint48(block.timestamp + 1 days));  // expiry = "until" timestamp
// Now myContract can call token.confidentialTransferFrom(user, ...) until that timestamp.
```

This is the ERC-7984 equivalent of `approve(spender, amount)` — but **time-based**, not amount-based. To revoke: `setOperator(spender, 0)`.

### Sepolia Deployment

For deploying to Sepolia testnet, you need:

**RPC URLs** (no API key needed for public RPCs):
```
https://ethereum-sepolia-rpc.publicnode.com
https://rpc.ankr.com/eth_sepolia
https://sepolia.infura.io/v3/YOUR_KEY  (if you have Infura)
```

**Sepolia ETH faucets:**
- https://www.alchemy.com/faucets/ethereum-sepolia
- https://cloud.google.com/application/web3/faucet/ethereum/sepolia
- https://faucets.chain.link/sepolia

### 2. Minimal Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {FHE, euint64, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";

contract ConfidentialCounter is ZamaEthereumConfig {
    euint64 private _count;

    function add(externalEuint64 encryptedValue, bytes calldata inputProof) external {
        euint64 value = FHE.fromExternal(encryptedValue, inputProof);
        _count = FHE.add(_count, value);
        FHE.allowThis(_count);
        FHE.allow(_count, msg.sender);
    }

    function getCount() external view returns (euint64) {
        return _count;
    }
}
```

### 3. Test It

```typescript
import { ethers, fhevm } from "hardhat";
import { FhevmType } from "@fhevm/hardhat-plugin";

it("increments encrypted counter", async function () {
    const encrypted = await fhevm
        .createEncryptedInput(contractAddress, signer.address)
        .add64(42)
        .encrypt();
    await contract.add(encrypted.handles[0], encrypted.inputProof);

    const handle = await contract.getCount();
    const clear = await fhevm.userDecryptEuint(FhevmType.euint64, handle, contractAddress, signer);
    expect(clear).to.equal(42n);
});
```

## Decision Trees

### "Which encrypted type should I use?"

```
Is the value a boolean (true/false)?
  └─ Yes → ebool
  └─ No → Is it an Ethereum address?
       └─ Yes → eaddress
       └─ No → What's the value range?
            └─ 0-255 → euint8  (ages, scores, small enums)
            └─ 0-65535 → euint16  (years, small counts)
            └─ 0-4B → euint32  (timestamps, medium counts)
            └─ 0-18.4×10¹⁸ → euint64  (token amounts, balances — MOST COMMON)
            └─ Larger → euint128 or euint256
                 ⚠️ euint256: only eq/ne comparisons, no ordering (gt/lt/ge/le)
```

### "How should I handle decryption?"

```
Who needs to see the plaintext?
  └─ Only the data owner (private) → User Decryption (EIP-712 flow)
       See: references/decryption-guide.md#user-decryption
  └─ Everyone / contract logic needs it → Public Decryption
       See: references/decryption-guide.md#public-decryption
  └─ A backend service on behalf of user → Delegated Decryption
       See: references/decryption-guide.md#delegated-decryption
```

### "Which ACL function do I need?"

```
When is the encrypted value needed?
  └─ In a FUTURE transaction (stored in state) → FHE.allow(value, account)
       Also call: FHE.allowThis(value) so the contract can access it later
  └─ Only within THIS transaction (cross-contract call) → FHE.allowTransient(value, account)
       Cheaper gas (EIP-1153 transient storage), cleared after tx
  └─ Anyone should be able to decrypt it → FHE.makePubliclyDecryptable(value)
```

### "Which library/import should I use?"

```
Using @openzeppelin/confidential-contracts (ERC-7984)? → FHE + ZamaEthereumConfig + ERC7984 base
Writing a standalone contract from scratch? → FHE + ZamaEthereumConfig
```

### "Cross-contract encrypted value?"

```
Same transaction? → FHE.allowTransient(value, target) before the call
Future transaction? → FHE.allow(value, target)
Forwarding user's input proof? → STOP: proofs bound to msg.sender. Use 2-tx flow.
```

### "My FHEVM code doesn't work — what's wrong?"

```
Is it a compilation error?
  └─ "type not found" → Check imports: use @fhevm/solidity/lib/FHE.sol
  └─ "div/rem type mismatch" → Divisor must be plaintext: FHE.div(enc, uint64)
  └─ "gt/lt not found for euint256" → euint256 only supports eq/ne
  └─ "cannot be declared as view" → FHE ops modify state. Remove view/pure modifier
  └─ "Cannot determine a test runner" → You have Hardhat 3 — downgrade to Hardhat 2
  └─ "invalid: ^2.0.0 from @fhevm/hardhat-plugin" → Same: npm install hardhat@^2.22.0
Is it a runtime revert?
  └─ "Sender not allowed" → Missing FHE.allowTransient() before cross-contract call
  └─ Reverts on FHE.rand → Random must be in non-view function
  └─ "evmVersion" errors → Set evmVersion: "cancun" in hardhat.config
Is it a silent failure (no revert, but wrong result)?
  └─ Transfer sends 0 → Insufficient balance (by design). Check with handle comparison
  └─ Contract can't read its own data → Missing FHE.allowThis() after operation
  └─ Decryption returns nothing → Missing FHE.allow(value, user)
```

## Core Patterns

### Pattern 1: The ACL Triple (MANDATORY after every FHE operation that produces a stored value)

```solidity
balances[user] = FHE.add(balances[user], amount);
FHE.allowThis(balances[user]);    // Contract can use it in future tx
FHE.allow(balances[user], user);  // User can decrypt their own balance
```

Forgetting `allowThis` is the #1 FHEVM bug — the contract creates a new handle but loses access to it because each FHE operation produces a NEW handle with NO inherited permissions.

### Pattern 2: Silent Transfer (no-revert on insufficient balance)

```solidity
function _transfer(address from, address to, euint64 amount) internal {
    ebool canTransfer = FHE.le(amount, balances[from]);
    euint64 transferValue = FHE.select(canTransfer, amount, FHE.asEuint64(0));

    balances[from] = FHE.sub(balances[from], transferValue);
    FHE.allowThis(balances[from]);
    FHE.allow(balances[from], from);

    balances[to] = FHE.add(balances[to], transferValue);
    FHE.allowThis(balances[to]);
    FHE.allow(balances[to], to);
}
```

FHE transfers NEVER revert on insufficient balance — reverting would leak balance information. They silently transfer 0. This is by design, not a bug.

> **⚠ Asymmetry — uninitialised handles DO revert.** OpenZeppelin's `ERC7984._transfer` reverts with `ERC7984ZeroBalance(from)` if the sender's balance handle has *never been written* (slot is `bytes32(0)`). The "silent transfer 0" rule applies only **after** the slot has been initialised at least once. To exercise the silent-fail path in tests, mint at least 1 unit to the sender first, then test with `amount > balance`. See `references/common-pitfalls.md` §6c. Selector for the revert: `0x5ff91cdc`.

### Pattern 3: Encrypted Conditional Logic (use `select`, never `if`)

```solidity
// WRONG — leaks which branch was taken
if (FHE.decrypt(condition)) { x = a; } else { x = b; }

// CORRECT — preserves confidentiality
euint64 result = FHE.select(condition, a, b);
```

### Pattern 4: Input Validation

```solidity
// Single encrypted input:
function deposit(externalEuint64 encAmount, bytes calldata inputProof) external {
    euint64 amount = FHE.fromExternal(encAmount, inputProof);
}

// Multiple encrypted inputs share ONE proof:
function swap(externalEuint64 encIn, externalEuint64 encOut, bytes calldata inputProof) external {
    euint64 amountIn = FHE.fromExternal(encIn, inputProof);   // Same proof
    euint64 amountOut = FHE.fromExternal(encOut, inputProof); // Same proof
}

// For already-verified handles (e.g., from another contract):
function processHandle(euint64 amount) external {
    require(FHE.isSenderAllowed(amount), "Sender not allowed");
}
```

### Pattern 5: Cross-Contract FHE (the 5-step pattern)

```solidity
// 1. Encrypt amount
euint64 encAmount = FHE.asEuint64(amount);
// 2. Grant transient access to the receiving contract
FHE.allowTransient(encAmount, address(token));
// 3. Call the other contract
euint64 transferred = token.confidentialTransferFrom(msg.sender, address(this), encAmount);
// 4. Grant persistent access to THIS contract for future use
FHE.allowThis(transferred);  // equivalent to FHE.allow(transferred, address(this))
// 5. Store the handle
escrow.budget = transferred;
```

Step 2 uses `allowTransient` (cheaper, same-tx only). Step 4 uses `allow` (persistent, needed for future transactions).

For collecting ERC-7984 token payments (lottery, escrow, payroll), see the step-by-step recipe in **[references/erc7984-guide.md](references/erc7984-guide.md)#recipe-fund-a-contract**.

### Pattern 6: ERC-7984 Confidential Token (Zama's core standard)

ERC-7984 is the FHE equivalent of ERC-20. Use `@openzeppelin/confidential-contracts`:

```bash
npm install @openzeppelin/confidential-contracts@^0.4.0 @fhevm/solidity@^0.11.1 @openzeppelin/contracts@^5.6.1
```

Key differences from ERC-20:
- **Function names**: `confidentialTransfer`, `confidentialBalanceOf`, `confidentialTotalSupply` (NOT `transfer`/`balanceOf`)
- **Operator model**: `setOperator(address, uint48 until)` — time-based, NOT amount-based approval
- **Transfers return `euint64`** (actual transferred amount), NOT `bool`
- **Silently sends 0** on insufficient balance (no revert)
- **Default decimals: 6** (not 18)
- **Events**: `ConfidentialTransfer(from, to, encAmount)` with encrypted handle

```solidity
import {ERC7984} from "@openzeppelin/confidential-contracts/token/ERC7984/ERC7984.sol";

contract MyToken is ZamaEthereumConfig, ERC7984, Ownable2Step {
    constructor() ERC7984("MyToken", "MTK", "https://example.com/token.json") Ownable(msg.sender) {}
    function mint(address to, uint64 amount) external onlyOwner { _mint(to, FHE.asEuint64(amount)); }
}
```

> **Warning**: The old `fhevm-contracts` package was **archived in June 2025**. Use `@openzeppelin/confidential-contracts` for all new development.

See **[references/erc7984-guide.md](references/erc7984-guide.md)** for complete interface, operator model, wrap/unwrap, extensions, and cross-contract patterns.

### Pattern 7: Snapshot-Then-Reveal (long-lived public decryption)

Whenever you have an **encrypted aggregate that mutates between request and finalize** (running tally, max-of-N, monthly volume, all-time-highest, leaderboard top-K, lifetime tip total, jar balance), you MUST snapshot the handle at request time. Use the snapshot — not the live storage value — in `FHE.checkSignatures` at finalize time.

**Why:** every FHE op produces a fresh handle. If a user action mutates the aggregate during the seconds-long KMS roundtrip, the storage handle is now different from the one the KMS signed. `FHE.checkSignatures` reverts with a signature mismatch.

```solidity
contract MonthlyReveal is ZamaEthereumConfig {
    euint64 private _monthlyTotal;        // mutates on every contribution()
    bytes32 private _pendingRevealHandle; // captured at request time
    uint64  public  revealedTotal;

    function requestReveal() external {
        FHE.makePubliclyDecryptable(_monthlyTotal);
        _pendingRevealHandle = FHE.toBytes32(_monthlyTotal);  // ← SNAPSHOT
    }

    function finalizeReveal(bytes calldata cleartexts, bytes calldata proof) external {
        require(_pendingRevealHandle != bytes32(0), "no reveal pending");
        bytes32[] memory handles = new bytes32[](1);
        handles[0] = _pendingRevealHandle;                     // ← snapshotted, not live
        FHE.checkSignatures(handles, cleartexts, proof);
        revealedTotal = uint64(abi.decode(cleartexts, (uint256)));
        _pendingRevealHandle = bytes32(0);                     // clear so a stale finalize can't re-fire
    }

    function contribute(externalEuint64 enc, bytes calldata p) external {
        _monthlyTotal = FHE.add(_monthlyTotal, FHE.fromExternal(enc, p));
        FHE.allowThis(_monthlyTotal);  // re-grant ACL on the new handle (the snapshot still works)
    }
}
```

**KMS hardening note:** a handle that was produced ONLY by `FHE.asEuint64(0)` (constructor init, never touched by a real FHE op) will fail `publicDecrypt` with `KMSInvalidSigner`. Guard `requestReveal` with a `if (eventCount == 0) revert NoActivityYet();` check, or pre-mutate the aggregate before exposing it.

For concurrent in-flight reveals (multiple requestIds outstanding) key the snapshot by request id — `mapping(uint256 => bytes32)` — see the unwrap recipe in `references/decryption-guide.md`. Also see `templates/confidential-tip-jar.sol` and `templates/confidential-lottery.sol` for the full shape in production templates.

### Privileged-Role Pattern Selection

Most FHEVM contracts need an admin role (mint, end auction, settle, slash, etc.). Pick by lifecycle:

| Use case | Pattern | Why |
|---|---|---|
| One-time deploy admin, never changes | `address public immutable owner` + `if (msg.sender != owner) revert NotOwner();` | Cheapest, no transfer surface, no `Ownable` 2-tx ownership-handoff confusion |
| Rotatable admin (production tokens, DAOs) | `Ownable2Step` from `@openzeppelin/contracts/access/Ownable2Step.sol` | Two-step transfer prevents typo'd owner = locked contract |
| Multiple roles (minter / pauser / treasurer) | `AccessControl` from `@openzeppelin/contracts` | Granular, on-chain visible role members |
| Critical mainnet (slashing, > $100k TVL) | Multisig (Safe / Gnosis) as the owner of `Ownable2Step` | Operational checks-and-balances, recovery from key loss |

See **[references/security-checklist.md](references/security-checklist.md)** for the full table with code examples.

## Critical Anti-Patterns

### 1. Branching on encrypted values [CRITICAL — leaks confidential data]

```solidity
// WRONG: if (FHE.decrypt(isEligible)) { grant(); }
// CORRECT:
euint64 reward = FHE.select(isEligible, fullReward, FHE.asEuint64(0));
```

`if/require/assert` with encrypted booleans reveals the value to validators by observing which branch executes.

### 2. Forgetting `FHE.allowThis()` [CRITICAL — contract loses access to its data]

```solidity
// WRONG: balances[user] = FHE.add(balances[user], amount);
// CORRECT:
balances[user] = FHE.add(balances[user], amount);
FHE.allowThis(balances[user]);
FHE.allow(balances[user], user);
```

### 3. Dividing by encrypted value [CRITICAL — does not compile]

```solidity
// WRONG: FHE.div(amount, encDivisor);
// CORRECT:
euint64 result = FHE.div(amount, uint64(100));  // Plaintext divisor only
```

### 4. Using `require()` for balance checks [CRITICAL — leaks balance info]

```solidity
// WRONG: require(balance >= amount, "Insufficient");
// CORRECT:
ebool sufficient = FHE.ge(balance, amount);
euint64 actual = FHE.select(sufficient, amount, FHE.asEuint64(0));
```

> **Plaintext role checks are FINE.** This anti-pattern is specifically about `require`/`if` on a *decrypted encrypted value*. Plaintext checks on plaintext state are correct and safe: `if (msg.sender != creator) revert NotCreator();`, `Ownable.onlyOwner`, `setOperator(address, expiry)` reverting with `ERC7984UnauthorizedSpender`, `block.timestamp >= deadline`, and similar — these reveal nothing confidential because the role / address / timestamp / operator relationship was never private to begin with. The leak only happens when a plaintext branch decision is derived from an encrypted comparison.

### 5. Random in view functions [HIGH — runtime revert]

Random generation mutates on-chain PRNG state. Must be a state-changing function.

### 6. Bounded random with non-power-of-2 [HIGH — runtime revert]

`FHE.randEuint8(6)` reverts at runtime ("upperBound must be a power of 2"). Use `FHE.randEuint8(8)` (power of 2), then `FHE.rem(result, uint8(6))` to project into [0, 6).

### 7. Deprecated TFHE library for new contracts [HIGH — wrong API]

Use `FHE` from `@fhevm/solidity/lib/FHE.sol`, not `TFHE` from `fhevm/lib/TFHE.sol`.

### 8. Unprotected view function returning encrypted handles [MEDIUM]

Always check `FHE.isAllowed(value, msg.sender)` before returning encrypted handles.

### 9. Ordering comparisons on euint256 [HIGH — does not compile]

`euint256` only supports `eq`/`ne`. Use `euint128` or smaller for `gt`/`lt`/`ge`/`le`.

### 10. Exceeding 2048-bit decryption limit [MEDIUM]

Max per request: 32 × euint64, or 16 × euint128, or 256 × euint8. Total ≤ 2048 bits.

## Do NOT Generate These (Common AI Hallucinations)

| Hallucinated | Reality |
|---|---|
| `FHE.decrypt()` in Solidity | Decryption is off-chain only via Relayer SDK |
| `Gateway.requestDecryption()` | Removed in v0.9+. Use self-relaying pattern |
| `ebytes64`, `eint8` (signed types) | These types do not exist in FHE.sol |
| `FHE.safeAdd()` / `safeSub()` / `safeMul()` | No `FHE.safe*`. Use `FHESafeMath.tryAdd/trySub` from `@openzeppelin/confidential-contracts` |
| `TFHE.*` for new contracts | Use `FHE.*` from `@fhevm/solidity` |
| `FHE.div(encrypted, encrypted)` | Divisor must be plaintext |
| `FHE.sealoutput()` | Does not exist in v0.11 |
| `FHE.allowForDecryption()` | Correct name: `FHE.makePubliclyDecryptable()` |
| `randEuint8Bounded(n)` | Correct: `FHE.randEuint8(uint8 upperBound)` |

## Battle Scars (Real-World Lessons)

1. **"Why did 0 tokens arrive?"**: We deployed a confidential ERC-20 and tested a transfer of 1000 tokens from an account with 500. No revert, no error, transaction succeeded. The recipient got 0. We spent hours debugging before realizing: confidential transfers (ERC-7984 and any sound FHE token) NEVER revert on insufficient balance — the `select(ge(balance, amount), amount, 0)` pattern quietly substitutes 0 to avoid leaking the balance. **Detection**: comparing handles before/after does **not** work — every FHE op produces a fresh non-deterministic ciphertext, so handles always change. Real detection requires (a) decrypting the recipient's balance and verifying it grew by `amount`, or (b) parsing the `ConfidentialTransfer` event's `amount` handle and decrypting that.

2. **"Why does the proof fail cross-contract?"**: Our vault contract received an encrypted input from a user and forwarded it to a token contract via `confidentialTransferFrom`. Proof validation failed every time. Root cause: input proofs are bound to `msg.sender`. When the vault forwarded the call, `msg.sender` changed from user to vault. **Fix**: 2-transaction flow — user sends to token directly, then vault triggers logic separately.

3. **"Why can't the contract read its own storage?"**: Contract stored an encrypted treasury balance. First transaction worked. Second transaction reverted with "not allowed." Root cause: `FHE.add()` creates a NEW handle — the old handle's ACL doesn't carry over. We were missing `FHE.allowThis()` after the operation. **Rule**: every FHE operation + state store = must call `allowThis`.

4. **"Gas doesn't change with amount?"**: We benchmarked `FHE.add(1, 2)` vs `FHE.add(MAX_UINT64, MAX_UINT64)` — identical gas. This is intentional: variable gas would leak information about encrypted values. Don't try to optimize around amount-based gas.

5. **"Same amount, different ciphertext?"**: Frontend test encrypted 100 twice and compared handles — they differed. This is correct: encryption is non-deterministic by design. Deterministic encryption would leak information through ciphertext equality comparison.

## Supported Types & Operations Quick Reference

| Type | Bits | Arithmetic | Comparison | Bitwise | Random |
|------|------|-----------|------------|---------|--------|
| `ebool` | 1 | - | eq, ne | and, or, xor, not | randEbool |
| `euint8` | 8 | add, sub, mul, div*, rem*, neg, min, max | all | all + shifts | randEuint8 |
| `euint16` | 16 | same | all | all + shifts | randEuint16 |
| `euint32` | 32 | same | all | all + shifts | randEuint32 |
| `euint64` | 64 | same | all | all + shifts | randEuint64 |
| `euint128` | 128 | same | all | all + shifts | randEuint128 |
| `euint256` | 256 | neg only | eq, ne only | all + shifts | randEuint256 |
| `eaddress` | 160 | - | eq, ne | - | - |

*`div` and `rem`: plaintext right-hand operand ONLY. Shift amounts are always `euint8`/`uint8`.

## Known Limitations

These are intrinsic constraints of the FHEVM stack you must design around — none of them have workarounds the skill is hiding from you.

| Limitation | What it means in practice |
|---|---|
| **Sepolia only (no mainnet yet)** | FHEVM is testnet-only as of mid-2026. Production deploys land on Sepolia + ZAMA's gateway chain (10901). Mainnet support is a future Zama protocol milestone. |
| **HCU per-tx 20 M / sequential depth 5 M** | Every FHE op consumes Homomorphic Compute Units. Long encrypted loops (e.g. eq-chains over many buckets) hit the budget. Plan with [`references/gas-optimization.md`](references/gas-optimization.md). |
| **`FHE.div` / `FHE.rem`: plaintext divisor only** | Encrypted-by-encrypted division is unsupported. Either pre-divide off-chain or restructure the math (see common-pitfalls.md §3). |
| **`euint256`: equality only** | No `gt/lt/ge/le/min/max/add/sub/mul`. Only `eq` / `ne`. Use `euint128` whenever you need ordering or arithmetic on big integers. |
| **`view` / `pure` cannot use state-mutating FHE ops** | All FHE arithmetic / comparison / select / random ops touch coprocessor state. Read-only getters can return handles but cannot compute them. |
| **ERC-7984 transfers silently return 0 on insufficient balance** | `confidentialTransfer` / `confidentialTransferFrom` never revert. **Always use the `euint64 transferred` return value, not the requested amount, for downstream computation.** Otherwise a caller with 0 balance can drain pools that compute outputs from the requested input. |
| **Hardhat 2 only** | `@fhevm/hardhat-plugin@0.4.x` is incompatible with Hardhat 3. Pin `hardhat@^2.28.4`. |
| **`@zama-fhe/relayer-sdk` exact pin** | The plugin hard-fails on `0.4.3`. Always `npm install --save-exact @zama-fhe/relayer-sdk@0.4.1`. See common-pitfalls.md §5b. |
| **Input proofs bound to msg.sender** | A proof generated for user A cannot be forwarded by contract X to contract Y as if X were the sender. Use a 2-tx flow or re-encrypt. |

## Reference Files

For detailed guides, read the corresponding reference file:

- **[Type System](references/type-system.md)** — Complete type details, casting, cross-type operations, operator overloads
- **[ACL Patterns](references/acl-patterns.md)** — allow, allowThis, allowTransient, makePubliclyDecryptable, delegation
- **[Input Proofs](references/input-proofs.md)** — Client-side encryption, contract-side validation, multi-input proofs
- **[Decryption Guide](references/decryption-guide.md)** — User decryption (EIP-712), public decryption, checkSignatures, delegated
- **[Testing Guide](references/testing-guide.md)** — Mock mode, Sepolia testing, decrypt helpers, test patterns
- **[Frontend Integration](references/frontend-integration.md)** — Gen-2 Relayer SDK setup, encryption, decryption, React patterns (still used in Hardhat tests)
- **[SDK v3 Guide](references/sdk-v3-guide.md)** — Gen-3 `@zama-fhe/sdk@3.x` (NEW): `ZamaSDK`, `Token`, viem/ethers signers, RelayerWeb, IndexedDB cache, migration map
- **[React SDK Guide](references/react-sdk-guide.md)** — Gen-3 `@zama-fhe/react-sdk@3.x` (NEW): `ZamaProvider`, `WagmiSigner`, `useConfidentialBalance`, `useConfidentialTransfer`, react-query integration
- **[ERC-7984 Guide](references/erc7984-guide.md)** — Confidential tokens, wrap/unwrap, ConfidentialERC20, OpenZeppelin contracts
- **[Common Pitfalls](references/common-pitfalls.md)** — Extended anti-patterns with explanations and battle scars
- **[Gas Optimization](references/gas-optimization.md)** — FHE-specific gas patterns, type sizing, batching strategies
- **[Time-Based Accrual](references/time-based-accrual.md)** — Encrypted interest / rewards / vesting math; the 37%-truncation footgun and the `ACCRUAL_SCALE=1000` pre-multiplier pattern
- **[Security Checklist](references/security-checklist.md)** — Production security audit checklist for FHEVM contracts
- **[Zama Upstream](references/zama-upstream.md)** — Pinned versions, upstream repo links, attribution, upgrade procedure when Zama ships breaking changes
- **[Foundry vs Hardhat](references/foundry-vs-hardhat.md)** — Why this skill targets Hardhat, when Foundry is the right call, direct equivalents cheat-sheet for translating patterns to forge-fhevm

## Templates

- **[templates/confidential-erc20.sol](templates/confidential-erc20.sol)** — ERC-7984 confidential token with encrypted balances
- **[templates/encrypted-voting.sol](templates/encrypted-voting.sol)** — Confidential yes/no voting with encrypted tallies
- **[templates/multi-option-voting.sol](templates/multi-option-voting.sol)** — N-bucket (3-5) token-weighted DAO vote (FHE.eq+select chain, multi-input single-proof, dynamic-N reveal)
- **[templates/blind-auction.sol](templates/blind-auction.sol)** — Sealed-bid auction with encrypted bids AND encrypted bidder identity (eaddress)
- **[templates/vickrey-auction.sol](templates/vickrey-auction.sol)** — Sealed-bid SECOND-price (Vickrey) auction with ERC-7984 escrow, top-2 chained-`gt`+`select` ranking, mixed-type 2-handle reveal, 4-state lifecycle (Bidding→Ended→Revealed→Settled)
- **[templates/confidential-amm.sol](templates/confidential-amm.sol)** — Single-pair constant-product AMM (encrypted reserves, plaintext LP supply, 0.3% fees, multiplicative-invariant gate with silent refund on caller-cheat, ACL-gated TVL reveal). Crystallises the design from `common-pitfalls.md` §11b.
- **[templates/cdp-vault.sol](templates/cdp-vault.sol)** — Collateral-debt position vault (encrypted collateral/debt, plaintext oracle, public-decryptable liquidation flag with `checkSignatures` round-trip, Pitfall #11 mul-overflow mitigations: 60/100→3/5, 80/100→4/5)
- **[templates/confidential-tip-jar.sol](templates/confidential-tip-jar.sol)** — Aggregate-with-private-contributors pattern (tip jars, fundraisers, GoFundMe). Per-tipper ACL split + Pattern 7 (Snapshot-Then-Reveal) for the lifetime total
- **[templates/confidential-lottery.sol](templates/confidential-lottery.sol)** — Encrypted-tickets lottery / commit-reveal raffle. Range allocation `[start, start+count)`, publicly-decryptable winner-flag proof, multi-round state machine, partial-pay handling on `confidentialTransferFrom` silent zero
- **[templates/hardhat.config.ts](templates/hardhat.config.ts)** — Production-ready Hardhat configuration
- **[templates/tsconfig.json](templates/tsconfig.json)** — TypeScript configuration. Without explicit `include` list, Hardhat 2 + Node infers `rootDir` from the test dir and crashes with `TS5011`. This template covers the canonical FHEVM dApp surface (config + scripts + tests + typechain).
- **[templates/test-template.ts](templates/test-template.ts)** — ERC-7984 test boilerplate matched to `templates/confidential-erc20.sol` (mint, confidentialTransfer, operator model, ACL)
- **[templates/test-erc7984-template.ts](templates/test-erc7984-template.ts)** — Generic ERC-7984 test boilerplate using a fixture pattern
- **[templates/test-multi-option-voting.ts](templates/test-multi-option-voting.ts)** — Full E2E test for the multi-option voting template (multi-input proof, dynamic-N publicDecrypt, winningChoice tie-break, out-of-range fall-through)
- **[templates/test-vickrey-auction.ts](templates/test-vickrey-auction.ts)** — Full E2E test for the Vickrey auction (4-bidder top-2 ranking, winner-pays-second-price refund, seller withdraw, loser refunds, state-machine guards)
- **[templates/test-confidential-amm.ts](templates/test-confidential-amm.ts)** — 16-test suite for the AMM (initialization, liquidity provision, A↔B swaps, cheating-trader silent refund, fee accumulation, TVL UX)
- **[templates/test-cdp-vault.ts](templates/test-cdp-vault.ts)** — 22-test suite for the CDP vault (deposit, borrow at safe LTV, silent over-LTV cap, repay, full liquidation flow with `publicDecrypt` + `checkSignatures` + on-chain liquidate, withdraw with debt-gate, access control)
- **[templates/confidential-escrow.sol](templates/confidential-escrow.sol)** — Escrow with encrypted deposits, release, refund, arbiter dispute
- **[templates/confidential-swap.sol](templates/confidential-swap.sol)** — Token swap with encrypted amounts and fee collection
- **[templates/react-dashboard.tsx](templates/react-dashboard.tsx)** — React component built on the foundational SDK (`@zama-fhe/relayer-sdk/web`): manual encryption, decrypt balance, encrypted transfer. For non-token contracts or when you need fine-grained control.
- **[templates/react-dashboard-v3.tsx](templates/react-dashboard-v3.tsx)** — React component built on the high-level Token API (`@zama-fhe/react-sdk`): hooks-based balance, transfer, shield/unshield, with `<ZamaProvider>` setup. For ERC-7984 token UIs.
- **[templates/vite-frontend/](templates/vite-frontend/)** — Vanilla-JS Vite frontend (no React). Lowest-friction prototype; includes the load-bearing `optimizeDeps.exclude` rule that keeps the relayer-sdk WASM init path intact.
- **[templates/deploy-template.ts](templates/deploy-template.ts)** — Plain-ethers deploy script (intentionally NOT `hardhat-deploy` — that package's transitive `zksync-web3@0.14.4` crashes on ethers v6; see Self-Correction Table)
- **[templates/onchain-e2e.ts](templates/onchain-e2e.ts)** — Paste-ready Sepolia end-to-end script: deploy → encrypt via `@zama-fhe/relayer-sdk/node` → submit → KMS roundtrip → on-chain `checkSignatures` proof → save evidence JSON. Adapt the contract method block to your dApp.
- **[templates/mock-erc20.sol](templates/mock-erc20.sol)** — Mock ERC-20 for testing wrap/unwrap flows

## Validation

Run the validator against your contracts to catch common FHEVM mistakes before deployment:

```bash
# macOS / Linux / Git Bash / WSL:
chmod +x scripts/validate-fhevm.sh
./scripts/validate-fhevm.sh ./contracts

# Native Windows PowerShell (no bash needed):
powershell -ExecutionPolicy Bypass -File scripts/validate-fhevm.ps1 ./contracts
```

Both scripts run the same 13 checks and use identical exit codes (0 = pass, 1 = warnings, 2 = errors). Checks: missing `allowThis`, deprecated TFHE usage, encrypted divisors, branching on encrypted values, missing `cancun` evmVersion, deprecated Gateway pattern, euint256 ordering comparisons, hallucinated FHE functions, non-existent encrypted types, discarded `confidentialTransferFrom` return value (drain bug), and more.

---
> Source: [Himess/fhevm-skill](https://github.com/Himess/fhevm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
