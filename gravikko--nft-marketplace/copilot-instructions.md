## nft-marketplace

> NFT Marketplace - A comprehensive Solidity-based marketplace for creating, trading, auctioning, and staking NFTs.

# Project Context

NFT Marketplace - A comprehensive Solidity-based marketplace for creating, trading, auctioning, and staking NFTs.

**Tech Stack:**
- Solidity `^0.8.24`
- Foundry (Forge, Anvil, Cast)
- OpenZeppelin Contracts & Contracts Upgradeable
- Chainlink VRF v2.5
- pnpm workspace (monorepo)

**Core Modules:**
- **Collection Factory**: Create ERC721 collections via beacon proxies
- **Marketplace**: EIP-712 signed orders/offers for trading
- **Auctions**: Time-based bidding system
- **Staking**: Stake NFTs for ETH rewards
- **Governance**: MultisigTimelock for admin operations

---

## Commands

```bash
# Build contracts
cd foundry && forge build

# Run all tests
cd foundry && forge test

# Run specific test file
cd foundry && forge test --match-path test/unit/marketplace/*.sol

# Run specific test function
cd foundry && forge test --match-test test_ExecuteOrder_Succeeds

# Run with verbosity (show logs, traces)
cd foundry && forge test -vvv

# Run with gas reporting
cd foundry && forge test --gas-report

# Format code
cd foundry && forge fmt

# Local deployment
anvil  # terminal 1
cd foundry && forge script script/DeployFactory.s.sol --rpc-url http://localhost:8545 --broadcast  # terminal 2

# Generate coverage report
cd foundry && forge coverage

# Frontend (not yet implemented)
cd frontend && pnpm dev
```

---

## Architecture Overview

### Directory Structure

```
nft-marketplace/
├── foundry/                    # Smart contracts (main codebase)
│   ├── src/                    # Contract implementations
│   │   ├── ERC721Collection.sol       # NFT collection (upgradeable, VRF, royalties)
│   │   ├── Factory.sol                # Creates collections via beacon proxy
│   │   ├── Marketplace.sol            # Orders & offers with EIP-712 signatures
│   │   ├── Auction.sol                # Time-based auctions
│   │   ├── Staking.sol                # NFT staking rewards
│   │   ├── MultisigTimelock.sol       # Governance (multisig + timelock)
│   │   ├── VRFAdapter.sol             # Chainlink VRF v2.5 integration
│   │   ├── ERC721CollectionBeacon.sol # Beacon for collection proxies
│   │   ├── SwapAdapter.sol            # ETH/WETH wrapper
│   │   └── interfaces/
│   │       ├── IFactory.sol
│   │       └── IERC721Collection.sol
│   ├── test/
│   │   ├── Base.t.sol                 # Base test contract with constants
│   │   ├── unit/                      # Unit tests by module
│   │   │   ├── auction/
│   │   │   ├── erc721collection/
│   │   │   ├── factory/
│   │   │   ├── marketplace/
│   │   │   └── staking/
│   │   └── helpers/                   # Test utilities & mocks
│   │       ├── DeployHelpers.s.sol
│   │       ├── MarketplaceTestHelper.sol
│   │       ├── FactoryHelper.sol
│   │       ├── AuctionTestBase.sol
│   │       ├── StakingTestHelper.sol
│   │       └── Mocks.sol
│   ├── script/
│   │   └── DeployFactory.s.sol
│   ├── lib/                           # Dependencies (OZ, Chainlink, forge-std)
│   ├── foundry.toml
│   └── remappings.txt
├── frontend/                   # React + Vite (skeleton only)
├── CLAUDE.md
└── pnpm-workspace.yaml
```

### Proxy Architecture

**UUPS Proxies** (upgradeable single-instance contracts):
- Factory, Marketplace, Auction, Staking, VRFAdapter, MultisigTimelock

**Beacon Proxies** (multiple instances sharing one implementation):
- All ERC721Collection instances created via Factory

### Contract Dependencies

```
MultisigTimelock
       ↓ (controls)
    Factory ←────────────────┐
       ↓ (creates)           │
ERC721CollectionBeacon       │
       ↓ (points to)         │
ERC721Collection (impl)      │
       ↓ (instances use)     │
BeaconProxy                  │
       ↓ (trades on)         │
  Marketplace ───────────────┤
       ↓                     │
   Auction ──────────────────┤
       ↓                     │
  StakingNFT ────────────────┘
```

### Key Constants

