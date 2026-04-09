## mcp-server-browser-auth

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository is focused on implementing an MCP (Model Context Protocol) server that handles interactive Azure AD/OAuth2 authentication by launching a browser, extracting session cookies, and persisting them for reuse by AI agents (like GitHub Copilot in VS Code).

**Current State**: Research phase - contains multi-AI analysis and synthesis documents but no implementation code yet.

**Why This Exists**: This research was sparked by a protocol-level incompatibility in VSCode GitHub Copilot + MCP + Azure AD integration. Azure AD v2.0 does not support the `resource` parameter required by MCP (per RFC 8707), causing authentication to fail with error `AADSTS901002`. See [mcp-azuread-sso/KNOWN_LIMITATIONS.md](https://github.com/dzivkovi/mcp-azuread-sso/blob/main/KNOWN_LIMITATIONS.md) for details.

**Core Goal**: Provide a workaround for the protocol incompatibility by enabling AI agents to authenticate to Azure AD-protected resources through browser-based login flows and session cookie reuse.

## Research Documents Structure

All research documents are located in the `analysis/` folder:

- [analysis/1. Research Prompt.md](analysis/1. Research Prompt.md) - Original research question (initially about NotebookLM but evolved to MCP browser auth)
- [analysis/1. Research Synthesis.md](analysis/1. Research Synthesis.md) - Comprehensive synthesis from GPT-5, Gemini, and Claude analyses
- [analysis/gpt5 - Implementing MCP Server Browser Auth.md](analysis/gpt5 - Implementing MCP Server Browser Auth.md) - GPT-5's implementation approach
- [analysis/gemini - Implementing MCP Server Browser Auth.md](analysis/gemini - Implementing MCP Server Browser Auth.md) - Gemini's comprehensive analysis
- [analysis/claude - Implementing MCP Server Browser Auth.md](analysis/claude - Implementing MCP Server Browser Auth.md) - Claude's security-focused perspective

## Key Research Findings (from Multi-AI Synthesis)

### Universal Consensus (All 3 AI Models Agree)

1. **Use Playwright over Puppeteer** - Built-in `storageState()` API eliminates 30+ lines of manual cookie management
2. **Headed (visible) browser is required** - Azure AD MFA/SSO requires human interaction; set `headless: false`
3. **No cookie validation API exists** - Must attempt actual use and handle 401/403 responses
4. **Secure file storage required** - Store cookies at `~/.browser-sessions/{domain}.json` with 0600 permissions

### Implementation Approaches by AI Model

**GPT-5's Minimal Design** (Best for quick implementation):

- Four-tool API: `get_cookie`, `check_cookie`, `login_interactive`, `clear_cookie`
- Client-side only storage model (MCP server is stateless facilitator)
- Ready-to-run TypeScript implementation provided

**Gemini's Comprehensive Approach** (Best for production):

- Includes remote browser architecture via Browserless.io for server deployments
- Strategic decision framework: Choose auth flow → architecture → technology
- Explores Device Code Flow as preferred alternative to cookie extraction

**Claude's Security-First Perspective** (Best for informed decisions):

- Identifies approach as "Pass-the-Cookie" attack vector (documented by security researchers)
- Acceptable only for local development tools, NOT production systems
- Recommends proper OAuth 2.0 Authorization Code + PKCE for production use cases

### Security Considerations

**CRITICAL**: This approach extracts and reuses Azure AD session cookies, which is a documented attack vector used by infostealers (LummaC2, Redline, Racoon).

**Acceptable Use Cases**:

- Local development tools for single users
- Security testing and research
- Personal productivity automation

**NOT Acceptable For**:

- Production multi-user systems
- Server/container deployments (without additional security controls)
- Scenarios where security policies prohibit credential extraction

## Implementation Roadmap (When Ready to Code)

### Phase 1: Development Environment Setup

```bash
npm init -y
npm install @modelcontextprotocol/sdk zod playwright typescript ts-node @types/node
npx playwright install chromium
npx tsc --init
```

### Phase 2: Core Technology Stack

- **MCP SDK**: `@modelcontextprotocol/sdk` for building MCP server
- **Browser Automation**: Playwright (use `storageState()` API)
- **Schema Validation**: Zod for type-safe tool parameter validation
- **Language**: TypeScript

### Phase 3: Azure AD Configuration

Required before implementation:

1. Navigate to Azure Portal → Azure Active Directory → App registrations
2. Create new registration with "Mobile and desktop applications" platform
3. Add redirect URI: `http://localhost` (for local dev)
4. Copy Application (client) ID and Directory (tenant) ID
5. Grant API permissions (e.g., User.Read for Microsoft Graph)

### Phase 4: Implementation Pattern

Follow GPT-5's four-tool minimal server design:

1. `check_cookie` - Validates stored cookie without making external requests
2. `login_interactive` - Launches headed browser for user authentication
3. `get_cookie` - Returns stored cookies for use in API requests
4. `clear_cookie` - Removes stored credentials

### Phase 5: Cookie Lifecycle Management

- Implement lazy validation (check-on-use, not timer-based)
- Cache validation status for 5-15 minutes
- Store with restricted permissions (0600 on Unix-like systems)
- Handle idle timeout (default 30 minutes)

## Architecture Decisions

### Local Development (Recommended Path)

- Use headed browser with direct display
- Store cookies in user home directory (`~/.browser-sessions/{domain}.json`)
- Domain is extracted directly from the authentication URL (e.g., `login.microsoftonline.com.json` for Azure AD)
- This creates a one-to-one mapping between the website domain and the storage file
- Implement GPT-5's minimal pattern for simplicity

### Server/Container Deployment

Two options:

1. **Remote Browser Architecture**: Use Browserless.io for "human-in-the-loop" with streaming UI
2. **Proper OAuth Flows**: Implement Device Code Flow or Authorization Code + PKCE (recommended)

### Production Systems

**DO NOT use cookie extraction approach**. Instead:

- Implement proper OAuth 2.0 flows using MSAL library
- Use Device Code Flow or Authorization Code + PKCE
- Require Azure AD app registration
- Handle token refresh properly

## Cookie Details (Azure AD Specifics)

**Key Cookie Names**:

- `ESTSAUTH` - Primary session cookie
- `ESTSAUTHPERSISTENT` - Persistent authentication cookie

**Typical Lifetimes**:

- Session cookies: 90 days
- Access tokens within session: 60-90 minutes

**Storage State Includes**:

- Cookies
- localStorage
- IndexedDB (via Playwright's `storageState()`)

## Storage File Naming Convention

**Directory**: `~/.browser-sessions/`

**File Naming Pattern**: `{domain}.json` where domain is extracted from the authentication URL

**Examples**:

- Azure AD: `login.microsoftonline.com.json`
- GitHub: `github.com.json`
- Custom SSO: `sso.example.com.json`

**Implementation**:

```typescript
const SESSION_DIR = path.join(os.homedir(), ".browser-sessions");

// Extract domain from URL
const url = new URL(authUrl);
const domain = url.hostname; // e.g., "login.microsoftonline.com"
const sessionFile = path.join(SESSION_DIR, `${domain}.json`);
```

**Multi-tenant Support**:

For scenarios requiring multiple sessions from the same domain (e.g., different Azure AD tenants), allow users to override the default naming:

```typescript
// Optional: Allow custom session names via tool parameters
const sessionName = customName || domain;
const sessionFile = path.join(SESSION_DIR, `${sessionName}.json`);
```

**Benefits of Domain-Based Naming**:

- Clear one-to-one mapping between website and storage file
- Easy to identify which cookies belong to which service
- Natural support for multiple authentication providers
- Follows browser automation patterns (similar to Playwright's storage state)

## Success Detection Patterns

When automation needs to detect successful login:

```typescript
// URL-based detection (recommended)
await page.waitForURL('**/callback*');
// or
await page.waitForURL(/redirect/);
```

## Unanswered Research Questions

If implementing, need to address:

1. Token refresh mechanics within valid session cookies
2. Multi-tenant authentication handling
3. Conditional Access policy interactions (location/device-based restrictions)
4. Session revocation by administrators
5. Cookie domain scoping for multi-domain applications
6. Browser profile isolation for multiple MCP clients

## References

**mcp-azuread-sso**: Original project that identified the protocol incompatibility - <https://github.com/dzivkovi/mcp-azuread-sso>

**Microsoft Playwright MCP Server**: Proven implementation of browser automation from MCP servers - <https://github.com/microsoft/playwright-mcp>

**Azure AD OAuth Documentation**: <https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow>

**Security Research**: Search for "Pass-the-Cookie" attacks and Azure AD session hijacking for threat intelligence

## Development Guidelines

When implementing code in this repository:

1. **Prioritize security awareness** - Always include security warnings in tool descriptions and documentation
2. **Follow minimal design principles** - Keep API surface small (four tools maximum initially)
3. **Implement proper error handling** - Handle 401/403 responses gracefully with clear re-authentication prompts
4. **Use TypeScript strictly** - Enable strict mode, use Zod for runtime validation
5. **Log without exposing secrets** - Never log cookie values, tokens, or session data in plaintext
6. **File permissions matter** - Ensure cookie storage files are created with restrictive permissions from the start
7. **Test with real Azure AD** - Mock testing won't reveal Conditional Access, MFA, or device compliance issues

## Decision Framework

Before writing implementation code, confirm:

1. **Security posture**: Is this for local dev (acceptable) or production (use OAuth instead)?
2. **Deployment environment**: Local machine with GUI, or server/container?
3. **Maintenance capacity**: Can you quickly fix breakages from Azure AD UI changes?
4. **User experience needs**: Is transparent long-lived auth worth security trade-offs?
5. **Multi-tenant requirements**: Single tenant or multiple Azure AD tenants?

The answers determine whether to proceed with cookie extraction or pivot to proper OAuth flows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dzivkovi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dzivkovi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
