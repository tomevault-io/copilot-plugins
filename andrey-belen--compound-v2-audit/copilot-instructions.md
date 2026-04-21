## compound-v2-audit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive security audit environment for Compound V2 protocol analysis, designed for educational and defensive security purposes. The project includes:

- **Security Testing Framework**: Complete test suite for identifying DeFi vulnerabilities
- **Attack Vector Simulations**: Flash loan, reentrancy, oracle manipulation, and liquidation attacks
- **Historical Bug Analysis**: Recreation of known Compound V2 vulnerabilities for learning
- **Audit Infrastructure**: Logging, monitoring, and reporting tools for security analysis

## Architecture

### Core Components

- `src/AuditBase.sol` - Base contract with common utilities (logging, time manipulation, token dealing, address constants)
- `src/CompoundHelpers.sol` - Helper functions for Compound V2 interactions (supply, borrow, liquidate) with interfaces
- `src/OracleManipulator.sol` - Price oracle manipulation utilities for testing vulnerabilities with DEX interactions
- `src/Counter.sol` - Simple counter contract (template/example)

### Contract Inheritance Pattern
All test contracts follow this pattern:
```solidity
contract MyAuditTest is Test, AuditBase, CompoundHelpers, OracleManipulator {
    // Inherits Foundry Test + audit utilities + Compound helpers + oracle manipulation
}
```

### Test Structure

```
test/
├── exploits/               # Attack vector test suites
│   ├── FlashLoanAttacks.t.sol
│   ├── ReentrancyTests.t.sol
│   └── CompoundHistoricalBugs.t.sol
├── unit/                   # Unit tests
├── integration/            # Integration tests
└── fixtures/               # Test data and configuration
```

### Key Directories

- `audit-reports/` - Generated audit reports and findings
- `scripts/` - Deployment and utility scripts (setup-env.sh, DeployAuditEnvironment.s.sol)
- `lib/` - External dependencies (OpenZeppelin, Compound protocol, forge-std)
- `out/` - Compiled contract artifacts and build information
- `cache/` - Foundry compilation cache

### Pre-configured Constants
The `AuditBase.sol` provides pre-configured mainnet addresses:

**Tokens**: USDC, WETH, DAI, USDT, COMP_TOKEN  
**Compound V2**: COMPTROLLER, CUSDC, CDAI, CETH, CUSDT, PRICE_ORACLE  
**Test Accounts**: ADMIN, USER1, USER2, LIQUIDATOR, ATTACKER, WHALE

## Development Commands

### Environment Setup
```bash
# Initial setup (run once) - sets up .env, git hooks, dependencies
./scripts/setup-env.sh

# Manual setup alternative
cp .env.example .env
# Edit .env with your RPC URLs and API keys
source .env

# Install dependencies and build
forge install
forge build
```

### Core Testing Commands
```bash
# Run all tests with mainnet fork
forge test --fork-url $ETH_RPC_URL

# Run specific vulnerability test contracts
forge test --match-contract FlashLoanAttacksTest --fork-url $ETH_RPC_URL -vv
forge test --match-contract ReentrancyTestsTest --fork-url $ETH_RPC_URL -vv
forge test --match-contract CompoundHistoricalBugsTest --fork-url $ETH_RPC_URL -vv

# Run individual test files
forge test --match-path test/exploits/FlashLoanAttacks.t.sol --fork-url $ETH_RPC_URL -vv
forge test --match-path test/exploits/ReentrancyTests.t.sol --fork-url $ETH_RPC_URL -vv

# Run specific test functions
forge test --match-test test_FlashLoanPriceManipulation --fork-url $ETH_RPC_URL -vvv
forge test --match-test test_SupplyReentrancy --fork-url $ETH_RPC_URL -vvv

# Quick security test suite (if script exists)
./scripts/test-quick.sh

# Full test suite with coverage (if script exists)
./scripts/test-full.sh

# Gas usage analysis (if script exists)
./scripts/analyze-gas.sh
```

### Build and Deploy
```bash
# Build contracts
forge build

# Deploy audit environment
forge script scripts/DeployAuditEnvironment.s.sol --rpc-url $ETH_RPC_URL

# Run with different profiles
forge test --profile ci          # CI profile (more fuzz runs)
forge test --profile intense     # Intensive testing
```

### Debugging and Analysis
```bash
# Run with maximum verbosity
forge test --fork-url $ETH_RPC_URL -vvvv

# Gas reporting
forge test --gas-report --fork-url $ETH_RPC_URL

# Coverage analysis
forge coverage --fork-url $ETH_RPC_URL --report lcov
```

## Critical Configuration

### Environment Variables Required
- `ETH_RPC_URL` - Mainnet RPC endpoint (Alchemy/Infura)
- `PRIVATE_KEY` - Private key for deployments (TEST KEY ONLY! Default: Anvil test key)
- `ETHERSCAN_API_KEY` - For contract verification (optional)
- `POLYGON_RPC_URL` - Polygon RPC endpoint (optional, for multi-chain testing)
- `ARBITRUM_RPC_URL` - Arbitrum RPC endpoint (optional, for multi-chain testing)

### Fork Configuration
- **Fork Block**: 19,000,000 (configured in foundry.toml)
- **Reason**: Block with stable Compound V2 deployment for reliable testing
- **Gas Limit**: 30M gas for complex attack simulations

### Security Settings
- All tests include safety checks to prevent mainnet operations
- Private key validation in git hooks
- Automated security scanning in pre-commit hooks

## Test Categories and Purpose

