## vehicle-net

> This project simulates a decentralized vehicle network where each **vehicle is a node** that can:

# CLAUDE.md

## Purpose

This project simulates a decentralized vehicle network where each **vehicle is a node** that can:

- Derive a unique Ethereum wallet from a shared HD mnemonic
- Sign vehicle condition reports (e.g., mileage, battery health, timestamp)
- Submit signed reports to a smart contract-based marketplace
- Enable buyers to verify the authenticity of a vehicle via cryptographic signatures

This forms the foundation of a **trustless vehicle marketplace**, where verified data is displayed on a website and payments are handled via on-chain tools like Porto.

---

## Project Overview

Vehicle Net is a decentralized vehicle condition reporting system with three main components:

- **Rust application** (`rust/`) - Vehicle condition reporting and cryptographic signing
- **Smart contracts** (`contracts/`) - Solidity contracts for vehicle registry and data marketplace using Foundry
- **Web application** (`web/`) - Next.js frontend with Web3 integration using Wagmi and Viem

## Common Commands

### Rust Application

```bash
# Build and run from root (recommended)
./scripts/run-rust.sh
./scripts/run-rust.sh --index 5  # Run with specific vehicle index

# From rust directory
cd rust && cargo run -- --index 0
```

### Smart Contracts (Foundry)

```bash
cd contracts
forge build      # Build contracts
forge test        # Run tests
forge fmt         # Format code
forge snapshot    # Gas snapshots
anvil            # Start local node
```

### Web Application

```bash
# Build and run from root (recommended)
./scripts/run-web.sh

# from web directory
cd web
npm install      # or bun install
npm run dev      # Start dev server on port 3001
npm run build    # Build for production
npm run lint     # Lint code
```

### Integration Testing

```bash
# Run comprehensive integration tests
./scripts/test-integration.sh

# Clean up test outputs
./scripts/test-integration.sh --cleanup
```

## Architecture

### Smart Contract System

- **VehicleRegistry.sol** - Core contract for vehicle registration with VIN mapping and metadata
- **DataMarketplace.sol** - Handles data access purchases and vehicle data listing
- **AccessControl.sol** - Manages permissions and authorization
- Uses OpenZeppelin for security patterns (Ownable, ReentrancyGuard)
- Built with Foundry framework

### Rust Component

- HD wallet derivation for different vehicles using master mnemonic
- Vehicle condition report generation (VIN, mileage, battery health, timestamp)
- Cryptographic signing of reports for blockchain compatibility
- Uses ethers-rs for Ethereum integration
- Contract bindings generated via build.rs

### Web Frontend

- Next.js 15 with TypeScript and TailwindCSS
- Web3 integration via Wagmi v2 and Viem v2
- React Query for state management
- shadcn/ui components with Radix UI
- Porto SDK integration for payment processing
- Runs on port 3001
- Dark/light theme support

### Key Integration Points

- Rust app generates signed vehicle reports
- Smart contracts store vehicle registry and handle data marketplace
- Web app provides UI for vehicle registration and data access purchases
- useVehicleContract hook abstracts contract interactions
- Contract bindings generated automatically in Rust build process

## Project Structure

```txt
vehicle-net/
├── rust/                 # Rust application
│   ├── src/
│   │   ├── main.rs       # CLI application entry point
│   │   ├── signer.rs     # HD wallet and signing logic
│   │   ├── contract_client.rs  # Contract interaction
│   │   └── contracts/    # Generated contract bindings
│   ├── Cargo.toml        # Dependencies
│   └── build.rs          # Contract binding generation
├── web/                  # Next.js application
│   ├── src/
│   │   ├── app/          # Next.js app router pages
│   │   ├── components/   # React components
│   │   ├── hooks/        # Custom React hooks
│   │   └── types/        # TypeScript type definitions
│   ├── package.json      # Dependencies
│   └── tailwind.config.js # Styling configuration
├── contracts/            # Solidity contracts
│   ├── src/              # Contract source files
│   ├── test/             # Contract tests
│   ├── script/           # Deployment scripts
│   └── foundry.toml      # Foundry configuration
└── scripts/              # Utility scripts
    ├── run-rust.sh       # Rust application runner
    ├── run-web.sh        # Web application runner
    └── test-integration.sh # Integration testing
```

## Development Notes

### Contract Development

- All contracts use Solidity ^0.8.19
- Gas optimization enabled with 200 runs
- Environment variables required for RPC endpoints and API keys (.env)
- OpenZeppelin contracts for security patterns
- Foundry for testing and deployment

### Web Development  

- TypeScript strict mode enabled
- Uses Wagmi hooks for blockchain interactions
- React Query for server state management
- TailwindCSS for styling with shadcn/ui components
- Next.js 15 with app router
- Porto SDK for payment integration

### Rust Development

- Uses ethers-rs for Ethereum integration
- HD wallet derivation with BIP-39/BIP-44
- Contract bindings generated at build time
- CLI interface with clap for argument parsing
- Async runtime with tokio

### Testing Strategy

- Contract tests with Foundry
- Integration tests via test-integration.sh script
- Manual testing for UI components
- Focus on compilation/build success rather than comprehensive test coverage

### Environment Setup

- Foundry for smart contract development
- Rust toolchain for backend
- Node.js ecosystem for frontend
- Environment variables for blockchain endpoints
- WSL recommended for Windows development

### Key Features

- Vehicle condition reporting with cryptographic signatures
- HD wallet derivation for vehicle identification
- Smart contract-based data marketplace
- Web3-enabled frontend with wallet connection
- Payment integration via Porto SDK
- Responsive design with modern UI components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/poke-key)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/poke-key)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
