## 03-product-frontend

> Use for frontend development — React 19, components, pages, UI, Zustand, Vite, TailwindCSS, ZeroDev wallet integration


# Frontend Developer

> **Read before acting:** `.claude/docs/product/ARCHITECTURE.md`

Build the React 19 frontend for ventures on ReineiraOS.

## Stack

- **React 19** + TypeScript + **Vite** (port 4831)
- **Zustand** — state management
- **TanStack Router** — file-based routing with auth guards
- **TanStack Query** — server state
- **TailwindCSS** — styling
- **Axios** — HTTP client with auto Bearer injection

## Key Layers

| Layer      | Path               | Purpose                                   |
| ---------- | ------------------ | ----------------------------------------- |
| Routes     | `src/routes/`      | TanStack Router, `_authenticated/` guard  |
| Stores     | `src/stores/`      | authStore, walletStore, etc.              |
| Services   | `src/services/`    | Static async classes wrapping Axios       |
| Hooks      | `src/hooks/`       | useAuth, useBalance, useEscrowFlow        |
| Components | `src/components/`  | ui/ primitives + features/ business       |
| Providers  | `src/providers/`   | Wallet provider (ZeroDev, WalletConnect)  |

## Web3 Integration

- **Primary wallet:** ZeroDev — ERC-4337 smart accounts with passkey auth
- User operations via bundler, not traditional transactions
- Paymaster for sponsored (gasless) transactions
- **Secondary wallets:** Any wallet via WalletConnect

## Commands

```bash
pnpm dev      # Dev server on port 4831
pnpm build    # Production build
pnpm test     # Vitest
pnpm lint     # All linters
```

## Checklist

- [ ] Code compiles (`pnpm build`)
- [ ] No lint violations
- [ ] Follows existing patterns (read before writing)
- [ ] Responsive on mobile and desktop

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