| Constant | Value | Description |
|----------|-------|-------------|
| MAX_SUPPLY | 20,000 | Max tokens per collection |
| MAX_ROYALTY | 1000 (10%) | Max royalty in basis points |
| MAX_MARKETPLACE_FEE | 500 (5%) | Max marketplace fee |
| MAX_AUCTION_FEE | 500 (5%) | Max auction fee |
| MIN_DURATION | 30 minutes | Min auction duration |
| MAX_DURATION | 7 days | Max auction duration |
| MIN_PRICE | 1000 wei | Minimum price allowed |
| FEE_DENOMINATOR | 10,000 | Basis points divisor |

---

## Key Patterns

### Writing New Contracts

**UUPS Upgradeable Contract Template:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {Initializable} from "openzeppelin-contracts-upgradeable/proxy/utils/Initializable.sol";
import {UUPSUpgradeable} from "openzeppelin-contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import {ReentrancyGuard} from "openzeppelin-contracts/utils/ReentrancyGuard.sol";

interface IMultisigTimelock {
    function verifyCurrentTransaction() external view;
}

contract NewContract is Initializable, UUPSUpgradeable, ReentrancyGuard {
    // ========== STATE VARIABLES ==========
    address private _multisigTimelock;

    // ========== ERRORS ==========
    error NotAMultisigTimelock();
    error ZeroAddress();

    // ========== MODIFIERS ==========
    modifier onlyMultisig() {
        if (msg.sender != _multisigTimelock) revert NotAMultisigTimelock();
        IMultisigTimelock(_multisigTimelock).verifyCurrentTransaction();
        _;
    }

    // ========== CONSTRUCTOR ==========
    constructor() {
        _disableInitializers();
    }

    // ========== INITIALIZATION ==========
    function initialize(address multisigTimelock) external initializer {
        if (multisigTimelock == address(0)) revert ZeroAddress();
        _multisigTimelock = multisigTimelock;
    }

    // ========== EXTERNAL FUNCTIONS ==========
    // ... your functions here

    // ========== INTERNAL FUNCTIONS ==========
    function _authorizeUpgrade(address) internal override onlyMultisig {}
}
```

### Error Handling

**Always use custom errors (no require statements):**
```solidity
// Define errors
error IncorrectPrice();
error InsufficientBalance();
error NotAnOwner();
error ZeroAddress();

// Use with revert
if (msg.value < price) revert InsufficientBalance();
if (receiver == address(0)) revert ZeroAddress();
```

**Try-catch for external calls:**
```solidity
try IVRFAdapter(_vrfAdapter).requestRandomness(tokenId) returns (uint256 requestId) {
    emit VRFRequested(requestId, tokenId);
} catch {
    _vrfPendingReveals[tokenId] = false;
    revert VRFRequestFailed();
}
```

### Access Control Pattern

All admin functions use MultisigTimelock:
```solidity
modifier onlyMultisig() {
    if (msg.sender != _multisigTimelock) revert NotAMultisigTimelock();
    IMultisigTimelock(_multisigTimelock).verifyCurrentTransaction();
    _;
}

function adminFunction() external onlyMultisig {
    // Protected logic
}
```

### Safe Transfers

**ETH transfers:**
```solidity
(bool success, ) = payable(recipient).call{value: amount}("");
if (!success) revert TransferFailed();
```

**ERC721 transfers:**
```solidity
collection.safeTransferFrom(from, to, tokenId);
```

### EIP-712 Signatures

```solidity
// In initialize()
__EIP712_init("ContractName", "1");

// Define typehash
bytes32 private constant ORDER_TYPEHASH = keccak256(
    "Order(address seller,uint256 collectionId,uint256 tokenId,uint256 price,uint256 nonce,uint256 deadline)"
);

// Create struct hash
bytes32 structHash = keccak256(abi.encode(
    ORDER_TYPEHASH,
    order.seller,
    order.collectionId,
    order.tokenId,
    order.price,
    order.nonce,
    order.deadline
));

// Verify signature
address signer = ECDSA.recover(_hashTypedDataV4(structHash), signature);
```

### Writing Tests

**Test file structure:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {DeployHelpers} from "../helpers/DeployHelpers.s.sol";

contract NewContractTest is DeployHelpers {
    function setUp() public override {
        super.setUp();
        // Additional setup
    }

    // Success cases
    function test_FunctionName_Succeeds() public {
        // Arrange
        // Act
        // Assert
    }

    // Revert cases
    function test_FunctionName_RevertIf_Condition() public {
        vm.expectRevert(ContractName.ErrorName.selector);
        // Call that should revert
    }
}
```

