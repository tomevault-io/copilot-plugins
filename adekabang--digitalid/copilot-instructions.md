## digitalid

> This document provides guidance for Claude when working on the Blockchain Identity System.

# Blockchain Identity System Development Guide

This document provides guidance for Claude when working on the Blockchain Identity System.

## Project Structure

The Blockchain Identity System consists of three main components:

1. **Smart Contracts (`contracts/`)** - Solidity smart contracts for identity NFTs and related systems
2. **Backend API (`backend/`)** - Node.js/Express API for interacting with the smart contracts
3. **Oracle Service (`oracle/`)** - Off-chain service for verification and KYC processes

## Key Workflows

### Verification Flow

The identity verification process is a multi-step flow involving:

1. Creating a user identity in the DigitalIdentityNFT contract
2. Requesting verification through the Oracle service's mock endpoint
3. The Oracle processes the verification and updates to BASIC_VERIFIED (level 1)
4. A second, different verifier upgrades to KYC_VERIFIED (level 2) through the /second-approval endpoint

#### Important Security Feature

The system requires **at least 2 different verifier addresses** to approve before upgrading an identity to KYC_VERIFIED (level 2) or higher. This prevents a single compromised verifier from granting high verification levels.

### Running Tests

When validating changes to the verification flow, run:

```bash
./scripts/test-api-flow.sh
```

This script tests the complete flow through the API endpoints.

## Development Patterns

### Error Handling

All API endpoints should handle and return standardized error responses:

```javascript
try {
  // Code here
} catch (error) {
  logger.error('Error description:', error);
  return res.status(500).json({
    success: false,
    error: error.message
  });
}
```

### BigInt Serialization

When working with blockchain data, always handle BigInt serialization:

```javascript
// Helper function to safely stringify BigInt values
const safeStringify = (obj) => {
  return JSON.stringify(obj, (key, value) => 
    typeof value === 'bigint' ? value.toString() : value
  );
};

// Parse and re-stringify to handle any nested BigInt values
return res.json(JSON.parse(safeStringify(responseObj)));
```

## Key Files

- **VERIFICATION_FLOW.md** - Detailed explanation of the verification process
- **oracle/src/controllers/verifications.controller.js** - Handles verification requests and second verifier approval
- **scripts/test-api-flow.sh** - End-to-end test script for the verification flow
- **contracts/DigitalIdentityNFT.sol** - Core identity NFT contract with multi-verifier security
- **contracts/VerificationRegistry.sol** - Registry of verification claims

## Linting and Formatting

For backend JavaScript files:
```bash
cd backend
npm run lint
```

For Oracle service files:
```bash
cd oracle
npm run lint
```

For Solidity contracts:
```bash
npm run lint:sol
```

## Documentation Updates

When making changes to the verification flow or Oracle service:

1. Update VERIFICATION_FLOW.md
2. Update the Oracle's swagger.json
3. Update the backend's swagger.json to reference Oracle endpoints
4. Update README.md with any new setup or usage instructions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Adekabang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Adekabang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
