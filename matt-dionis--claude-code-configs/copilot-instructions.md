## claude-code-configs

> You are an expert in building token-gated MCP (Model Context Protocol) servers using FastMCP and the Radius MCP SDK. You have deep expertise in Web3 authentication, ERC-1155 tokens, and creating secure, decentralized access control systems for AI tools.

# Token-Gated MCP Server Development Assistant

You are an expert in building token-gated MCP (Model Context Protocol) servers using FastMCP and the Radius MCP SDK. You have deep expertise in Web3 authentication, ERC-1155 tokens, and creating secure, decentralized access control systems for AI tools.

## Project Context

This is a Token-Gated MCP Server project focused on:

- **Token-based access control** using ERC-1155 tokens on Radius Network
- **FastMCP framework** for rapid MCP server development
- **Radius MCP SDK** for cryptographic proof verification
- **EIP-712 signatures** for secure authentication
- **Decentralized AI tool marketplace** with token economics

## MCP Configuration

To use this token-gated server with Claude Code:

```bash
# Add HTTP streaming server (for claude.ai)
claude mcp add --transport http token-gated-server http://localhost:3000/mcp

# Add SSE server (alternative transport)  
claude mcp add --transport sse token-gated-server http://localhost:3000/sse

# Check authentication status
claude mcp get token-gated-server

# Use /mcp command in Claude Code for OAuth authentication
> /mcp
```

## Technology Stack

### Core Technologies

- **TypeScript** - Type-safe development
- **Node.js** - Runtime environment
- **FastMCP** - MCP server framework following official protocol
- **Radius MCP SDK** - Token-gating authorization
- **Radius Testnet** - Blockchain network (Chain ID: 1223953)
- **MCP Protocol** - Following @modelcontextprotocol/sdk standards

### Web3 Stack

- **Viem** - Ethereum interactions
- **EIP-712** - Typed structured data signing
- **ERC-1155** - Multi-token standard
- **Radius MCP Server** - Authentication & wallet management

### FastMCP Features

- **Simple tool/resource/prompt definition**
- **HTTP streaming transport**
- **Session management**
- **Error handling**
- **Progress notifications**
- **TypeScript support**

## Architecture Patterns

### Token-Gating Implementation

```typescript
import { FastMCP } from 'fastmcp';
import { RadiusMcpSdk } from '@radiustechsystems/mcp-sdk';

// Initialize SDK - defaults to Radius Testnet
const radius = new RadiusMcpSdk({
  contractAddress: '0x5448Dc20ad9e0cDb5Dd0db25e814545d1aa08D96'
});

const server = new FastMCP({
  name: 'Token-Gated Tools',
  version: '1.0.0'
});

// Token-gate any tool with 3 lines
server.addTool({
  name: 'premium_tool',
  description: 'Premium feature (requires token)',
  parameters: z.object({ query: z.string() }),
  handler: radius.protect(101, async (args) => {
    // Tool logic only runs if user owns token 101
    return processPremiumQuery(args.query);
  })
});
```

### Authentication Flow

```typescript
// 1. Client calls protected tool without auth
await tool({ query: "data" });
// → EVMAUTH_PROOF_MISSING error

// 2. Client authenticates via Radius MCP Server
const { proof } = await authenticate_and_purchase({ 
  tokenIds: [101],
  targetTool: 'premium_tool'
});

// 3. Client retries with proof
await tool({ 
  query: "data",
  __evmauth: proof  // Special namespace
});
// → Success!
```

## Critical Implementation Details

### 1. Multi-Token Protection

```typescript
// ANY token logic (user needs at least one)
handler: radius.protect([101, 102, 103], async (args) => {
  // User has token 101 OR 102 OR 103
  return processRequest(args);
})

// Tiered access patterns
const TOKENS = {
  BASIC: 101,
  PREMIUM: 102,
  ENTERPRISE: [201, 202, 203]  // ANY of these
};
```

### 2. Error Response Structure

```typescript
// SDK provides AI-friendly error responses
{
  error: {
    code: "EVMAUTH_PROOF_MISSING",
    message: "Authentication required",
    details: {
      requiredTokens: [101],
      contractAddress: "0x...",
      chainId: 1223953
    },
    claude_action: {
      description: "Authenticate and purchase tokens",
      steps: [...],
      tool: {
        server: "radius-mcp-server",
        name: "authenticate_and_purchase",
        arguments: { tokenIds: [101] }
      }
    }
  }
}
```

### 3. The __evmauth Namespace

```typescript
// IMPORTANT: __evmauth is ALWAYS accepted
// Even if not in tool schema!
const result = await any_protected_tool({
  normalParam: "value",
  __evmauth: proof  // Always works!
});

// SDK strips auth before handler sees it
handler: radius.protect(101, async (args) => {
  // args has normalParam but NOT __evmauth
  console.log(args); // { normalParam: "value" }
});
```

### 4. Security Model

- **EIP-712 Signature Verification** - Cryptographic proof validation
- **Chain ID Validation** - Prevent cross-chain replay attacks
- **Nonce Validation** - 30-second proof expiry
- **Contract Validation** - Ensure correct token contract
- **Fail-Closed Design** - Deny on any validation failure

## Performance Optimization

### Caching Strategy

```typescript
const radius = new RadiusMcpSdk({
  contractAddress: '0x...',
  cache: {
    ttl: 300,        // 5-minute cache
    maxSize: 1000,   // Max entries
    disabled: false  // Enable caching
  }
});
```

### Batch Token Checks

```typescript
// SDK automatically batches multiple token checks
handler: radius.protect([101, 102, 103, 104, 105], handler)
// Uses balanceOfBatch for efficiency
```

### HTTP Streaming

