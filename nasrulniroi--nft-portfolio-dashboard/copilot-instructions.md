## nft-portfolio-dashboard

> This document provides a comprehensive guide for AI agents (Claude Code, Hermes Agent, etc.) working on this project.

# Agent Development Guide

## Project: DeFi Lending Rate Aggregator

This document provides a comprehensive guide for AI agents (Claude Code, Hermes Agent, etc.) working on this project.

## Repository Structure

```
defi-lending-aggregator/
├── config/              # YAML configuration files
├── src/
│   ├── engine/          # Python: rate fetching, risk models, optimization
│   │   ├── models/      # Data models (protocol, lending_rate, opportunity)
│   │   ├── data/        # Protocol data fetchers (aave, compound, morpho, defillama)
│   │   └── utils/       # Config loader, logging
│   ├── scanner/         # Go: on-chain data collection
│   │   ├── cmd/         # CLI commands (scan, serve)
│   │   └── internal/
│   │       ├── chain/   # Per-chain implementations
│   │       ├── protocol/# Protocol rate types and aggregation
│   │       └── rpc/     # JSON-RPC client
│   ├── web/             # Next.js 14 App Router
│   │   ├── app/         # Pages and API routes
│   │   ├── components/  # React components
│   │   ├── lib/         # Utilities and API client
│   │   └── styles/      # Global CSS
│   ├── db/              # SQL schema and migrations
│   └── scripts/         # Shell scripts for ops
├── tests/               # Test suites (mirror src/ structure)
├── docs/                # Documentation and screenshots
└── docker-compose.yml   # Container orchestration
```

## Agent Workflow

### When Adding a New Protocol

1. Create `src/engine/data/<protocol>.py` — implement `ProtocolFetcher` interface
2. Add protocol config to `config/protocols.yaml`
3. Add chain config to `config/chains.yaml` if new chain
4. Create `src/scanner/internal/chain/<chain>.go` if new chain
5. Update `src/engine/protocol_registry.py` to register the new protocol
6. Add database migration in `src/db/migrations/`
7. Update frontend protocol list in `src/web/lib/constants.ts`
8. Add tests in `tests/engine/` and `tests/scanner/`
9. Update `docs/API.md` with new endpoints/fields
10. Update `README.md` features list

### When Adding a New Chain

1. Create `src/scanner/internal/chain/<chain>.go` implementing `ChainScanner`
2. Add chain config to `config/chains.yaml`
3. Add RPC URL env var to `.env.example`
4. Update `docker-compose.yml` with chain-specific config
5. Update frontend chain selector in `src/web/lib/constants.ts`
6. Add chain to `src/web/components/ui/badge.tsx` color mapping
7. Write tests for the new chain scanner

### When Modifying the Rate Model

1. Update `src/engine/models/lending_rate.py`
2. Update `src/scanner/internal/protocol/types.go`
3. Run database migration if schema changes
4. Update `docs/RATE_MODEL.md`
5. Update API response examples in `docs/API.md`
6. Run full test suite: `make test`

## Coding Conventions

### Python
```python
# Every public function needs type hints and a docstring
async def fetch_supply_rates(
    protocol: str,
    chain: str,
    assets: list[str] | None = None,
) -> dict[str, LendingRate]:
    """Fetch current supply rates for a protocol on a chain.

    Args:
        protocol: Protocol identifier (e.g., 'aave', 'compound').
        chain: Chain identifier (e.g., 'ethereum', 'arbitrum').
        assets: Optional list of asset symbols to filter.

    Returns:
        Dictionary mapping asset symbol to LendingRate.

    Raises:
        ProtocolError: If the protocol API is unreachable.
    """
```

### Go
```go
// Table-driven tests are mandatory
func TestFetchRates(t *testing.T) {
    tests := []struct {
        name    string
        chain   string
        want    int
        wantErr bool
    }{
        {"ethereum mainnet", "ethereum", 10, false},
        {"unsupported chain", "solana", 0, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := FetchRates(context.Background(), tt.chain)
            if (err != nil) != tt.wantErr {
                t.Errorf("FetchRates() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if len(got) != tt.want {
                t.Errorf("FetchRates() got %d rates, want %d", len(got), tt.want)
            }
        })
    }
}
```

### TypeScript
```tsx
// Functional components, strict types, Tailwind
interface RateCardProps {
  protocol: string;
  chain: string;
  supplyApy: number;
  borrowApy: number;
}

export function RateCard({ protocol, chain, supplyApy, borrowApy }: RateCardProps) {
  return (
    <div className="rounded-lg border border-gray-200 bg-white p-4 shadow-sm">
      <h3 className="text-lg font-semibold text-indigo-600">{protocol}</h3>
      <p className="text-sm text-gray-500">{chain}</p>
      <div className="mt-2 flex justify-between">
        <span>Supply: {supplyApy.toFixed(2)}%</span>
        <span>Borrow: {borrowApy.toFixed(2)}%</span>
      </div>
    </div>
  );
}
```

## Environment Setup

```bash
# Required environment variables
ETHEREUM_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
ARBITRUM_RPC_URL=https://arb-mainnet.g.alchemy.com/v2/YOUR_KEY
POLYGON_RPC_URL=https://polygon-mainnet.g.alchemy.com/v2/YOUR_KEY
BASE_RPC_URL=https://base-mainnet.g.alchemy.com/v2/YOUR_KEY
AVALANCHE_RPC_URL=https://avax-mainnet.g.alchemy.com/v2/YOUR_KEY
DATABASE_URL=postgresql://user:pass@localhost:5432/defi_rates
REDIS_URL=redis://localhost:6379
```

## Deployment Checklist

- [ ] All tests pass (`make test`)
- [ ] Linting clean (`make lint`)
- [ ] Database migrations applied
- [ ] Environment variables set in production
- [ ] RPC endpoints are rate-limited appropriately
- [ ] Redis is running and accessible
- [ ] Docker images built and tagged
- [ ] Health checks passing
- [ ] Monitoring dashboards configured
- [ ] Alert thresholds set for rate changes

---
> Source: [nasrulniroi/nft-portfolio-dashboard](https://github.com/nasrulniroi/nft-portfolio-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
