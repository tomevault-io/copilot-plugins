## polytorus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PolyTorus is a cutting-edge modular blockchain platform designed for the post-quantum era. It features a **4-layer modular architecture** with separate crates for execution, settlement, consensus, and data availability, providing unprecedented customization and optimization capabilities.

## Essential Commands

### Build & Development
```bash
# Development build
cargo build

# Release build  
cargo build --release

# Run all tests
cargo test

# Run individual layer tests
cargo test -p execution
cargo test -p settlement  
cargo test -p consensus
cargo test -p data-availability

# Run specific test by name
cargo test test_block_creation -- --nocapture
```

### Code Quality & Linting
```bash
# Check for compilation errors
cargo check

# Run clippy linter
cargo clippy -- -D warnings

# Format code
cargo fmt

# Build without warnings (zero tolerance policy)
cargo build 2>&1 | grep -i warning && echo "Warnings found!" || echo "Clean build!"
```

### Running the Application
```bash
# Start the blockchain node
cargo run start

# Show help for available commands
cargo run -- --help

# Process a transaction
cargo run send --from alice --to bob --amount 100

# Check blockchain status
cargo run status
```

## Architecture Overview

### 4-Layer Modular Architecture
The project is organized as a Rust workspace with separate crates for each layer:

1. **`crates/execution/`** - Transaction processing and WASM smart contracts
   - WASM-based contract execution with gas metering
   - Account-based state management with rollback capabilities
   - Host functions for blockchain operations (balance queries, transfers)
   - **Tests**: 4 comprehensive test functions

2. **`crates/settlement/`** - Dispute resolution and finalization
   - Optimistic rollup processing with fraud proof verification
   - Challenge submission and processing with time-based expiration
   - Settlement history tracking and penalty system
   - **Tests**: 6 comprehensive test functions

3. **`crates/consensus/`** - Block ordering and validation  
   - Proof-of-Work consensus with configurable difficulty
   - Block validation (structure, PoW, timestamps, transactions)
   - Validator management and mining capabilities
   - **Tests**: 8 comprehensive test functions

4. **`crates/data-availability/`** - Data storage and distribution
   - Merkle tree construction and proof verification
   - Data storage with metadata and integrity checks
   - Network-aware data distribution simulation
   - **Tests**: 9 comprehensive test functions

5. **`crates/traits/`** - Shared interfaces and types
   - Common traits for all layers (ExecutionLayer, SettlementLayer, etc.)
   - Shared data structures (Transaction, Block, Hash, etc.)
   - Result types and error handling

6. **External Wallet** - Cryptographic wallet functionality
   - **Repository**: https://github.com/PolyTorus/wallet
   - HD wallet implementation with BIP32/BIP44 support
   - Address generation and key pair management
   - Digital signature creation and verification
   - Multi-signature wallet capabilities
   - **Integration**: Available as `wallet` crate dependency

### Main Orchestrator (`src/main.rs`)
The `PolyTorusBlockchain` struct coordinates all layers:
- Manages layer lifecycle (initialization, coordination)
- Processes transactions through all layers sequentially
- Provides CLI interface with clap for command handling
- Supports both default and custom layer configurations

### Configuration System
Configuration is managed through:
- **`config/modular.toml`** - Layer-specific settings (gas limits, difficulty, retention periods)
- **Layer configs** - Each layer has its own configuration struct with sensible defaults
- **Test configurations** - Special configurations for testing (e.g., difficulty=0 for fast mining)

## Development Guidelines

### Code Quality Standards
- **Zero dead code policy**: All variables and functions must be used
- **Zero compiler warnings**: No warnings allowed in builds
- **Comprehensive testing**: Each layer has extensive unit tests
- **Proper error handling**: Use `anyhow::Result` consistently

### Testing Architecture
Tests are distributed across the crates:
- **Unit tests**: Located in each crate's `src/lib.rs` 
- **Integration tests**: Located in `src/main.rs` for full blockchain functionality
- **Layer isolation**: Each layer can be tested independently
- **Configuration testing**: Tests use difficulty=0 for fast mining where appropriate

### Testing Best Practices
```bash
# Test individual layers during development
cargo test -p execution
cargo test -p consensus

# Run full integration tests
cargo test

# Test with output for debugging
cargo test test_name -- --nocapture
```

### Layer Implementation Patterns
Each layer follows a consistent pattern:
1. **Configuration struct** with `Default` implementation
2. **Main implementation struct** (e.g., `PolyTorusExecutionLayer`)
3. **Trait implementation** for the layer interface
4. **Comprehensive unit tests** in `#[cfg(test)]` module
5. **Error handling** using `anyhow::Result`

### Common Development Tasks

#### Adding New Layer Functionality
1. Define the interface in `crates/traits/src/lib.rs`
2. Implement in the appropriate layer crate
3. Add comprehensive tests
4. Update the main orchestrator if needed
5. Update configuration if new settings are required

#### Integrating Wallet Functionality
The external wallet crate provides comprehensive cryptographic capabilities:

```rust
// Import wallet functionality
use wallet::{
    HDWallet, Wallet, Address, KeyPair, Signature, 
    AddressFormat, WalletError
};

// Create a new HD wallet
let mnemonic = HDWallet::generate_mnemonic()?;
let wallet = HDWallet::from_mnemonic(&mnemonic, None)?;

// Generate addresses
let address = wallet.derive_address(0, false)?; // First receiving address
let change_address = wallet.derive_address(0, true)?; // First change address

// Create key pairs for signing
let keypair = wallet.derive_keypair(0, false)?;
let signature = keypair.sign(message_hash)?;

// Verify signatures
let is_valid = keypair.verify(message_hash, &signature)?;
```

**Wallet Features:**
- **HD Wallet**: BIP32/BIP44 hierarchical deterministic wallet support
- **Address Generation**: Multiple address formats (Legacy, SegWit, Native SegWit)
- **Key Management**: Secure key pair generation and management
- **Digital Signatures**: ECDSA signature creation and verification
- **Multi-signature**: Multi-signature wallet creation and transaction signing
- **Mnemonic**: BIP39 mnemonic phrase generation and recovery

#### Debugging Layer Interactions
The orchestrator processes transactions through layers in this order:
1. **Execution**: Process and validate transaction
2. **Data Availability**: Store transaction data
3. **Consensus**: Add to pending transactions for block creation
4. **Settlement**: Handle any disputes or challenges

#### Mining and Block Creation
- Default difficulty is configured for reasonable mining times
- Tests use `difficulty=0` for instant mining
- Custom configurations can be passed to `PolyTorusBlockchain::new_with_configs()`

### Configuration Management
Layer configurations are defined in:
- `ExecutionConfig`: Gas limits, WASM settings
- `SettlementConfig`: Challenge periods, batch sizes
- `ConsensusConfig`: Block time, difficulty, block size limits  
- `DataAvailabilityConfig`: Retention periods, network settings

### Performance Considerations
- Each layer runs independently and can be optimized separately
- WASM execution includes gas metering for resource control
- Data availability includes configurable retention and replication
- Consensus supports configurable difficulty for different deployment scenarios

### Current Development Status
All layers are **fully implemented and tested**:
- **31 total tests** across all layers and main orchestrator
- **Zero compiler warnings** across the entire codebase
- **Production-ready** modular architecture
- **Clean separation of concerns** between layers

---
> Source: [PolyTorus/polytorus](https://github.com/PolyTorus/polytorus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
