## melodot

> _Comprehensive instructions for AI agents helping develop Melodot_

# Melodot Development Guide

_Comprehensive instructions for AI agents helping develop Melodot_

## Project Overview

**Melodot** is a decentralized music discovery and tipping platform that:
- Records audio using Web Audio API
- Identifies songs through audio recognition
- Enables direct artist tipping via smart contracts on Polkadot
- Allows artists to claim their identity and withdraw donations
- Integrates Web3Auth for seamless user onboarding

## Tech Stack

### Core Technologies
- **Smart Contracts**: Solidity 0.8.28, Hardhat, Polkadot Paseo Asset Hub
- **Frontend**: React 18, TypeScript, Vite
- **Web3**: Wagmi, Viem, Web3Auth Modal
- **Styling**: TailwindCSS 4, Radix UI, Lucide icons
- **Audio**: Web Audio API
- **API**: Vercel serverless functions
- **Package Manager**: Bun
- **Testing**: Playwright (e2e)

### Key Dependencies
```json
{
  "wagmi": "Web3 React hooks",
  "viem": "TypeScript Web3 client",
  "@web3auth/modal": "Social login for Web3",
  "@tanstack/react-query": "Data fetching",
  "lucide-react": "Icon library",
  "@radix-ui": "Headless UI components"
}
```

## Project Structure

```
shaz/
├── contracts/              # Smart contracts
│   ├── contracts/
│   │   └── Donate.sol     # Artist donation contract
│   ├── hardhat.config.ts  # Hardhat configuration
│   ├── test/              # Contract tests
│   └── ignition/          # Deployment configs
├── api/                   # Vercel serverless functions
│   ├── endpoints/
│   │   ├── discover.ts   # Music discovery endpoint
│   │   ├── artist.ts      # Single artist info
│   │   ├── artists.ts     # Multiple artists info
│   │   ├── track.ts       # Track information
│   │   └── verify.ts      # Artist verification
│   ├── api.ts             # Centralized API utilities
│   ├── mock-discovery-data.ts
│   └── utils.ts           # API utilities
├── src/                   # React application
│   ├── components/        # React components
│   │   ├── DiscoveryButton.tsx
│   │   ├── DiscoveryRecorder.tsx
│   │   ├── DiscoveryCard.tsx
│   │   ├── DonationForm.tsx
│   │   ├── OwnerWithdrawForm.tsx
│   │   ├── ArtistPayoutForm.tsx
│   │   ├── ClaimCard.tsx
│   │   ├── ClaimFlow.tsx
│   │   ├── UserMenu.tsx
│   │   ├── AppSidebar.tsx
│   │   ├── Header.tsx
│   │   ├── StatisticsDashboard.tsx
│   │   └── ui/            # Radix UI components
│   ├── pages/            # Page components
│   │   ├── Discover.tsx   # Main discovery page
│   │   ├── Claim.tsx      # Artist claim page
│   │   ├── Donations.tsx  # Donations history
│   │   └── Owner.tsx      # Owner dashboard
│   ├── hooks/            # Custom React hooks
│   ├── lib/              # Utilities
│   ├── generated.ts      # Auto-generated Wagmi types
│   └── wagmi-config.ts   # Wagmi configuration
├── tests/                # Playwright e2e tests
├── dist/                 # Production build
└── package.json
```

## Network Configuration

### Paseo Asset Hub Testnet

**Network Details:**
- Chain ID: `420420417` (0x190f1b41)
- RPC: `https://services.polkadothub-rpc.com/testnet`
- Explorer: `https://blockscout-testnet.polkadot.io`
- Faucet: `https://faucet.polkadot.io/`
- Currency: PAS

**Hardhat Configuration:**
```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "@parity/hardhat-polkadot";

const config: HardhatUserConfig = {
  solidity: "0.8.28",
  resolc: {
    compilerSource: "npm",
    settings: {},
  },
  networks: {
    passetHub: {
      polkavm: true, // REQUIRED for Polkadot
      url: "https://services.polkadothub-rpc.com/testnet",
      accounts: [process.env.PRIVATE_KEY],
    },
  },
};
```

## Development Workflow

### Essential Commands

```bash
# Development
bun install                  # Install dependencies
bun dev                      # Start dev server (port 5173)
bun run build               # Build for production
bun run reinstall           # Clean reinstall

# Contracts
cd contracts
bun run compile             # Compile contracts
bun run test                # Run contract tests
bun run deploy-contract     # Deploy to Paseo + generate types

# Testing
bun run test:e2e           # Run e2e tests (Playwright)
bun run test:api           # Test API functions
bun run test:contract      # Run contract tests
bun run test:all           # Run all tests

# Code Generation
bun run generate           # Generate Wagmi types from contract
```

