## cursor-rules

> yarn add vechain-sdk-js

# VeChain SDK JS MCP Rules (Context7)

## Installation

Install the SDK:
```bash
yarn add vechain-sdk-js
```

## Client Setup

Create a Thor client for the desired network:
```typescript
import { ThorClient } from 'vechain-sdk-js';
const thorClient = ThorClient.at('https://testnet.veblocks.net');
```

## Transaction Patterns

### Single Clause Transaction (VET Transfer)
```typescript
import { Clause, Address, VET, Transaction, Secp256k1 } from 'vechain-sdk-js';

const clauses = [
  Clause.transferVET(Address.of('0x...'), VET.of(10000))
];
const gas = Transaction.intrinsicGas(clauses).wei;
const body = {
  chainTag: 0x4a,
  blockRef: '0x0000000000000000',
  expiration: 32,
  clauses,
  gasPriceCoef: 128,
  gas,
  dependsOn: null,
  nonce: 12345678
};
const privateKey = await Secp256k1.generatePrivateKey();
const signedTx = Transaction.of(body).sign(privateKey);
const encodedRaw = signedTx.encoded;
const decodedTx = Transaction.decode(encodedRaw, true);
```

### Multiple Clauses (VET + VTHO)
```typescript
const clauses = [
  Clause.transferVET(Address.of('0x...'), VET.of(10000)),
  Clause.transferVTHOToken(Address.of('0x...'), VTHO.of(10000))
];
const gas = Number(Transaction.intrinsicGas(clauses).wei);
// ...build and sign as above
```

### Simulate Transaction
```typescript
const simulatedTx = await thorClient.transactions.simulateTransaction(
  clauses,
  { caller: '0x...' }
);
```

### Fee Delegation
```typescript
import { VeChainProvider, ProviderInternalBaseWallet } from 'vechain-sdk-js';
const provider = new VeChainProvider(
  thorClient,
  new ProviderInternalBaseWallet([
    { privateKey: senderPrivateKey, address: senderAddress }
  ], {
    gasPayer: { gasPayerPrivateKey }
  }),
  true // enable fee delegation
);
const signer = await provider.getSigner(senderAddress);
```

## Contract Patterns

### Deploy Contract
```typescript
const contractFactory = thorClient.contracts.createContractFactory(
  contractAbi,
  contractBytecode,
  signer
);
const contract = await (await contractFactory.startDeployment()).waitForDeployment();
```

### Interact with Contract
```typescript
await (await contract.transact.deposit({ value: 1000 })).wait();
const balance = await contract.read.getBalance(address);
```

### Batch Read Operations
```typescript
const results = await thorClient.contracts.executeMultipleClausesCall([
  contract.clause.totalSupply(),
  contract.clause.name(),
  contract.clause.symbol(),
  contract.clause.decimals()
]);
```

## Gas Estimation & Fee Calculation
```typescript
const gasResult = await thorClient.transactions.estimateGas(clauses, senderAddress);
const defaultBodyOptions = await thorClient.transactions.fillDefaultBodyOptions();
const txBody = await thorClient.transactions.buildTransactionBody(
  clauses,
  gasResult.totalGas,
  {
    chainTag: 0x4a,
    blockRef: '0x0000000000000000',
    expiration: 32,
    gasPriceCoef: 128,
    dependsOn: null,
    nonce: 12345678,
    ...defaultBodyOptions
  }
);
```

## Debugging & Tracing

### Trace Transaction Clause
```typescript
const result = await thorClient.debug.traceTransactionClause({
  target: {
    blockId: BlockId.of('0x...'),
    transaction: BlockId.of('0x...'),
    clauseIndex: 0
  },
  config: {}
}, 'call');
console.log(result);
```

### Trace Contract Call
```typescript
const result = await thorClient.debug.traceContractCall({
  target: {
    to: Address.of('0x...'),
    data: HexUInt.of('0x...'),
    value: VET.of(0)
  },
  options: {
    caller: '0x...',
    gasPayer: '0x...',
    expiration: 18,
    blockRef: '0x...'
  },
  config: {}
}, 'call');
console.log(result);
```

## Transaction API Reference
- `sendTransaction(signedTx: Transaction): Promise<SendTransactionResult>`
- `getTransaction(id: string): Promise<TransactionDetail | null>`
- `getTransactionReceipt(id: string): Promise<TransactionReceipt | null>`

## Testing
- Run all tests:
```bash
yarn test
```
- Run documentation examples as tests:
```bash
yarn test:examples
```
- With thor-solo node:
```bash
yarn test:examples:solo
```

## Best Practices
- Always simulate and estimate gas before sending transactions.
- Use fee delegation for user-friendly dApps.
- Batch contract reads for efficiency.
- Use try/catch for all async blockchain calls.
- Validate all addresses and amounts before sending.
- Use environment variables for sensitive config.
- Test contract deployment and interaction in a local/solo environment before mainnet.

## Directory Structure (Recommended)
```
lib/
  vechain-sdk/        # SDK utilities and client setup
contracts/
  abi/                # Contract ABIs
  bytecode/           # Contract bytecode
scripts/
  deploy.ts           # Deployment scripts
  interact.ts         # Interaction scripts
```

## References
- [Official VeChain SDK JS Docs](mdc:https:/github.com/vechain/vechain-sdk-js)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/databeforedishonor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/databeforedishonor)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
