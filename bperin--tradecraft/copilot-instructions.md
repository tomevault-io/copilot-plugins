## tradecraft

> Overview of Tradecraft application structure, state management, and cross-contract communication


# Tradecraft Application Structure

## Core Architecture

1. **TradecraftManager**: Central contract registry that holds references to all functional contracts
   - Serves as a contract registry and address lookup service
   - Handles access control via AccessManagerUpgradeable
   - Provides strongly-typed getters for contract instances

2. **BaseContract**: Foundation that all functional contracts inherit from
   - Defines common utilities, constants, and access control
   - Imports and re-exports Structs library for use by child contracts
   - Handles pagination for large data sets
   - Stores reference to TradecraftManager for cross-contract communication

3. **Functional Contracts**: Domain-specific contracts that implement business logic
   - Each manages its own state variables for its domain
   - Expose read and write methods through interfaces
   - Cross-contract calls are made through TradecraftManager

## State Management

1. **State Ownership**: Each contract owns and manages only its own domain data
   - UserContract manages users
   - VendorContract manages vendors and their attributes
   - ProductContract manages products and pricing
   - OrderContract manages orders and their lifecycle

2. **State Updates**: Contracts can trigger updates in other contracts through interfaces
   - Example: OrderContract.acceptOrder() causes stats updates in Product and Vendor entities
   - StatsContract.updateAllStats() pulls data from other contracts to calculate trending lists

3. **Data Access Patterns**:
   - Direct access: Contract reads from its own state variables
   - Cross-contract reads: Contract accesses other contract's data via interface calls
   - Cross-contract writes: Contract triggers state changes in other contracts via interface calls

## Cross-Contract Communication

1. **Interface Pattern**: For frequently used cross-contract calls
   - Interfaces like IVendorContract, IProductContract define available methods
   - Registered in TradecraftManager for type-safe access
   - Used for business logic that spans multiple contracts

2. **Direct Function Calls**: For less frequent operations
   - Some contracts call specific functions directly via TradecraftManager
   - Used for utility functions or one-off operations

3. **Event-Based Communication**:
   - Contracts emit events to notify external systems of state changes
   - Used for tracking state transitions and feeding external systems

## Example Cross-Contract Flows

1. **Order Creation and Acceptance**:
   - OrderContract creates new order
   - When order is accepted:
     - Funds are transferred
     - Order state is updated
     - Vendor and Product entities have stats updated

2. **Stats Calculation**:
   - StatsContract.updateAllStats() reads order, product, vendor data
   - Processes all stats in a single batch operation
   - Updates trending and top lists based on sales metrics

3. **Market Participation**:
   - VendorContract.joinMarket() checks eligibility with MarketContract
   - MarketContract verifies vendor meets requirements
   - VendorContract updates vendor's markets list 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bperin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
