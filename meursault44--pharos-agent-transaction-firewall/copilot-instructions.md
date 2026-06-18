## pharos-agent-transaction-firewall

> Pre-execution safety firewall for AI agents on Pharos Atlantic testnet and Pharos mainnet. Use when an agent or user wants to inspect a transaction before signing or sending it, decode calldata, classify approvals/transfers/admin calls, detect unlimited approvals, unknown spenders, risky token interactions, suspicious value transfers, or decide whether an onchain action should be ALLOW, WARN, or BLOCK.


# Pharos Agent Transaction Firewall

Use this skill as a mandatory pre-signing safety layer for AI agents on Pharos. It inspects a proposed transaction or an existing transaction hash and returns a policy decision: `ALLOW`, `WARN`, or `BLOCK`.

This skill is read-only. It never asks for a private key and never sends transactions.

The implementation is written and executed as TypeScript at `scripts/inspect-transaction.ts`. Run `npm install` once, then use the npm scripts to invoke the TypeScript entrypoints.

## Automatic Agent Rule

Agents must run this firewall automatically before any Pharos write transaction, including approvals, transfers, swaps, contract writes, deployments, admin operations, and batch operations. The user should not need to explicitly ask for inspection.

Decision handling:

- If the firewall returns `BLOCK`, do not sign or broadcast the transaction.
- If the firewall returns `WARN`, ask for explicit user confirmation before continuing.
- If the firewall returns `ALLOW`, continue only when the decoded action matches the user's original intent.

## Inputs

Support either:

- Proposed transaction: `--to <ADDRESS> --data <CALLDATA> --value <WEI_OR_ETH>`.
- Existing transaction: `--tx <TX_HASH>`.

Optional:

- `--from <ADDRESS>` to include sender context.
- `--network atlantic-testnet` or `--network mainnet`; default to `mainnet` for transaction inspection.
- `--value-eth <AMOUNT>` for human-friendly native value input.

Use `mainnet` for Pharos Pacific Mainnet / chain `1672`.

## Fast Path

Prepare the TypeScript runtime:

```bash
npm install
npm run build
```

Inspect a proposed ERC20 approval:

```bash
npm run inspect -- --network mainnet --to <TOKEN_ADDRESS> --data <CALLDATA>
```

Inspect an already submitted transaction:

```bash
npm run inspect -- --network mainnet --tx <TX_HASH>
```

Run the natural-language chat agent demo:

```bash
npm run agent:chat -- "swap 10 PDOG to PROS on Pharos mainnet"
```

## Decision Policy

Return:

- `ALLOW`: normal interaction with low static risk.
- `WARN`: unknown contract, nonzero value transfer, limited approval, admin call, unverified risk, or incomplete decoding.
- `BLOCK`: unlimited approval to an unknown spender, transaction to an EOA with calldata, no target contract for a contract call, native value plus unknown calldata, known dangerous selector, or obviously malformed calldata.

The core inspector must also produce `tokenRisk` for token-touching transactions:

- Known local Pharos token: lower token-risk score.
- Unknown token with readable ERC20 metadata: warn as unverified/dust-risk until liquidity, source, and official links are verified.
- Locally flagged unsolicited/dust-risk token: classify as `UNKNOWN_UNSOLICITED_OR_UNVERIFIED`.
- Unknown/dust-risk token plus unlimited approval or unknown spender: `BLOCK`.

Do not present the decision as a guarantee. Explain what was decoded and what the user or agent should verify next.

## Workflow

1. Resolve network RPC from local `assets/networks.json` when available, otherwise from `pharos-skill-engine/assets/networks.json`.
2. If `--tx` is provided, fetch transaction data with `eth_getTransactionByHash`.
3. Validate addresses, value, and calldata.
4. Fetch target bytecode and identify whether `to` is an EOA or contract.
5. Read ERC20 metadata when the target looks token-like and compare the address with the local Pharos known-token list.
6. Build `tokenRisk` with classification, score, metadata, warnings, and blocks.
7. Decode known selectors:
   - ERC20: `approve`, `transfer`, `transferFrom`.
   - ERC721/ERC1155: `setApprovalForAll`, `safeTransferFrom`.
   - ownership/admin: `transferOwnership`, `renounceOwnership`, `pause`, `unpause`.
   - DEX-like calls: common swap selectors.
8. For approvals, inspect spender bytecode and known-token status.
9. Detect unlimited approvals, dust/scam-token interactions, and suspicious patterns.
10. Produce a structured JSON report with:
   - `decision`
   - `severity`
   - `tokenRisk`
   - decoded action
   - positive signals
   - warning/block reasons
   - recommended next step

For policy details, read `references/firewall-policy.md`.

---
> Source: [Meursault44/pharos-agent-transaction-firewall](https://github.com/Meursault44/pharos-agent-transaction-firewall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