### Private Key Setup

**IMPORTANT**: Never commit private keys
```bash
# In contracts/.env
PRIVATE_KEY=your_private_key_without_0x_prefix
```

### Get Testnet Tokens
1. Visit [Polkadot Faucet](https://faucet.polkadot.io/)
2. Enter your wallet address
3. Receive PAS tokens for testing

## Smart Contract Architecture

### Donate.sol

**Contract Features:**
- Artist donation tracking by ID (Spotify artist ID)
- Platform fee: 1% (100 basis points)
- Artist claiming system
- Withdrawal mechanisms for artists and platform

**Key Functions:**
```solidity
// Donate to artist
donateToArtist(string memory artistId) external payable

// Artist claims identity
claimArtist(string memory artistId) external

// Artist withdraws donations
withdrawDonates(string memory artistId, address recipient) external

// Owner withdraws platform fees
withdrawPlatformFees(address recipient) external onlyOwner

// View functions
getArtistInfo(string memory artistId) public view
getPlatformFeeBalance() public view
```

**Security Patterns:**
- Custom errors for gas efficiency
- Ownable pattern (based on thirdweb)
- Platform fee system with basis points
- Reentrancy protection via checks-effects-interactions

### Contract Limitations

- **Size Limit**: ~100KB bytecode
- **No OpenZeppelin**: Use minimal custom implementations (already done)
- **PolkaVM Required**: Must set `polkavm: true` in network config

## Frontend Architecture

### Web3 Integration

**Wagmi Configuration:**
```typescript
// Auto-generated from contract deployment
import { donateConfig } from "./generated";
import { useWriteContract } from "wagmi";

// Donate to artist
const { writeContract } = useWriteContract();
writeContract({
  ...donateConfig,
  functionName: "donateToArtist",
  args: [artistId],
  value: parseEther(amount),
});
```

### Web3Auth Setup

**Required Environment Variable:**
```bash
VITE_WEB3AUTH_CLIENT_ID=your_web3auth_client_id
```

Get your client ID from [Web3Auth Dashboard](https://dashboard.web3auth.io/)

**Configuration Structure:**
- `src/web3authContext.tsx`: Minimal config with client ID and network only
- `src/wagmi-config.ts`: Chain definitions (single source of truth)
- `src/main.tsx`: Providers setup (Web3AuthProvider wraps WagmiProvider)

### Audio Recording Flow

1. Request microphone permission
2. Record audio using Web Audio API
3. Send audio blob to `/api/discover`
4. Display song metadata and artist info
5. Enable donation to identified artist

**Key Components:**
- `DiscoveryButton.tsx` / `DiscoveryRecorder.tsx`: Audio recording interface
- `DiscoveryCard.tsx`: Display identified song with donation options
- `DonationForm.tsx`: Multi-artist donation form with amount controls
- `OwnerWithdrawForm.tsx`: Platform fee withdrawal (owner only)
- `ArtistPayoutForm.tsx`: Artist donation withdrawal
- `ClaimCard.tsx` / `ClaimFlow.tsx`: Artist identity claiming
- `UserMenu.tsx`: Wallet connection and user info
- `AppSidebar.tsx`: Navigation sidebar
- `Header.tsx`: App header with theme toggle
- `StatisticsDashboard.tsx`: Platform statistics display
- `DonationsArtistsTable.tsx`: Donations by artist view
- `DonationsDonorsTable.tsx`: Donations by donor view
- `DonationsTransactionsTable.tsx`: Transaction history

**Pages:**
- `Discover.tsx`: Main discovery and donation page
- `Claim.tsx`: Artist identity claiming page
- `Donations.tsx`: Donations history and analytics
- `Owner.tsx`: Owner dashboard for platform management

## API Endpoints

All API endpoints are Vercel serverless functions located in `api/endpoints/`.

### POST /api/discover

**Purpose**: Discover audio and identify music

**Request:**
```typescript
const formData = new FormData();
formData.append('file', audioBlob, 'recording.webm');

const response = await fetch('/api/discover', {
  method: 'POST',
  body: formData,
});
```

**Response:**
```typescript
{
  track: {
    title: string;
    subtitle: string; // Artist name
    images: { coverart: string };
    hub: { 
      providers: [{ type: "SPOTIFY", actions: [{ uri: string }] }]
    };
    artists: [{ adamid: string }]; // Spotify artist ID
  },
  artistInfo: {
    type: "Artist";
    country: string;
    socialLinks: {
      twitter?: string;
      instagram?: string;
      spotify?: string;
      facebook?: string;
      youtube?: string;
      tiktok?: string;
      soundcloud?: string;
      bandcamp?: string;
      website?: string;
    }
  }
}
```

**Current Implementation**: Uses mock data when `USE_MOCK_DATA=true`, otherwise integrates with Shazam API via RapidAPI and Spotify API.

### GET /api/artist

**Purpose**: Fetch single artist information from Spotify

**Request:**
```typescript
const response = await fetch('/api/artist?artistUrl=ARTIST_ID_OR_URL');
```

**Response:**
```typescript
{
  artist: SpotifyArtistResponse; // Full Spotify artist object
}
```

**Requires**: `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`

### GET /api/artists

**Purpose**: Fetch multiple artists information (up to 50)

**Request:**
```typescript
const response = await fetch('/api/artists?ids=id1,id2,id3');
```

**Response:**
```typescript
{
  artists: ArtistsMap; // Map of artist ID to SpotifyArtistResponse
}
```

**Requires**: `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`

### GET /api/track

**Purpose**: Fetch track information from Spotify

**Request:**
```typescript
const response = await fetch('/api/track?uri=spotify:track:TRACK_ID');
```

**Response:**
```typescript
{
  track: SpotifyTrackResponse; // Full Spotify track object
}
```

**Requires**: `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`

### GET /api/verify

**Purpose**: Verify artist identity by checking biography for verification code

**Request:**
```typescript
const response = await fetch('/api/verify?artistId=ARTIST_ID');
```

**Response:**
```typescript
{
  verified: boolean;
}
```

**Requires**: `RAPIDAPI_KEY`, `VITE_VERIFICATION_CODE`

## Environment Variables

### Required

```bash
# .env (root)
VITE_WEB3AUTH_CLIENT_ID=       # Web3Auth dashboard
PRIVATE_KEY=                    # Deployment wallet (no 0x prefix)
```

### Optional (for music APIs)

```bash
VITE_RAPIDAPI_KEY=             # RapidAPI key for Shazam/Spotify scraper
SPOTIFY_CLIENT_ID=             # Spotify API client ID
SPOTIFY_CLIENT_SECRET=         # Spotify API client secret
VITE_VERIFICATION_CODE=        # Code artists must include in bio to verify
```

## Deployment

### Contract Deployment

```bash
cd contracts
bun run deploy-contract        # Deploys and generates types
```

**Deployment Checklist:**
- [ ] Private key set in `.env`
- [ ] Sufficient PAS tokens in wallet
- [ ] Contract compiles without errors
- [ ] Contract size under 100KB
- [ ] `polkavm: true` in hardhat.config.ts

**Reset Deployment State:**
```bash
cd contracts
rm -rf ignition/deployments/
bun run deploy-contract
```

### Frontend Deployment (Vercel)

```bash
# Automatic deployment
git push origin main

# Or manual
vercel deploy --prod
```

**Vercel Configuration:**
- Build command: `bun run build`
- Output directory: `dist`
- Install command: `bun install`
- Framework preset: Vite
- Environment variables: Set in Vercel dashboard

## Testing

### E2E Tests (Playwright)
```bash
bun run test:e2e          # Run e2e tests
```

### Contract Tests
```bash
cd contracts
bun run test              # Hardhat tests
```

## Common Issues & Solutions

### "CodeRejected" Error
**Cause**: Missing PolkaVM configuration
**Solution**: Ensure `polkavm: true` in `hardhat.config.ts` networks config

### "initcode is too big"
**Cause**: Contract exceeds 100KB limit
**Solution**: Already optimized with minimal implementations instead of OpenZeppelin

### Contract Deployment Fails
**Checklist**:
- Private key format correct (no `0x` prefix)
- Sufficient PAS balance
- Network connectivity to RPC
- Clean deployment: `rm -rf ignition/deployments/`

### Web3Auth Not Working
**Checklist**:
- `VITE_WEB3AUTH_CLIENT_ID` set correctly
- Network matches (Paseo Asset Hub)
- Browser has Web3 wallet extension

### Type Errors After Contract Changes
**Solution**:
```bash
bun run deploy-contract  # Regenerates types
# or
bun run generate
```

## Security Best Practices

### Environment Variables & Secrets
- **Never commit .env files** - Add to `.gitignore`
- **Rotate exposed credentials immediately** - If secrets are committed, they're compromised
- **Use separate keys for dev/prod** - Don't reuse production credentials
- **Prefix client-side vars with VITE_** - Only VITE_ vars are bundled into frontend

### Input Validation
- **Validate Spotify IDs**: Must be 22 alphanumeric characters (`/^[a-zA-Z0-9]{22}$/`)
- **Check HTTP responses**: Always verify `response.ok` before parsing JSON
- **Use type guards**: Validate contract return types with proper type guards, not unsafe casts

### Frontend Security
- **No demo bypasses in production** - Remove test buttons that skip verification
- **Verify artist identity** - Always require verification before claiming
- **Use type-safe patterns** - Create type guards for contract data instead of `as` casts

### API Security
- **Add CORS headers** - Restrict API access to your domain
- **Implement rate limiting** - Prevent API abuse and quota exhaustion
- **Validate all inputs** - Check format, length, and type of all parameters
- **Don't expose internal errors** - Log server-side, return generic messages to clients

### Smart Contract Security
- **Follow checks-effects-interactions** - Update state before external calls
- **Consider reentrancy guards** - Add mutex locks for state-modifying functions
- **Validate string lengths** - Prevent storage bloat with maximum length checks

## Code Style & Conventions

### Writing Style (Documentation)

**Core Principles:**
- Use active voice: "Deploy the contract" not "The contract should be deployed"
- Lead with results: State what happens, then explain how
- Be brief: Cut qualifiers like "very", "quite", "rather"
- Use simple words: "use" not "utilize", "do" not "implement"
- Provide exact commands, not abstractions

**Technical Documentation:**
- Start with what it does, not how it works
- Use concrete examples over abstract descriptions
- Write instructions as commands
- Assume intelligence but not knowledge

### Contract Development

- **Solidity Version**: 0.8.28
- **Gas Optimization**: Use custom errors instead of require strings
- **Security**: Follow checks-effects-interactions pattern
- **Naming**: Clear, descriptive function and variable names

### Frontend Development

- **TypeScript**: Strict mode enabled
- **Components**: Functional components with hooks
- **Styling**: TailwindCSS utility classes
- **State**: React Query for server state, useState for local state

### Performance Patterns

- **Use useMemo** - Memoize expensive computations (sorting, filtering, mapping)
- **Use useCallback** - Memoize functions passed to child components
- **Memoize arrays** - Arrays passed to hooks should be memoized to prevent re-fetching
- **Avoid inline objects** - Objects/arrays created in render cause child re-renders
- **writeContract is synchronous** - Don't wrap in try-catch, use error state instead

## Troubleshooting Checklist

**Before asking for help:**

- [ ] Dependencies installed: `bun install`
- [ ] Environment variables set in `.env`
- [ ] Private key set (no `0x` prefix)
- [ ] Sufficient PAS testnet tokens
- [ ] Contract compiles: `cd contracts && bun run compile`
- [ ] Network RPC accessible
- [ ] Web3Auth client ID configured
- [ ] Generated types up to date: `bun run generate`

## Quick Reference

### Key Files
- `contracts/contracts/Donate.sol` - Main donation contract
- `contracts/hardhat.config.ts` - Network configuration
- `src/main.tsx` - App entry point and providers setup
- `src/wagmi-config.ts` - Chain definitions (single source of truth)
- `src/web3authContext.tsx` - Web3Auth client configuration
- `src/components/DiscoveryButton.tsx` - Recording interface
- `src/components/DiscoveryCard.tsx` - Song display and donation
- `src/components/DonationForm.tsx` - Multi-artist donation form
- `src/pages/Discover.tsx` - Main discovery page
- `src/pages/Claim.tsx` - Artist claim page
- `src/pages/Donations.tsx` - Donations history
- `src/pages/Owner.tsx` - Owner dashboard
- `api/endpoints/discover.ts` - Music discovery API
- `api/api.ts` - Centralized API utilities
- `src/generated.ts` - Auto-generated Wagmi contract types
- `package.json` - Scripts and dependencies

### Key URLs
- Local dev: `http://localhost:5173`
- Block explorer: `https://blockscout-testnet.polkadot.io`
- Faucet: `https://faucet.polkadot.io/`
- Web3Auth dashboard: `https://dashboard.web3auth.io`

### Package Manager: Bun
This project uses Bun, not npm or yarn. Always use:
- `bun install` (not npm install)
- `bun run <script>` (not npm run)
- `bun add <package>` (not npm install)

## Resources

- [Polkadot Smart Contracts](https://docs.polkadot.com/develop/smart-contracts/)
- [Hardhat Polkadot Plugin](https://github.com/paritytech/hardhat-polkadot)
- [Wagmi Documentation](https://wagmi.sh/)
- [Web3Auth Documentation](https://web3auth.io/docs)
- [Viem Documentation](https://viem.sh/)
- [Shazam API (RapidAPI)](https://rapidapi.com/apidojo/api/shazam)

---

**Remember**: This is a Polkadot-based project using EVM compatibility. Always ensure `polkavm: true` in network config and use PAS testnet tokens.

---
> Source: [janmichek/melodot](https://github.com/janmichek/melodot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
