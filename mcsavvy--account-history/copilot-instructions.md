## account-history

> This project integrates with the Stacks blockchain for wallet connection and smart contract interaction.


# Stacks Blockchain Development

This project integrates with the Stacks blockchain for wallet connection and smart contract interaction.

## Wallet Integration

- **Wallet Hook**: [hooks/wallet.ts](mdc:hooks/wallet.ts) provides `useWallet()` hook
- **Connection Flow**: Uses `@stacks/connect` for wallet connection
- **Session Management**: [providers/auth-session-provider.tsx](mdc:providers/auth-session-provider.tsx) manages authentication state
- **Storage**: Uses localStorage for session persistence

## Key Patterns

### Wallet Connection
```typescript
const { data, isConnected, connect, disconnect } = useWallet();
```

### Session Management
```typescript
const { session, setSession, clearSession } = useAuthSession();
```

## Stacks Dependencies

- `@stacks/connect` - Wallet connection library
- `@stacks/network` - Network configuration
- `@stacks/transactions` - Transaction building

## Smart Contracts

- **Clarinet**: Use [contracts/Clarinet.toml](mdc:contracts/Clarinet.toml) for contract development
- **Testing**: Vitest configuration in [contracts/vitest.config.js](mdc:contracts/vitest.config.js)
- **TypeScript**: Contract types in [contracts/tsconfig.json](mdc:contracts/tsconfig.json)

## Environment Configuration

- **Client Config**: [lib/config/client.ts](mdc:lib/config/client.ts) for client-side environment variables
- **Server Config**: [lib/config/server.ts](mdc:lib/config/server.ts) for server-side configuration
- **Validation**: Zod schemas for environment variable validation

## Best Practices

1. Always validate wallet connection state before making transactions
2. Use proper error handling for wallet operations
3. Implement proper session cleanup on disconnect
4. Validate environment variables at startup
5. Use TypeScript for type safety in wallet interactions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mcsavvy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