### 1. Flash Loan Attacks (`FlashLoanAttacks.t.sol`)
**Educational Purpose**: Understanding how flash loans can be weaponized
- Price manipulation + liquidation attacks  
- Arbitrage exploitation
- Liquidation threshold manipulation
- Cross-protocol flash loan strategies

### 2. Reentrancy Tests (`ReentrancyTests.t.sol`)  
**Educational Purpose**: Identifying reentrancy vulnerabilities in DeFi
- Supply/redeem reentrancy via malicious tokens
- Borrow/repay callback exploitation
- Liquidation callback attacks
- Cross-function reentrancy patterns

### 3. Historical Bugs (`CompoundHistoricalBugs.t.sol`)
**Educational Purpose**: Learning from real-world DeFi incidents
- The $80M Compound liquidation bug (Proposal 062)
- Interest rate model manipulation
- Oracle staleness exploitation
- Liquidation incentive calculation errors

### 4. Oracle Manipulation (`OracleManipulator.sol`)
**Educational Purpose**: Understanding price feed vulnerabilities
- Flash loan + DEX price manipulation
- Sandwich attacks around large trades
- Oracle delay exploitation
- Cross-market arbitrage attacks

## Security Considerations

### This is a DEFENSIVE Security Project
- **Purpose**: Educational and defensive security research
- **Ethics**: Only for identifying vulnerabilities to build better defenses
- **Usage**: Never use attack patterns for malicious purposes
- **Scope**: Testnet and mainnet fork testing only

### Key Security Features
1. **Mainnet Protection**: Tests include chain ID validation
2. **Private Key Safety**: Git hooks prevent committing real keys
3. **RPC Rate Limiting**: Configured to avoid API overuse
4. **Gas Monitoring**: Alerts for unusual gas consumption patterns

## Common Patterns

### Test Structure Pattern
```solidity
contract MyAuditTest is Test, AuditBase, CompoundHelpers {
    function setUp() public {
        setUpAudit(); // Initialize fork and test accounts
        // Custom setup...
    }
    
    function test_MyVulnerability() public withLogging("Test", "Action") {
        // Test implementation with automatic logging
    }
}
```

### Attack Simulation Pattern
```solidity
// 1. Setup victim position
setupVictimPosition();

// 2. Record initial state  
uint256 initialBalance = token.balanceOf(attacker);

// 3. Execute attack
vm.startPrank(ATTACKER);
// ... attack logic ...
vm.stopPrank();

// 4. Analyze results and log vulnerability if found
if (profit > 0) {
    logVulnerability("Attack Name", target, severity, "Description");
}
```

### Helper Usage Pattern
```solidity
// Compound interactions
supplyToCompound(user, ICToken(CUSDC), amount);
borrowFromCompound(user, ICToken(CDAI), borrowAmount);
liquidatePosition(liquidator, borrower, cTokenBorrowed, repayAmount, cTokenCollateral);

// Price manipulation
simulateFlashLoanPriceManipulation(targetToken, flashAmount, isPump);
```

## Troubleshooting

### Common Issues

1. **RPC Rate Limiting**
   - Use `--slow` flag for tests
   - Configure delay between calls in foundry.toml

2. **Fork Block Issues**
   - Ensure fork block is before major protocol changes
   - Update `fork_block_number` in foundry.toml if needed

3. **Gas Estimation Failures**
   - Increase `gas_limit` in foundry.toml
   - Use `vm.fee()` and `vm.coinbase()` for gas price control

4. **Memory Issues with Large Tests**
   - Run test categories separately
   - Use `--no-match-test` to exclude heavy tests during development

5. **Contract Does Not Exist Errors**
   - If you see "Contract 0x... does not exist" during setUp, the issue may be in token dealing
   - Some tests work (e.g., ReentrancyTests) while others fail (e.g., FlashLoanAttacks)
   - This typically occurs when `dealToken()` or `deal()` tries to access a contract during fork setup
   - Try running individual test contracts to isolate the issue

### Debug Commands
```bash
# Debug specific test
forge test --match-test test_FlashLoanAttack -vvvv --fork-url $ETH_RPC_URL

# Test individual contracts to isolate issues
forge test --match-contract ReentrancyTestsTest --fork-url $ETH_RPC_URL -v
forge test --match-contract FlashLoanAttacksTest --fork-url $ETH_RPC_URL -v

# Check contract sizes
forge build --sizes

# Verify RPC connection
cast block latest --rpc-url $ETH_RPC_URL
```

### Known Test Status
- ✅ **ReentrancyTests**: Working properly with fork setup
- ❌ **FlashLoanAttacks**: Currently has setUp() issues with contract existence
- ❓ **CompoundHistoricalBugs**: Status unknown, needs testing

To run working tests only:
```bash
forge test --match-contract ReentrancyTestsTest --fork-url $ETH_RPC_URL
```

## Reporting and Documentation

### Generated Reports
- Gas usage analysis in `./gas-analysis.txt`
- Coverage reports in `./coverage/`
- Audit findings logged via events and console output

### Report Templates
- Audit report template in `audit-reports/templates/`
- Customize for specific audit requirements

## Educational Value

This project serves as a comprehensive learning resource for:
- **Security Auditors**: Understanding real-world attack patterns
- **DeFi Developers**: Identifying common vulnerability classes
- **Researchers**: Analyzing historical DeFi incidents
- **Students**: Learning about blockchain security through hands-on testing

The goal is to build better, more secure DeFi protocols by understanding how attacks work and developing robust defenses.

---

**Disclaimer**: This audit framework is for educational and defensive purposes only. Do not use attack patterns for malicious activities. Always conduct security research ethically and responsibly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrey-belen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
