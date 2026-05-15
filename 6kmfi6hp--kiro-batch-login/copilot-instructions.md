## kiro-batch-login

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Kiro Batch Login CLI** - A batch semi-automatic login tool that uses Kiro's built-in AWS SSO OIDC configuration for authentication. This tool enables batch login to multiple accounts without requiring any OAuth client credentials configuration.

**Key Innovation**: Uses AWS SSO OIDC's dynamic client registration API (RegisterClient) to obtain temporary OAuth credentials on-the-fly, eliminating the need for pre-configured client_id/client_secret.

## Development Commands

### Installation
```bash
npm install
```

### Running the CLI
```bash
# Single login
node src/index.js login

# Batch login (e.g., 3 accounts)
node src/index.js batch 3

# List saved tokens
node src/index.js list

# Show token details
node src/index.js show <filename>

# Refresh token
node src/index.js refresh <filename>

# Delete token
node src/index.js delete <filename>

# Interactive mode
node src/index.js
```

### Testing Authentication Flow
```bash
# Test BuilderId (AWS Builder ID)
node src/index.js login --provider BuilderId --region us-east-1

# Test Enterprise (requires Start URL)
node src/index.js login --provider Enterprise --start-url https://your-org.awsapps.com/start

# Test Social providers (Google/GitHub)
node src/index.js login --provider Google
node src/index.js login --provider Github
```

## Architecture Overview

### Core Design Pattern: Provider Factory System

The codebase uses a **provider factory pattern** to support multiple authentication methods:

1. **IdC Providers** (AWS SSO OIDC): BuilderId, Enterprise, Internal
   - Uses AWS SSO OIDC APIs (RegisterClient, CreateToken)
   - Implements PKCE (RFC 7636) for security
   - Random port strategy for callback server

2. **Social Providers**: Google, GitHub
   - Uses Kiro's proprietary auth service (auth.desktop.kiro.dev)
   - Predefined port strategy (8443, 8080, 8081)
   - Simplified OAuth flow through Kiro's service

### Module Responsibilities

#### `src/index.js` - CLI Entry Point
- Command-line interface using Commander.js
- Interactive prompts using Inquirer.js
- Commands: login, batch, list, show, refresh, delete
- User input validation and error handling

#### `src/auth-flow.js` - Authentication Orchestrator
- High-level authentication flow coordination
- Calls provider factory to create appropriate provider
- Handles token saving after successful authentication
- Manages refresh token flow

#### `src/providers/` - Provider System
- **`provider-factory.js`**: Creates provider instances based on provider ID
- **`base-provider.js`**: Abstract base class defining provider interface
- **`idc-provider.js`**: IdC authentication (AWS SSO OIDC)
- **`social-provider.js`**: Social authentication (Kiro Auth Service)

#### `src/aws-sso-client.js` - AWS SSO OIDC Client
- Implements AWS SSO OIDC API calls:
  - `registerClient()` - Dynamic client registration
  - `createToken()` - Token exchange (authorization_code, refresh_token)
  - `generatePKCE()` - PKCE parameter generation (RFC 7636)
  - `buildAuthorizationUrl()` - Authorization URL construction
- Based on Kiro's implementation (extension.js lines 107762-107826)

#### `src/kiro-auth-client.js` - Kiro Auth Service Client
- Client for Kiro's proprietary authentication service
- Endpoints:
  - `/login` - Initiate social login
  - `/oauth/token` - Exchange authorization code
  - `/refreshToken` - Refresh access token
- Based on Kiro's mh class (extension.js lines 102072-102180)

#### `src/oauth-server.js` - Local Callback Server
- HTTP server for OAuth callbacks
- Two port strategies:
  - **Random port** (port 0): For IdC providers
  - **Predefined ports** (8443, 8080, 8081): For social providers
- Features:
  - State validation (CSRF protection)
  - 5-minute timeout
  - XSS protection
  - Beautiful HTML response pages
- Based on Kiro's implementation (extension.js lines 123055-123220)

#### `src/token-manager.js` - Token Storage
- Token file management in `tokens/` directory
- File naming: `{provider}_{accountName}_{timestamp}.json`
- Operations: save, list, read, delete, check expiration
- Token format includes: accessToken, refreshToken, idToken, clientId, clientSecret, metadata

## Key Implementation Details

### PKCE (Proof Key for Code Exchange) - RFC 7636

```javascript
// Code Verifier: 32-byte random string
const codeVerifier = crypto.randomBytes(32).toString('base64url');

// Code Challenge: SHA256 hash of verifier
const codeChallenge = crypto
  .createHash('sha256')
  .update(codeVerifier)
  .digest('base64url');
```

**Purpose**: Prevents authorization code interception attacks. Even if the authorization code is stolen, it cannot be exchanged for tokens without the code_verifier.

