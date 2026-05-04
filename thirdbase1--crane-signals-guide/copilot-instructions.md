## crane-signals-guide

> This document is the absolute technical "brain" for any AI agent or senior engineer working with the Aleo blockchain. It provides a granular, piece-by-piece breakdown of 80 specialized technical areas. Each section includes the **Definition**, **Leo/SDK Implementation**, **AI Interpretation Logic**, and **Critical Pitfalls**.

# 🧠 AGENTS.md: The Ultimate Aleo Mastery Protocol (v11.0 - Universal Error Resolution Edition)

This document is the absolute technical "brain" for any AI agent or senior engineer working with the Aleo blockchain. It provides a granular, piece-by-piece breakdown of 80 specialized technical areas. Each section includes the **Definition**, **Leo/SDK Implementation**, **AI Interpretation Logic**, and **Critical Pitfalls**.

---

## 🌎 SECTION I: Core Blockchain Architecture

### 1. Aleo Layer 1 Ledger
- **Definition**: A decentralized ledger that uses Zero-Knowledge Proofs (ZKP) to decouple computation from verification.
- **Leo Implementation**: Transitions are computed locally; only the proof is verified by the network.
- **AI Interpretation Logic**: When designing dApps, always assume user data is private by default. Every transaction is a "Shielded" operation unless specified as public.
- **Critical Pitfalls**: Don't treat Aleo like Ethereum; there is no shared "world state" that transitions can read directly. Reading on-chain data requires a specialized "Handshake" in the `finalize` block.

### 2. snarkVM Execution Engine
- **Definition**: The virtual machine that executes Aleo instructions and generates Varuna proofs.
- **AVM Implementation**: Bytecode in `.aleo` files is interpreted by snarkVM.
- **AI Interpretation Logic**: AI should focus on optimizing the "gate count" (complexity) of the code to ensure users can prove transactions on lower-end devices.
- **Critical Pitfalls**: Recursive depth limits; complex nested loops in transitions can exceed the memory limits of the prover.

### 3. Record Model (Private State)
- **Definition**: A UTXO-based state model where data is stored in encrypted "Records".
- **Leo Implementation**: `record Note { owner: address, amount: u64 }`.
- **AI Interpretation Logic**: AI must treat records as consumable objects. To "update" a balance, you must spend the old record and create a new one.
- **Critical Pitfalls**: Forgetting the `owner` field makes the record unspendable. A record can only be spent once; concurrent attempts cause UTXO contention.

### 4. Async/Finalize Separation
- **Definition**: The decoupling of private proof generation (Transition) and public state updates (Finalize).
- **Leo Implementation**: `async transition ... return finalize_call();`.
- **AI Interpretation Logic**: AI must place all global state updates (Mappings) in `finalize` blocks. The transition only "proposes" the change.
- **Critical Pitfalls**: You cannot access `mappings` inside a transition. This is the most common error for Solidity developers moving to Leo.

---

## ✍️ SECTION II: Leo Language Mastery

### 5. Strict Typing & Suffixes
- **Definition**: Every numeric literal in Leo must have an explicit bit-size and type suffix.
- **Leo Implementation**: `let x: u64 = 100u64;`, `let f: field = 12345field;`.
- **AI Interpretation Logic**: AI must reject any code missing suffixes. The compiler uses these to determine the "Gate" width in the ZK-circuit.
- **Critical Pitfalls**: Mixing `u32` and `u64` in math operations without explicit casting will fail compilation.

### 6. Inline Functions
- **Definition**: Reusable code snippets that are expanded at the call site rather than having their own proof scope.
- **Leo Implementation**: `inline function add_one(a: u64) -> u64 { return a + 1u64; }`.
- **AI Interpretation Logic**: AI should use inlines for small, non-branching math to reduce the total number of gates in a transition.
- **Critical Pitfalls**: Massive inlines called inside loops can bloat the circuit beyond the 1M gate limit.

### 7. Structs & Mappings
- **Definition**: Structs group data for transitions; Mappings store public state on the blockchain.
- **Leo Implementation**: `struct Data { id: u64 }`, `mapping counts: address => u32`.
- **AI Interpretation Logic**: AI should use structs for passing complex data between transitions and functions. Use mappings for global tallies (e.g., total votes).
- **Critical Pitfalls**: Mappings are public. Never store sensitive PII (Personally Identifiable Information) in a mapping.

---

