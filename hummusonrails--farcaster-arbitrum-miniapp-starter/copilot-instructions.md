## farcaster-arbitrum-miniapp-starter

> > This file is automatically read by Claude Code. It teaches AI agents how to work with this codebase safely.

# Farcaster + Arbitrum Mini-App Starter

> This file is automatically read by Claude Code. It teaches AI agents how to work with this codebase safely.

## What This Is

A starter template for building Farcaster mini-apps with Arbitrum smart contracts. It has three parts:
- **Smart Contract** (Rust) - An NFT contract that runs on Arbitrum
- **Frontend** (React) - A web app that connects wallets and mints NFTs
- **Farcaster Integration** - Configuration that makes this work inside Farcaster

## Critical Rules (Always Follow)

### Rule 1: Never modify the contract without updating the ABI
If you change `contracts/template/src/lib.rs`, you MUST also update `frontend/src/abi/SampleNFT.json`. The frontend uses this file to talk to the contract.

**After contract changes:**
```bash
cd contracts/template && cargo stylus export-abi --json
```
Then copy the output to `frontend/src/abi/SampleNFT.json`

### Rule 2: Contract address must match deployment
The constant `CONTRACT_ADDRESS` in `frontend/src/App.tsx` (line 13) must match the actually deployed contract. If you deploy a new contract, update this value.

### Rule 3: Farcaster URLs must be consistent
These two files must have matching URLs:
- `frontend/public/.well-known/farcaster.json` (homeUrl, iconUrl, splashImageUrl)
- `frontend/index.html` (fc:frame meta tag)

### Rule 4: Propose changes before writing code
Before making any code changes, explain what you plan to modify and why. List the files that will be affected.

## File Map (What Lives Where)

| What You Want to Change | File to Edit |
|------------------------|--------------|
| NFT logic (minting, metadata) | `contracts/template/src/lib.rs` |
| NFT name, symbol, image | `contracts/template/src/lib.rs` (name, symbol, token_uri functions) |
| Contract ABI (after contract changes) | `frontend/src/abi/SampleNFT.json` |
| Contract address | `frontend/src/App.tsx` line 13 |
| Frontend UI | `frontend/src/App.tsx` |
| Wallet/chain configuration | `frontend/src/wagmi.ts` and `frontend/src/viemChains.ts` |
| Farcaster app metadata | `frontend/public/.well-known/farcaster.json` |
| Farcaster frame embed | `frontend/index.html` (fc:frame meta tag) |
| App styling | `frontend/src/App.tsx` (Tailwind classes) |

## Common Tasks

### "I want to test my app in Farcaster locally"

Testing in Farcaster requires exposing your local servers via ngrok:

1. **Start the local devnode** (if using local blockchain):
   ```bash
   pnpm --filter contract-template nitro-node
   ```

2. **Start the frontend**:
   ```bash
   pnpm dev
   ```

3. **Start ngrok tunnels** (in a new terminal):
   ```bash
   ngrok start --all --config ngrok.yml
   ```

4. **Get your ngrok URLs** from the ngrok terminal output:
   - Frontend URL: `https://xxxx-xxx.ngrok.app` (port 5173)
   - RPC URL: `https://yyyy-yyy.ngrok.app` (port 8547)

5. **Update these files with ngrok URLs**:
   - `frontend/src/viemChains.ts` - Replace `http://localhost:8547` with ngrok RPC URL
   - `frontend/public/.well-known/farcaster.json` - Set `homeUrl` to ngrok frontend URL
   - `frontend/index.html` - Update fc:frame meta tag URLs

6. **Add ngrok RPC to your wallet** (MetaMask):
   - Network Name: Nitro Dev
   - RPC URL: Your ngrok RPC URL (https://yyyy-yyy.ngrok.app)
   - Chain ID: 412346

7. **Test in Farcaster Mini-App Previewer**:
   - Go to: https://farcaster.xyz/~/developers/mini-apps/preview
   - Enter your ngrok frontend URL

**Important:** Revert the URL changes before committing! Use `git checkout` to restore original files.

### "I want to change what the NFT is about"
1. Edit `contracts/template/src/lib.rs`:
   - Change `name()` function return value
   - Change `symbol()` function return value
   - Change `token_uri()` to return your metadata
2. Export new ABI: `cd contracts/template && cargo stylus export-abi --json`
3. Copy ABI to `frontend/src/abi/SampleNFT.json`
4. Update image in `frontend/public/` if needed

### "I want to change the app appearance"
Edit `frontend/src/App.tsx`. This file uses Tailwind CSS classes for styling.

### "I want to add a new contract function"
1. Add the function to `contracts/template/src/lib.rs`
2. Export new ABI (see above)
3. Update `frontend/src/abi/SampleNFT.json`
4. Add frontend call in `frontend/src/App.tsx` using `writeContract` or `publicClient.readContract`

### "I want to deploy to production"
1. Deploy contract: `cd contracts/template && cargo stylus deploy` (needs RPC URL and private key)
2. Copy the deployed address to `frontend/src/App.tsx` CONTRACT_ADDRESS
3. Update URLs in `frontend/public/.well-known/farcaster.json`
4. Update URLs in `frontend/index.html` fc:frame meta tag
5. Build frontend: `pnpm build`

## Project Structure

```
/
├── contracts/template/
│   ├── src/lib.rs          ← Smart contract code (Rust)
│   ├── Cargo.toml          ← Contract dependencies
│   └── package.json        ← Contract build scripts
├── frontend/
│   ├── src/
│   │   ├── App.tsx         ← Main React component + CONTRACT_ADDRESS
│   │   ├── abi/SampleNFT.json  ← Contract ABI (must match lib.rs)
│   │   ├── wagmi.ts        ← Wallet configuration
│   │   └── viemChains.ts   ← Chain definitions
│   ├── public/
│   │   ├── .well-known/farcaster.json  ← Farcaster manifest
│   │   └── nft.png         ← NFT image
│   └── index.html          ← HTML template + fc:frame meta
└── CLAUDE.md               ← This file (AI agent instructions)
```

## Detailed Documentation

For deeper understanding, see the `.ai/` directory:
- `.ai/ARCHITECTURE.md` - Complete system architecture
- `.ai/TASKS.md` - Step-by-step modification guides
- `.ai/SYNC_REQUIREMENTS.md` - What must stay synchronized
- `.ai/PROMPTS.md` - Example prompts you can use
- `.ai/TROUBLESHOOTING.md` - Common problems and fixes

## Quick Validation

After making changes, verify:
1. **Contract compiles:** `cd contracts/template && cargo build --release`
2. **Frontend builds:** `pnpm build`
3. **ABI matches contract:** Function names in `SampleNFT.json` match `lib.rs`
4. **URLs are set:** No "YOUR_APP_URL" placeholders remain

---
> Source: [hummusonrails/farcaster-arbitrum-miniapp-starter](https://github.com/hummusonrails/farcaster-arbitrum-miniapp-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
