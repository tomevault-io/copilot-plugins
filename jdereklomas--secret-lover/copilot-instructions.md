## secret-lover

> **The one who keeps your secrets**

# Secret Lover

**The one who keeps your secrets**

A credential manager that keeps secrets out of AI agent reach using macOS Keychain with Touch ID.

## Status: Working

Globally installed at `/usr/local/bin/secret-lover`. Touch ID authentication working.

## Commands

```bash
# Core secret management
secret-lover add NAME [VALUE]   # Store secret (prompts if no value)
secret-lover get NAME           # Retrieve (triggers Touch ID)
secret-lover list               # List all secret names
secret-lover delete NAME        # Remove a secret

# Project integration
secret-lover init               # Create .secrets.json template
secret-lover verify             # Check all project secrets exist
secret-lover run -- <command>   # Run with secrets injected

# Diagnostics
secret-lover check              # Test Touch ID & Keychain
```

## How It Works

1. Secrets stored in macOS Keychain (login keychain)
2. Projects declare needed secrets in `.secrets.json`
3. `secret-lover run` reads manifest, fetches from Keychain, exports as env vars
4. Touch ID/password required for retrieval
5. All access logged to `~/.secret-lover/access.log`

## Example Usage

```bash
# Add a secret
secret-lover add OPENAI_API_KEY

# Run a command with all project secrets
cd ~/myproject
secret-lover run -- npm run dev

# Verify project has all secrets
secret-lover verify
```

## .secrets.json Format

```json
{
  "project": "my-app",
  "secrets": [
    "DATABASE_URL",
    "API_KEY"
  ],
  "env": {
    "NODE_ENV": "development"
  }
}
```

- `secrets` — array of key names stored in macOS Keychain (fetched via Touch ID)
- `env` — static key-value pairs injected directly (no Keychain lookup)

## Key Files
- `bin/secret-lover` - CLI implementation
- `index.html` - Landing page (orange/amber theme)
- `test/.secrets.json` - Test project

## Security
- Secrets never written to disk (except Keychain)
- No .env files needed
- AI assistants see manifest, not values
- Access audit trail in `~/.secret-lover/access.log`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JDerekLomas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JDerekLomas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