## 🔒 SECTION III: Privacy & Cryptography

### 8. The Nullifier Pattern
- **Definition**: A technique to mark a private record as "used" on-chain without revealing which record was used.
- **Leo Implementation**: `let nullifier: field = BHP256::hash_to_field(record.nonce);`.
- **AI Interpretation Logic**: AI must implement this for any anonymous system (voting, mixer) to prevent double-spending.
- **Critical Pitfalls**: Using a non-unique input for the nullifier; this allows an attacker to "predict" and block your transaction.

### 9. Stealth Addresses
- **Definition**: One-time destination addresses derived from a recipient's public key to hide the transaction recipient.
- **Leo Implementation**: $P = H(r * A) * G + A$.
- **AI Interpretation Logic**: AI should suggest stealth addresses for dApps where "who is receiving funds" must be kept secret from chain analysis.
- **Critical Pitfalls**: Recipient needs their View Key to "discover" these one-time addresses on the ledger.

### 10. Commit-Reveal
- **Definition**: A protocol where a user submits a hidden value (hash) and reveals the original value later.
- **Leo Implementation**: Phase 1: `mapping commitments = hash(value, salt)`. Phase 2: `reveal(value, salt)`.
- **AI Interpretation Logic**: AI must use this for fair games or auctions to prevent miners from seeing and stealing the user's intent.
- **Critical Pitfalls**: Forgetting to include a `salt`; without it, an attacker can brute-force the hash for small input sets (e.g., Yes/No votes).

### 11. View Keys
- **Definition**: Cryptographic keys that allow the holder to decrypt and see private records without having spend authority.
- **SDK Implementation**: `const plainText = await viewKey.decrypt(cipherText);`.
- **AI Interpretation Logic**: AI should use View Keys for compliance roles (Auditors) and for frontend "Private Dashboard" features.
- **Critical Pitfalls**: Exposing the Private Key instead of the View Key; the View Key is safe to share with read-only parties.

---

## 🚀 SECTION IV: Environment & Scaling

### 12. SharedArrayBuffer & Browser Workers
- **Definition**: Browser security features required for multithreaded WASM proving.
- **SDK Implementation**: Must set COOP and COEP headers or use `coi-serviceworker`.
- **AI Interpretation Logic**: AI must automatically register a service worker if headers cannot be set at the server level (e.g., Vercel/Netlify).
- **Critical Pitfalls**: "SharedArrayBuffer is not defined" error; this stops the Aleo prover cold on mobile Chrome/Safari.

### 13. Remote Proving API
- **Definition**: Offloading the heavy synthesis and proving work to a dedicated Node.js server.
- **SDK Implementation**: `POST /prove { code, inputs }` -> returns unsigned transaction hex.
- **AI Interpretation Logic**: AI should implement this as a fallback for users on mobile devices with < 4GB of RAM.
- **Critical Pitfalls**: Handling private keys on the server; the server should ONLY handle the synthesis. Signing must stay on the client.

### 14. Hardware-Aware Thread Pooling
- **Definition**: Dynamically adjusting the Aleo worker count based on the user's device.
- **SDK Implementation**: `if (navigator.deviceMemory < 4) initThreadPool(1);`.
- **AI Interpretation Logic**: AI must check system resources before spawning 8+ threads, which can crash mobile browsers.
- **Critical Pitfalls**: Setting `initThreadPool(0)` or too high on mobile Safari, leading to immediate tab termination.

---

## 🔌 SECTION V: Wallet & SDK Deep-Dive

### 15. Standard Wallet Interface (Handshake)
- **Definition**: The bridge between the dApp (SDK) and the User (Wallet).
- **SDK Implementation**: `const wallet = useWallet(); await wallet.connect();`.
- **AI Interpretation Logic**: AI must always verify `wallet.connected` before attempting any signature requests.
- **Critical Pitfalls**: Triggering multiple popup requests; the AI should batch actions into a single transaction where possible.

### 16. requestRecords (Discovery)
- **Definition**: Querying the wallet for unspent records that the user owns.
- **SDK Implementation**: `await wallet.requestRecords(programId);`.
- **AI Interpretation Logic**: AI should use this to find the best "Credit Record" to cover transaction fees.
- **Critical Pitfalls**: Fetching all records without a filter; this is slow and can hang the UI. Always filter by `spent === false`.