### OAuth Flow Differences

#### IdC Providers (BuilderId, Enterprise)
1. Register temporary OAuth client via AWS SSO OIDC API
2. Start local server on random port
3. Generate PKCE parameters
4. Build authorization URL with AWS SSO OIDC
5. User manually opens URL and logs in
6. AWS redirects to local server with authorization code
7. Exchange code for tokens using PKCE verifier

#### Social Providers (Google, GitHub)
1. Start local server on predefined ports (8443, 8080, 8081)
2. Generate PKCE parameters
3. Build login URL with Kiro Auth Service
4. User manually opens URL and logs in
5. Kiro Auth Service redirects to local server with authorization code
6. Exchange code for tokens via Kiro Auth Service

### Port Strategy Rationale

- **IdC providers**: Use random port because AWS SSO OIDC allows dynamic redirect URIs during client registration
- **Social providers**: Use predefined ports because Kiro's auth service has pre-configured redirect URIs (http://127.0.0.1:8443/oauth/callback, etc.)

## Security Features

1. **PKCE (RFC 7636)**: Prevents authorization code interception
2. **State Parameter**: CSRF protection using random UUID
3. **Local-only Server**: Binds to 127.0.0.1 (not exposed externally)
4. **Timeout Protection**: 5-minute timeout for callback
5. **XSS Protection**: HTML escaping in response pages
6. **Dynamic Client Registration**: Temporary credentials that auto-expire

## Token File Format

```json
{
  "provider": "BuilderId",
  "region": "us-east-1",
  "accountName": "my-account",
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "1//0gHdP9C...",
  "idToken": "eyJhbGciOiJSUzI1NiIs...",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "expiresAt": "2025-10-21T06:36:23.149Z",
  "createdAt": "2025-10-21T05:36:23.150Z",
  "clientId": "abc123...",
  "clientSecret": "xyz789...",
  "clientSecretExpiresAt": "2025-10-28T05:36:23.150Z",
  "savedAt": "2025-10-21T05:36:23.150Z",
  "version": "1.0"
}
```

## Adding New Providers

To add a new authentication provider:

1. **Determine auth method**: IdC (AWS SSO OIDC) or Social (Kiro Auth Service)

2. **Add to `PROVIDER_CONFIGS`** in `src/providers/provider-factory.js`:
```javascript
'NewProvider': {
  authMethod: 'IdC',  // or 'social'
  region: 'us-east-1',
  startUrl: null
}
```

3. **Update CLI prompts** in `src/index.js`:
```javascript
choices: [
  // ...existing choices
  { name: 'New Provider', value: 'NewProvider' }
]
```

4. **No changes needed** to provider classes - they automatically handle new providers based on authMethod

## Common Issues and Solutions

### Issue: "Client registration failed"
- **Cause**: Network issue or invalid AWS region
- **Solution**: Check network connectivity, verify region is valid

### Issue: "Callback timeout (5 minutes)"
- **Cause**: User didn't complete login within timeout
- **Solution**: Restart authentication flow, complete login faster

### Issue: "Token exchange failed: invalid_grant"
- **Cause**: Authorization code expired (10-minute validity)
- **Solution**: Restart authentication flow

### Issue: "Port already in use" (Social providers)
- **Cause**: Another process using ports 8443, 8080, or 8081
- **Solution**: Server automatically tries next port in list

### Issue: "Refresh token failed"
- **Cause**: Client secret expired (typically 7 days)
- **Solution**: Re-run login command to register new client

## Reverse Engineering Notes

This implementation is based on reverse engineering Kiro's `extension.js`:

- **Client Registration**: lines 107762-107780
- **PKCE Generation**: line 107803
- **Authorization URL**: lines 107803-107811
- **Token Exchange**: lines 107819-107826
- **Callback Server**: lines 123055-123220
- **Kiro Auth Service**: lines 102072-102180
- **Provider Types**: lines 97726-97746

## Dependencies

- **commander**: CLI framework
- **inquirer**: Interactive prompts
- **chalk**: Terminal colors
- **axios**: HTTP client
- **qrcode-terminal**: QR code generation
- **open**: Open URLs in browser (for social providers)

## Project Status

✅ Fully functional for:
- BuilderId (AWS Builder ID)
- Enterprise (AWS IAM Identity Center)
- Google (Social authentication)
- GitHub (Social authentication)
- Internal (AWS Internal)

✅ Features:
- Single and batch login
- Token management (list, show, refresh, delete)
- Interactive CLI
- QR code support
- PKCE security
- State validation

---
> Source: [6Kmfi6HP/kiro-batch-login](https://github.com/6Kmfi6HP/kiro-batch-login) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
