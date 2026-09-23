## next-injective-git

> Next Injective Git stores packfiles on IPFS and control-plane state in a

# Repository Guidance

Next Injective Git stores packfiles on IPFS and control-plane state in a
non-upgradeable Injective EVM Suite. Ordinary clients trust only one configured
`SuiteDirectory`; never add a direct module address, compatibility backend,
legacy fallback, proxy, diamond, or `delegatecall` path.

**Current Project Status:** P0 baseline complete (2026-08-19), P1 testnet
deployment preparation in progress. See [docs/project-status.md](docs/project-status.md)
(English) or [docs/project-status-zh.md](docs/project-status-zh.md) (中文) for
a comprehensive status overview.

## Why EVM V2

CosmWasm V1 Push ran natively on Linux, but the supported Windows workflow
required a WSL2 compatibility environment for the Linux CLI, `injectived`, and
Kubo. EVM V2 is specifically the second-generation control plane built on
Injective EVM so ordinary Windows operation can use the native Windows CLI,
encrypted EVM keystore, JSON-RPC, and native storage tooling. WSL2 and
`injectived` are not prerequisites for the EVM V2 product path.

The EVM V2 product-generation name is distinct from the on-chain Suite protocol
version, which is currently 3. Completion requires clean native Windows and
Linux acceptance; source code or CI configuration alone is not evidence. See
[ADR 0001](docs/adr/0001-evm-v2-runtime-and-migration-scope.md).

IPFS/Kubo is the currently implemented pack-storage adapter, not the permanent
product core. Amazon S3 and Cloudflare R2 adapters are a planned direction, not
current functionality. The existing immutable Suite accepts only `ipfs://`
pack URIs, so storage neutrality requires a reviewed successor protocol and
migration rather than a documentation-only endpoint switch. See
[ADR 0002](docs/adr/0002-pluggable-pack-storage.md).

## Source Boundaries

- `contracts/evm-v2`: nine Solidity contracts, fixed `solc 0.8.24`, checked ABI
  and deployment artifacts.
- `cli`: Go CLI, Git remote helper, EVM transactor, deployment tooling, and
  deterministic offline migration tooling.
- `web`: React/Vite and viem. Wallet sends must specify legacy type, estimated
  gas, and gas price at least `160000000 wei`, then verify receipt status.
- `archive/cosmwasm-v1`: isolated read-only historical source and evidence
  tooling. Never import it into ordinary runtime, CI, or release paths.

Published profiles intentionally leave `SuiteDirectory` empty until real
deployment and cutover evidence is approved. Do not add an address from a test,
example, source artifact, or unreviewed deployment.

## Required Checks

```sh
(cd cli && go vet ./... && go test ./...)
(cd web && npm ci && npm run test:api && npm run typecheck && npm run build)
npm ci --prefix contracts/evm-v2
bash scripts/ci/evm-v2-check.sh --required
bash scripts/ci/suite-readiness.sh --required
bash scripts/migration/migration-cutover-readiness-test.sh
# Or via Make: make vet test check
```

CI also runs Go race tests, native Windows tests, source/profile guards, and
release asset verification. A missing compiler or Foundry executable must fail
required gates rather than become a passing skip.

## Safety Properties

- The client verifies chain ID, suite version 3, active state, every module code
  hash, and every module binding before reads or writes.
- One `EVMTransactor` owns nonce lookup, gas estimation, legacy signing,
  broadcast, and bounded receipt polling. An uncertain receipt returns the tx
  hash and invalidates cached nonce state.
- Push pins the pack durably before updating the ref. A failed transaction must
  retain retryable content. A force push publishes a self-contained pack.
- Moderation hooks are mandatory for Core ref mutation and Economic sponsor
  mutation. Recovery is the only ownership-recovery capability.
- Snapshot imports are ordered, bounded, payload-hashed, rolling-committed, and
  finalized once. Directory activation occurs only after all modules verify.
- Username migration requires all historical deposits refunded and escrow
  balance zero. Pending temporal operations are re-created after cutover.
- Deployment, plans, manifests, journals, receipts, fixed-block state, and
  approval are immutable evidence. Never overwrite or manufacture them.

## Coding Conventions

User-visible CLI messages are bilingual through `i18n`. Read-only commands use
contract-only validation and must not require a key. Git-compatible unknown
commands may be forwarded to Git; inspect command dispatch before adding names.

Use checked-in ABIs through go-ethereum or viem. Do not hand-code selectors or
word decoders. Preserve the established Git pack and current IPFS gateway
behavior until a separately reviewed storage protocol is implemented. Do not
claim S3 or R2 support from roadmap documentation alone.

Deployment is restricted to rotated encrypted testnet keys. Production needs a
separate multisig/timelock governance design. Never deploy or broadcast merely
to satisfy a test or documentation task.

---
> Source: [Hny0305Lin/next-injective-git](https://github.com/Hny0305Lin/next-injective-git) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