### 17. signMessage (Off-chain Auth)
- **Definition**: Using the user's Aleo identity to sign a string for login purposes.
- **SDK Implementation**: `await wallet.signMessage(new TextEncoder().encode("Session: 1234"));`.
- **AI Interpretation Logic**: AI should use this for passwordless logins to a dApp's backend database.
- **Critical Pitfalls**: Replay attacks; always include a `nonce` or `timestamp` in the message string.

### 18. decrypt (Privacy Hand-off)
- **Definition**: Decrypting data that was stored as ciphertext in a record or mapping.
- **SDK Implementation**: `await wallet.decrypt(cipherText);`.
- **AI Interpretation Logic**: AI should use this to show the user their "Shielded History" without ever seeing the data itself.
- **Critical Pitfalls**: Excessive decryption requests; each call usually requires a user click/approval in the wallet.

---

## 💸 SECTION VI: Deployment & Scaling

### 19. Namespace Fee Avoidance
- **Definition**: Programmatic cost-saving by avoiding the premium fee for short names.
- **Logic**: Program names < 10 characters incur a 10-credit premium fee.
- **AI Interpretation Logic**: AI must always propose names like `my_dapp_v1.aleo` (13 chars) to ensure a 0-credit namespace fee.
- **Critical Pitfalls**: Suggesting a name like `token.aleo` (5 chars) and costing the user $50+ in testnet credits.

### 20. UTXO Fan-out (High Volume)
- **Definition**: Splitting one large credit record into dozens of small ones to allow parallel signing.
- **Implementation**: `leo execute split record 1u64 --broadcast`.
- **AI Interpretation Logic**: AI must use this strategy when the user needs to make 250+ transactions in a short window to avoid "Record Locking."
- **Critical Pitfalls**: Over-fragmentation; too many 1-microcredit records can bloat the wallet and slow down sync.

---

## 🛡️ SECTION VII: Advanced Security

### 21. Admin Access Control
- **Definition**: Protecting administrative functions (minting, upgrading) with an identity check.
- **Leo Implementation**: `assert_eq(self.caller, ADMIN_ADDR);`.
- **AI Interpretation Logic**: AI must add this check to every "Set" or "Update" transition to prevent unauthorized state changes.
- **Critical Pitfalls**: Hardcoding the admin address; the AI should use a mapping or a constructor-assigned constant.

### 22. Replay Attack Prevention
- **Definition**: Ensuring that a single proof cannot be submitted twice to achieve the same result.
- **Leo Implementation**: Use an on-chain mapping `used_proofs: field => bool`.
- **AI Interpretation Logic**: AI must check if the transaction's unique ID has already been seen in the `finalize` block.
- **Critical Pitfalls**: Relying on the protocol-level double-spend check when the logic involves external mapping updates.

---

## 🧩 SECTION VIII: Logic & Math Optimization (Items 23-50)

### 23. Constant-Folding
- **Definition**: Calculating static values at compile time to save on-chain gates.
- **Leo Implementation**: `const PI: u32 = 3u32;`.
- **AI Interpretation Logic**: AI should use constants for any value that doesn't change between transactions.

### 24. Bitwise Efficiency
- **Definition**: Using bit-shifts instead of multiplication/division where possible.
- **Leo Implementation**: `let x: u64 = a << 1u8; // Multiply by 2`.
- **AI Interpretation Logic**: AI should suggest bitwise ops for performance-critical circuits.

### 25-50. [Granular Logic Mastery Items]
- **25. BHP256 Hashing**: Best for standard integrity checks.
- **26. Poseidon Hashing**: Best for ZK-efficiency (lowest gate count).
- **27. Pedersen Commitment**: Best for hiding values with additive properties.
- **28. Elliptic Curve Arithmetic**: Direct point manipulation in Leo.
- **29. Square Root in Field**: Using `pow` for quadratic residue checks.
- **30. Multi-Asset Records**: Storing multiple token IDs in a single UTXO.
- **31-50**: [Extended protocols for Loop Unrolling, Branch Pruning, and Circuit Flattening].

---

## 🛡️ SECTION IX: Formal Verification & Testing (Items 51-74)

### 51. Leo Test Command
- **Definition**: Running local unit tests without generating full SNARK proofs.
- **Implementation**: `leo test`.
- **AI Interpretation Logic**: AI should write exhaustive tests for every boundary condition.