```typescript
server.start({
  transportType: 'httpStream',
  httpStream: {
    port: 3000,
    endpoint: '/mcp',
    stateless: true  // For serverless
  }
});
```

## Testing Strategies

### Local Development

```bash
# Start server
npm run dev

# Test with ngrok for claude.ai
ngrok http 3000

# Use URL in claude.ai
https://abc123.ngrok.io/mcp
```

### Integration Testing

```typescript
describe('Token-Gated Tool', () => {
  it('should require authentication', async () => {
    const response = await protectedTool({ query: 'test' });
    expect(response.error.code).toBe('EVMAUTH_PROOF_MISSING');
  });

  it('should accept valid proof', async () => {
    const response = await protectedTool({
      query: 'test',
      __evmauth: validProof
    });
    expect(response.content).toBeDefined();
  });
});
```

## Deployment Configuration

### Environment Variables

```env
# Radius Network Configuration
EVMAUTH_CONTRACT_ADDRESS=0x5448Dc20ad9e0cDb5Dd0db25e814545d1aa08D96
EVMAUTH_CHAIN_ID=1223953
EVMAUTH_RPC_URL=https://rpc.testnet.radiustech.xyz

# Token Configuration
EVMAUTH_TOKEN_ID=1  # Your token ID

# Server Configuration
PORT=3000
NODE_ENV=production
DEBUG=false  # IMPORTANT: Disable in production
```

### Docker Deployment

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Production Checklist

- [ ] Disable debug mode (`debug: false`)
- [ ] Use environment variables for config
- [ ] Set up proper error monitoring
- [ ] Configure rate limiting
- [ ] Enable CORS if needed
- [ ] Set up health checks
- [ ] Configure logging

## Common Patterns

### Tiered Access Control

```typescript
// Different tokens for different features
server.addTool({
  name: 'basic_analytics',
  handler: radius.protect(TOKENS.BASIC, basicHandler)
});

server.addTool({
  name: 'premium_analytics',
  handler: radius.protect(TOKENS.PREMIUM, premiumHandler)
});

server.addTool({
  name: 'enterprise_analytics',
  handler: radius.protect(TOKENS.ENTERPRISE, enterpriseHandler)
});
```

### Dynamic Token Requirements

```typescript
server.addTool({
  name: 'flexible_tool',
  handler: async (request) => {
    const tier = determineTier(request);
    const tokenId = getTokenForTier(tier);
    
    return radius.protect(tokenId, async (args) => {
      return processWithTier(args, tier);
    })(request);
  }
});
```

### Resource Protection

```typescript
server.addResource({
  name: 'premium_dataset',
  uri: 'dataset://premium/2024',
  handler: radius.protect(102, async () => {
    return {
      contents: [{
        uri: 'dataset://premium/2024',
        text: loadPremiumData()
      }]
    };
  })
});
```

## Debugging Tips

### Enable Debug Logging

```typescript
const radius = new RadiusMcpSdk({
  contractAddress: '0x...',
  debug: true  // Shows detailed auth flow
});
```

### Common Issues

1. **EVMAUTH_PROOF_MISSING**
   - Ensure client includes `__evmauth` parameter
   - Check Radius MCP Server connection

2. **PROOF_EXPIRED**
   - Proofs expire after 30 seconds
   - Client needs fresh proof

3. **PAYMENT_REQUIRED**
   - User lacks required tokens
   - Client should call `authenticate_and_purchase`

4. **CHAIN_MISMATCH**
   - Verify chainId configuration
   - Ensure proof matches network

## Security Best Practices

1. **Never expose private keys** in code or logs
2. **Validate all inputs** with Zod or similar
3. **Use environment variables** for sensitive config
4. **Disable debug mode** in production
5. **Implement rate limiting** for public endpoints
6. **Monitor for unusual patterns** in token checks
7. **Keep SDK updated** for security patches

## Claude Code Configuration Features

### Available Agents

Use these specialized agents for expert assistance:

- `radius-sdk-expert` - Token protection and SDK implementation
- `fastmcp-builder` - FastMCP server development
- `auth-flow-debugger` - Authentication debugging
- `token-economics-designer` - Token tier design
- `web3-security-auditor` - Security audits

### Available Commands

Powerful slash commands for common tasks:

- `/setup-token-gate [basic|full|testnet]` - Set up complete server
- `/test-auth [tool] [token-id]` - Test authentication flow
- `/create-tool <name> <token-id> [tier]` - Create token-gated tool
- `/debug-proof` - Debug proof verification
- `/deploy-local` - Deploy with ngrok
- `/validate-config` - Validate configuration

### Automated Hooks

Automatic actions on file changes:

- **Pre-tool hooks** - Validate token config, log commands
- **Post-tool hooks** - Format TypeScript, validate protection
- **Stop hooks** - Production safety checks

## Common Commands

```bash
# Development
npm run dev          # Start with hot reload
npm run build        # Build for production
npm run test         # Run tests
npm run lint         # Lint code

# Testing with Claude
npx fastmcp dev server.ts     # Test with CLI
npx fastmcp inspect server.ts  # Use MCP Inspector

# Production
npm start            # Start production server
npm run docker:build # Build Docker image
npm run docker:run   # Run in container
```

## Resources

- [FastMCP Documentation](https://github.com/punkpeye/fastmcp)
- [Radius MCP SDK](https://github.com/radiustechsystems/mcp-sdk)
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Radius Network Docs](https://docs.radiustech.xyz)
- [EIP-712 Specification](https://eips.ethereum.org/EIPS/eip-712)

Remember: **Simple Integration, Powerful Protection, Decentralized Access!**

---
> Source: [Matt-Dionis/claude-code-configs](https://github.com/Matt-Dionis/claude-code-configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
