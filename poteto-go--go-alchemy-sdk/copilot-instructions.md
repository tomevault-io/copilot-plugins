## go-alchemy-sdk

> Golang sdk for alchemy, inspired by https://github.com/alchemyplatform/alchemy-sdk-js.

# Go-Alchemy-Sdk

Golang sdk for alchemy, inspired by https://github.com/alchemyplatform/alchemy-sdk-js.

## Tree

always update

```
.
├── CONTRIBUTING.md
├── GEMINI.md
├── LICENSE
├── README.md
├── _fixture
│   ├── artifacts
│   │   └── PotetoStorage.go
│   ├── build
│   │   └── PotetoStorage.abi
│   ├── contracts
│   │   └── PotetoStorage.sol
│   └── kurtosis
│       ├── network_params.yaml
│       └── scripts
│           ├── clean-up.sh
│           ├── detect-rpc-port.sh
│           ├── install-kurtosis.sh
│           └── run.sh
├── alchemymock
│   ├── mock.go
│   └── mock_test.go
├── constant
│   ├── errors.go
│   └── ether_methods.go
├── coverage.out
├── docs
│   ├── README.md
│   ├── build
│   │   ├── img
│   │   │   ├── logo.svg
│   │   │   └── poteto-go.png
│   │   ├── index.html
│   │   └── sitemap.xml
│   ├── docs
│   │   ├── core-namespace
│   │   │   ├── EstimateGas.md
│   │   │   ├── GetBlock.md
│   │   │   ├── GetBlockNumber.md
│   │   │   ├── GetCode.md
│   │   │   ├── GetGasPrice.md
│   │   │   ├── GetStorageAt.md
│   │   │   ├── GetTransaction.md
│   │   │   ├── GetTransactionReceipt.md
│   │   │   ├── GetTransactionReceipts.md
│   │   │   ├── PeerCount.md
│   │   │   └── _category_.json
│   │   ├── development
│   │   │   ├── Mock.md
│   │   │   └── _category_.json
│   │   ├── quickstart.md
│   │   ├── setting
│   │   │   ├── _category_.json
│   │   │   └── alchemySetting.md
│   │   ├── transact-namespace
│   │   │   ├── WaitDeployed.md
│   │   │   ├── WaitMined.md
│   │   │   └── _category_.json
│   │   ├── tutorial
│   │   │   ├── _category_.json
│   │   │   └── deployContract.md
│   │   └── wallet
│   │       ├── Connect.md
│   │       ├── ContractTransact.md
│   │       ├── ContractTransactNoWait.md
│   │       ├── DeployContract.md
│   │       ├── DeployContractNoWait.md
│   │       ├── GetAddress.md
│   │       ├── GetBalance.md
│   │       ├── PendingNonceAt.md
│   │       ├── ResetPool.md
│   │       ├── SendTransaction.md
│   │       ├── SignTx.md
│   │       ├── Wallet.md
│   │       └── _category_.json
│   ├── docusaurus.config.ts
│   ├── node_modules
├── e2e
│   └── scenario_test.go
├── ether
│   ├── ether.go
│   ├── ether_config.go
│   ├── ether_config_test.go
│   ├── ether_core_test.go
│   ├── ether_network_test.go
│   ├── ether_transact_test.go
│   └── ether_wallet_test.go
├── examples
│   └── deploy-contract
│       ├── abi
│       │   └── Storage.go
│       ├── build
│       │   └── Storage.abi
│       ├── contracts
│       │   └── Storage.sol
│       ├── go.mod
│       ├── go.sum
│       └── main.go
├── gas
│   ├── alchemy.go
│   ├── alchemy_config.go
│   ├── alchemy_config_test.go
│   ├── alchemy_setting.go
│   ├── alchemy_setting_test.go
│   ├── alchemy_test.go
│   ├── http_provider.go
│   └── http_provider_test.go
├── go.mod
├── go.sum
├── internal
│   ├── backoff.go
│   ├── backoff_test.go
│   ├── batch_request.go
│   ├── batch_request_benchmark_test.go
│   ├── batch_request_test.go
│   ├── chain_id_list.go
│   ├── dispatch.go
│   ├── dispatch_test.go
│   ├── errors.go
│   └── errors_test.go
├── namespace
│   ├── core_namespace.go
│   ├── core_namespace_test.go
│   ├── transact_namespace.go
│   └── transact_namespace_test.go
├── playground
│   ├── a.sh
│   ├── ethSepolia.json
│   ├── main.go
│   └── polygonAmoy.json
├── types
│   ├── alchemy_provider.go
│   ├── arg.go
│   ├── backoff.go
│   ├── block.go
│   ├── ether_api.go
│   ├── filter.go
│   ├── network.go
│   ├── token_balance.go
│   ├── token_metadata.go
│   └── transaction.go
├── utils
│   ├── block.go
│   ├── block_test.go
│   ├── body.go
│   ├── body_test.go
│   ├── fetch.go
│   ├── fetch_test.go
│   ├── hex.go
│   ├── hex_test.go
│   ├── random.go
│   ├── random_test.go
│   ├── transaction.go
│   ├── transcation_test.go
│   ├── transformer.go
│   ├── transformer_test.go
│   ├── validator.go
│   └── validator_test.go
├── wallet
│   ├── wallet.go
│   └── wallet_test.go
└── whitelist.txt
```

## Rule

- Use TDD for development.
- This project internal use go-ethereum client.
  - If you can use go-ethereum, you should use go-ethereum.
- If you create public API, you should document.

## Testing

### Mocking with alchemymock

This project uses `alchemymock` to mock Alchemy RPC responses (which `ethclient` uses internally) for unit tests.

**Usage:**

1.  **Initialize Mock:**
    Use `newAlchemyMockOnEtherTest(t)` (helper in `ether` package tests) or `alchemymock.NewAlchemyHttpMock`.

    ```go
    alchemyMock := newAlchemyMockOnEtherTest(t)
    defer alchemyMock.DeactivateAndReset()
    ```

2.  **Register Responder:**
    Use `RegisterResponder` to define the JSON-RPC response for a specific method.

    ```go
    // Example: Mocking eth_call
    expectedRes := "0x0000...01" // Hex result
    alchemyMock.RegisterResponder("eth_call", `{"jsonrpc":"2.0","id":1,"result":"`+expectedRes+`"}`)
    ```

    The response string must be a valid JSON-RPC response format.

**Note:** This allows testing functions that interact with `ethclient` without making actual network requests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/poteto-go)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/poteto-go)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