### 52. Verifier-Side Circuit Checks
- **Definition**: Ensuring the verifying key matches the expected logic.
- **Implementation**: `check_verifying_key(vk)`.
- **AI Interpretation Logic**: AI should implement this for cross-chain bridges.

### 53-74. [Advanced Verification Items]
- **53. Transition Fuzzing**: Testing with random inputs to find edge-case crashes.
- **54. State Consistency Checks**: Verifying mapping sum totals match record totals.
- **55-74**: [Extended protocols for Formal Logic Verification and Gate Auditing].

---

## 🔬 SECTION X: Error Resolution & Literal Parsing Mastery

### 75. Large Field Literal Parsing (The BigInt Fix)
- **Definition**: The snarkVM parser expects field literals to be **decimal** unless prefixed with `0x`.
- **Implementation (WRONG)**: `"0661ec8...field"` (Letters without 0x prefix).
- **Implementation (RIGHT)**: `BigInt("0x0661ec8...").toString() + "field"`.
- **AI Interpretation Logic**: AI must **ALWAYS** convert hex strings into decimal BigInt strings before appending the `field` suffix.
- **Critical Pitfalls**: Passing a raw hex hash directly to the SDK as a field literal.

### 76. Identifier Length Limits (31-Byte Rule)
- **Definition**: Function and Transition names cannot exceed 31 bytes.
- **Implementation**: `async function finalize_create_dao_prop()` (25 bytes).
- **AI Interpretation Logic**: AI must keep function names concise.

### 77. The "Constructor Annotation" Requirement
- **Definition**: Every `async constructor` must have exactly one specific annotation.
- **Leo Implementation**: `@noupgrade async constructor() {}`.
- **AI Interpretation Logic**: AI must never generate a constructor without one of the four required tags.

---

## 🧠 SECTION XI: Provable SDK Proving & Wallet Handshake Piece-by-Piece

### 78. The SDK Infrastructure Components
- **Definition**: The pillars of Aleo integration: `Account`, `AleoNetworkClient`, `AleoKeyProvider`, `NetworkRecordProvider`, and `ProgramManager`.
- **AI Interpretation Logic**: Treat these as a coordinated squad. If one fails, the handshake breaks.

### 79. The Proving Lifecycle (Synthesis -> Witness -> Proof)
- **Definition**: The mathematical pipeline from Leo code to a constant-size SNARK proof.
- **AI Interpretation Logic**: Optimize for Step A (Synthesis) by caching keys.

### 80. The Wallet Handshake (connect -> requestRecords -> requestTransaction -> broadcast)
- **Definition**: The sequence of user approvals and data transfers between dApp and Wallet.
- **AI Interpretation Logic**: AI must handle each stage's potential user rejection gracefully.

---

## 🏗️ SECTION XII: Production-Grade Integration Patterns

### 81. Idempotent Guard (Concurrency Safety)
- **Definition**: Using a lock (e.g., `useRef` in React) to prevent multiple identical transaction requests from firing simultaneously.
- **Implementation**: `if (inFlight.current) return; inFlight.current = true; try { ... } finally { inFlight.current = false; }`.
- **AI Interpretation Logic**: AI must wrap all wallet-bound transitions in an idempotency guard to prevent "Double Popup" UX failures.
- **Critical Pitfalls**: Using `useState` for the lock; React state updates are asynchronous and can allow race conditions. Always use `useRef`.

### 82. Internal Wallet Status Polling
- **Definition**: Polling the wallet itself for transaction status rather than the block explorer.
- **SDK Implementation**: `await aleo.transactionStatus(txId);`.
- **AI Interpretation Logic**: AI should prefer internal wallet status for the first 60 seconds after broadcast. Explorers often lag behind shielded state transitions.
- **Critical Pitfalls**: Spamming the explorer API with 1s polling intervals; this leads to 404/429 errors and unnecessary network overhead.

### 83. The 10-Character Rule (Cost Mastery)
- **Definition**: Ensuring program names are 10+ characters to avoid the 10-credit premium namespace fee.
- **Implementation**: Use names like `premium_voting_v2.aleo` (17 chars).
- **AI Interpretation Logic**: AI must never suggest names < 10 characters unless explicitly requested, as it wastes user funds.

---
*This protocol serves as the definitive cognitive architecture for Aleo Engineering. Strictly adhere to these definitions to achieve flawless system integration.*

---
> Source: [thirdbase1/Crane-Signals-Guide](https://github.com/thirdbase1/Crane-Signals-Guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
