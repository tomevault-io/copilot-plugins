## sirc

> A modern desktop IRC client with XDCC download support, built with Go (Wails3) backend and React/TypeScript frontend.

# SIRC - Simple IRC Client

A modern desktop IRC client with XDCC download support, built with Go (Wails3) backend and React/TypeScript frontend.

## Project Structure

```
/home/user/sirc/
├── main.go                    # Wails app entry point, service registration
├── go.mod                     # Go dependencies
├── Taskfile.yml               # Build automation
│
├── pkg/                       # Go packages (backend)
│   ├── irc/                   # IRC protocol implementation
│   │   ├── client.go          # Connection, message handling (923 lines)
│   │   ├── types.go           # Server, Channel, Message, WhoisInfo structs
│   │   └── templates.go       # Server presets
│   ├── services/              # Wails services (exposed to frontend)
│   │   ├── irc_service.go     # IRCService - main IRC operations
│   │   ├── xdcc_service.go    # XDCCService - pack downloads
│   │   ├── settings_service.go # SettingsService - app preferences
│   │   └── notification_service.go
│   ├── config/                # Settings system
│   │   └── config.go          # 6 config sections, persistence
│   ├── storage/               # Data persistence
│   │   └── storage.go         # servers.json, settings.json
│   ├── download/              # Download manager
│   │   └── manager.go
│   └── dcc/                   # DCC transfers
│       ├── types.go
│       └── transfer.go
│
├── frontend/                  # React/Next.js frontend
│   ├── app/
│   │   ├── page.tsx           # Main IRC client UI (1091 lines)
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                # shadcn/ui components (22 files)
│   │   ├── AddServerDialog.tsx
│   │   ├── JoinChannelDialog.tsx
│   │   ├── ChannelBrowserDialog.tsx
│   │   ├── SettingsDialog.tsx
│   │   ├── WhoisDialog.tsx
│   │   ├── KeyboardShortcutsDialog.tsx
│   │   └── MessageText.tsx
│   ├── lib/                   # Utilities
│   │   ├── message-parser.ts  # IRC message parsing
│   │   ├── notifications.ts
│   │   ├── keyboard-shortcuts.ts
│   │   └── wails-bindings.ts
│   ├── bindings/              # Auto-generated Wails TypeScript bindings
│   └── package.json
│
├── docs/features/             # Feature documentation
└── build/                     # Build artifacts
```

## Technology Stack

- **Backend**: Go 1.24 with Wails3 framework
- **Frontend**: React 19 + Next.js 16 (static export)
- **UI Components**: shadcn/ui + Radix UI + Tailwind CSS 4
- **Desktop**: Wails3 (Go app with embedded web frontend)
- **Package Managers**: Go modules (backend), bun (frontend)

## Development Commands

```bash
# Development (hot reload)
task dev

# Build production binary
task build

# Frontend only
cd frontend && bun run build
```

## Adding New Features

### Backend Service Pattern

1. Create service in `pkg/services/`:
```go
type MyService struct {
    mu sync.RWMutex
    // fields
}

func (s *MyService) MyMethod() error {
    s.mu.Lock()
    defer s.mu.Unlock()
    // implementation
}
```

2. Register in `main.go`:
```go
application.NewService(&services.MyService{})
```

3. Wails auto-generates TypeScript bindings in `frontend/bindings/`

### Frontend Component Pattern

1. Create component in `frontend/components/`:
```tsx
export function MyDialog({ open, onOpenChange }: Props) {
    return (
        <Dialog open={open} onOpenChange={onOpenChange}>
            {/* content */}
        </Dialog>
    )
}
```

2. Import Wails bindings:
```tsx
import { MyMethod } from "@/bindings/sirc/pkg/services/myservice"
```

## Configuration

Settings stored in:
- Linux: `~/.config/sirc/`
- macOS: `~/Library/Application Support/sirc/`
- Windows: `%APPDATA%\sirc\`

Files:
- `servers.json` - Saved server configurations
- `settings.json` - User preferences (6 sections)

## Current Features (v0.2.0)

- Multi-server IRC connections with SSL/TLS
- Auto-reconnect with exponential backoff
- NickServ authentication
- XDCC download support with DCC transfers
- Channel browser (LIST command)
- WHOIS user lookup
- Inline image preview in messages
- Desktop notifications with mention highlighting
- Comprehensive settings system (6 categories)
- Native menu integration with keyboard shortcuts

---

## Planned Feature: Cryptocurrency Wallet Integration

### Overview

Add wallet-centric IRC commands for peer-to-peer cryptocurrency payments, starting with BSV and Solana support.

### MVP Scope

**Supported Coins (Phase 1):**
1. **BSV (Bitcoin SV)** - Using BRC-100 wallet interface standard
2. **SOL (Solana)** - Using @solana/web3.js and gagliardetto/solana-go

### Proposed IRC Commands

| Command | Description |
|---------|-------------|
| `/wallet` | Show wallet status, balances, addresses |
| `/tip <nick> <amount> <coin>` | Send crypto to IRC user |
| `/deposit [coin]` | Show deposit address for coin |
| `/withdraw <address> <amount> <coin>` | Withdraw to external address |
| `/balance [coin]` | Show balance (all coins or specific) |
| `/history [coin]` | Show transaction history |
| `/price <coin>` | Get current price |

### Architecture

**New Backend Packages:**
- `pkg/wallet/` - Multi-chain wallet abstraction
- `pkg/wallet/bsv/` - BSV implementation (BRC-100, go-sdk)
- `pkg/wallet/solana/` - Solana implementation (solana-go)
- `pkg/services/wallet_service.go` - Wails service

**New Frontend Components:**
- `WalletDialog.tsx` - Wallet management UI
- `TipDialog.tsx` - Send payment interface
- `TransactionHistory.tsx` - Transaction list

### Required Libraries

**Go (Backend):**
- `github.com/bsv-blockchain/go-sdk` - BSV transactions, keys, SPV
- `github.com/gagliardetto/solana-go` - Solana transactions, wallets

**TypeScript (Frontend):**
- `@bsv/sdk` - BSV operations (optional, mostly backend)
- `@solana/web3.js` - Solana operations (optional, mostly backend)

### Security Considerations

- Keys encrypted at rest (AES-256-GCM)
- Master password required to unlock wallet
- Transaction limits (configurable)
- NickServ identity verification for tips
- No plaintext key logging

### Implementation Phases

**Phase 1: Core Wallet Infrastructure**
- Wallet service skeleton
- Key generation/storage (encrypted)
- Balance checking for both chains

**Phase 2: Deposit/Withdraw**
- Address generation
- External withdrawals
- Transaction history

**Phase 3: IRC Tips**
- `/tip` command implementation
- User-to-user payments
- Identity verification via NickServ

**Phase 4: UI Polish**
- WalletDialog component
- Settings integration
- Transaction notifications

### File Locations

Feature spec: `docs/features/wallet-integration.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rohenaz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rohenaz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