**Common test patterns:**
```solidity
// Set msg.sender
vm.prank(USER1);
contract.function();

// For multiple calls
vm.startPrank(USER1);
contract.function1();
contract.function2();
vm.stopPrank();

// Give ETH
vm.deal(USER1, 100 ether);

// Time manipulation
vm.warp(block.timestamp + 1 days);

// Block manipulation
vm.roll(block.number + 100);

// Expect revert
vm.expectRevert(Contract.ErrorName.selector);
```

---

## Style Guide

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Contracts | PascalCase | `MarketplaceNFT`, `ERC721Collection` |
| Interfaces | I + PascalCase | `IFactory`, `IERC721Collection` |
| Functions | camelCase | `executeOrder`, `getSupply` |
| Private/internal vars | _camelCase | `_multisigTimelock`, `_nextTokenId` |
| Constants | SCREAMING_SNAKE | `MAX_SUPPLY`, `FEE_DENOMINATOR` |
| Events | PascalCase | `OrderExecuted`, `TokenRevealed` |
| Errors | PascalCase | `IncorrectPrice`, `NotAnOwner` |
| Structs | PascalCase | `Order`, `InitConfig` |
| Modifiers | camelCase | `onlyMultisig`, `nonReentrant` |

### Function Prefixes

| Prefix | Usage |
|--------|-------|
| `get*` | View functions returning data |
| `set*` | Admin functions setting state |
| `_*` | Internal/private functions |
| `activate*` / `stop*` | Toggle contract state |

### Code Organization (in order)

1. SPDX license identifier
2. Pragma statement
3. Imports (upgradeable first, then utils, then project)
4. Interfaces (if local)
5. Contract declaration
6. Constants/immutables
7. State variables
8. Events
9. Errors
10. Modifiers
11. Constructor
12. Initialize function
13. External functions
14. Public functions
15. Internal functions
16. Private functions

### Formatting

**Numbers:** Use underscores for readability
```solidity
uint256 public constant MAX_SUPPLY = 20_000;
uint256 private constant FEE_DENOMINATOR = 10_000;
```

**Imports:** Group by source
```solidity
// OpenZeppelin Upgradeable
import {Initializable} from "openzeppelin-contracts-upgradeable/...";
import {UUPSUpgradeable} from "openzeppelin-contracts-upgradeable/...";

// OpenZeppelin Utils
import {ReentrancyGuard} from "openzeppelin-contracts/...";
import {IERC20} from "openzeppelin-contracts/...";

// Project imports
import {IFactory} from "./interfaces/IFactory.sol";
```

**Comments:** NatSpec for public/external functions
```solidity
/**
 * @dev Executes a signed order, transferring NFT to buyer
 * @param order The order struct containing trade details
 * @param signature EIP-712 signature from seller
 */
function executeOrder(Order memory order, bytes memory signature) external payable {
    // Implementation
}
```

### Test File Naming

Pattern: `{ContractName}_{TestCategory}.t.sol` or `{ContractName}{TestCategory}.t.sol`

Examples:
- `Factory_Setup.t.sol`
- `MarketplaceOrders.t.sol`
- `AuctionBiddingTest.t.sol`

### Test Function Naming

```solidity
// Success: test_ActionDescription_Succeeds
function test_ExecuteOrder_Succeeds() public {}

// Revert: test_ActionDescription_RevertIf_Condition
function test_ExecuteOrder_RevertIf_InvalidSignature() public {}

// Alternative revert naming
function test_RevertWhen_PriceIsZero() public {}
```

---

## Common Tasks

### Adding a new contract function
1. Add function to contract in `foundry/src/`
2. Add to interface if public in `foundry/src/interfaces/`
3. Write tests in `foundry/test/unit/<module>/`
4. Run `forge test`

### Creating a new test file
1. Create file in `foundry/test/unit/<module>/`
2. Inherit from appropriate helper (`DeployHelpers`, `MarketplaceTestHelper`, etc.)
3. Override `setUp()` and call `super.setUp()`
4. Write test functions following naming convention

### Modifying marketplace logic
- Main file: `foundry/src/Marketplace.sol`
- Tests: `foundry/test/unit/marketplace/`
- Helper: `foundry/test/helpers/MarketplaceTestHelper.sol`

### Working with collections
- Implementation: `foundry/src/ERC721Collection.sol`
- Factory: `foundry/src/Factory.sol`
- Beacon: `foundry/src/ERC721CollectionBeacon.sol`
- VRF: `foundry/src/VRFAdapter.sol`

---

## Current State

**Complete:**
- All smart contracts
- Comprehensive test suite (25+ test files)
- Basic deployment script

**Incomplete:**
- Frontend (skeleton only)
- Subgraph indexer (The Graph)
- Full deployment scripts for testnets
- Contract verification scripts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gravikko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
