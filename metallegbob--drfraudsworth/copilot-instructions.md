## drfraudsworth

> This project was built collaboratively with [Claude](https://claude.ai) (Anthropic's AI assistant), primarily using Claude Code. The entire 104-phase build journey is documented in `.planning/` -- from initial documentation audit through mainnet deployment.

# AI-Assisted Development with Claude

This project was built collaboratively with [Claude](https://claude.ai) (Anthropic's AI assistant), primarily using Claude Code. The entire 104-phase build journey is documented in `.planning/` -- from initial documentation audit through mainnet deployment.

## Development Philosophy

- Built with Anchor/Rust, professionally done. Always considering security and attack vectors because crypto is dangerous at the best of times
- Break tasks down into the smallest steps and explain the why of each step
- If we can fork something that is secure, well-reviewed/starred on GitHub and open source, we should always choose that instead of building it ourselves
- No cutting corners -- dedicated to quality at every step
- When docs clash or are incompatible, stop and discuss rather than making assumptions

## Environment Setup

Before running Rust/Cargo/Anchor/Solana commands, source the environment:
```bash
source "$HOME/.cargo/env"
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"
```

### Required Tools
- **Rust/Cargo**: `~/.cargo/bin/` (loaded via `source "$HOME/.cargo/env"`)
- **Solana CLI**: `~/.local/share/solana/install/active_release/bin/`
- **Anchor CLI**: Installed via AVM, lives in `~/.cargo/bin/`
- **Node.js**: v18+ (for TypeScript scripts and Next.js frontend)

### Wallet Configuration
- Configure your wallet path in `Anchor.toml` and `.env` files
- See `.env.example` for all required environment variables

## Solana Documentation Tip

Solana documentation is LLM-ready -- just add `.md` to any URL. For example:
- `solana.com/docs/core` -> `solana.com/docs/core.md`

This gives you optimized documentation for AI-assisted development.

## Build and Test

```bash
# Build all programs
anchor build

# Build with devnet feature flags
anchor build && anchor build -p epoch_program -- --features devnet

# Run tests
anchor test
```

## Project Structure

See the README for full architecture overview. Key directories:
- `programs/` -- 7 Anchor programs (AMM, Transfer Hook, Tax, Epoch, Staking, Conversion Vault, Bonding Curve)
- `app/` -- Next.js frontend
- `scripts/` -- Deploy, crank, graduation, and test scripts
- `shared/` -- TypeScript shared constants
- `.planning/` -- 104 phases of build journey documentation
- `.audit/`, `.bulwark/`, `.bok/` -- Security audit reports and formal verification

---
> Source: [MetalLegBob/drfraudsworth](https://github.com/MetalLegBob/drfraudsworth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
