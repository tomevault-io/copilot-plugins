## simplefin-rs

> A Rust library and CLI for the [SimpleFIN protocol](https://www.simplefin.org/protocol.html) (v2.0.0) — read-only financial data sharing.

# simplefin-rs

A Rust library and CLI for the [SimpleFIN protocol](https://www.simplefin.org/protocol.html) (v2.0.0) — read-only financial data sharing.

## Build & run

```sh
cargo build
cargo test
cargo run -- --help   # CLI usage
```

## Crate structure

```
src/
  lib.rs      # public re-exports
  error.rs    # Error enum
  models.rs   # AccountSet, Account, Transaction, Connection, SfinError
  client.rs   # SimpleFINClient + AccountsRequest
  main.rs     # CLI binary
```

## Key dependencies

| Crate | Purpose |
|---|---|
| `reqwest` | Async HTTP client (rustls, HTTPS-only) |
| `tokio` | Async runtime |
| `serde` / `serde_json` | JSON deserialization of API responses |
| `base64` | Decoding setup tokens |
| `url` | Parsing and manipulating Access URLs |
| `thiserror` | `Error` enum derivation |
| `clap` | CLI argument parsing (`--start`, `--end`, etc.) |
| `jiff` | Date parsing and formatting (`YYYY-MM-DD` ↔ UNIX timestamp) |
| `expect-test` *(dev)* | Snapshot tests for model deserialization |

## Testing

```sh
cargo test

# After changing a model or its Debug output, update snapshots in place:
UPDATE_EXPECT=1 cargo test
```

Model deserialization tests in `src/models.rs` use `expect-test` snapshots — each test calls a shared `check(json, expect![...])` helper that deserializes JSON into `AccountSet` and asserts the full `Debug` output. To add a new model test, write `expect![[""]]` as a placeholder and run `UPDATE_EXPECT=1 cargo test` to fill it in automatically.

`src/client.rs` has unit tests for `from_access_url` parsing and `claim` token validation (no network required). `src/main.rs` has unit tests for `parse_date` covering `YYYY-MM-DD`, raw UNIX timestamps, and invalid inputs.

## Authentication flow

SimpleFIN uses a two-step auth flow. The setup token is single-use; the Access URL is long-lived credentials.

### Step 1 — first-time claim

Direct the user to the SimpleFIN server's `/create` endpoint to obtain a Base64 setup token, then exchange it:

```rust
let client = SimpleFINClient::claim(&setup_token).await?;
// Store this securely — treat it like a password
let access_url = client.access_url_str();
```

A `403` response returns `Error::TokenClaimFailed`. Per the protocol, warn the user: the token may be compromised.

### Step 2 — subsequent runs

```rust
let client = SimpleFINClient::from_access_url(&stored_access_url)?;
```

## Fetching accounts

```rust
// All accounts, all transactions
let account_set = client.get_accounts(AccountsRequest::default()).await?;

// With filters
let account_set = client.get_accounts(AccountsRequest {
    start_date: Some(1_700_000_000),
    end_date:   Some(1_710_000_000),
    pending:    true,
    accounts:   vec!["account-id-1".into()],
    balances_only: false,
}).await?;
```

`start_date` is inclusive, `end_date` is exclusive (UNIX timestamps).

## Working with the response

```rust
// Server-side partial errors (data may still be present)
for err in &account_set.errors {
    // err.code is "prefix" or "prefix.subcode" (e.g. "gen.auth", "con.auth")
    // SECURITY: sanitize err.message before displaying — it comes from the institution
    eprintln!("server error: {}", err.code);
}

for account in &account_set.accounts {
    println!("{}: {} {}", account.name, account.balance, account.currency);

    if let Some(txns) = &account.transactions {
        for t in txns {
            // t.posted == 0 means the transaction is pending
            // t.amount is a numeric string; positive = credit/deposit
        }
    }
}
```

## Error handling

All public methods return `simplefin::Result<T>` (`std::result::Result<T, simplefin::Error>`).

| Variant | When |
|---|---|
| `Error::TokenClaimFailed` | 403 on `/claim` — token may be compromised |
| `Error::Unauthorized` | 403 on `/accounts` — access URL rejected |
| `Error::PaymentRequired` | 402 on `/accounts` |
| `Error::InvalidToken(msg)` | Setup token is not valid base64 or a URL |
| `Error::Http(e)` | Any other HTTP-level failure |
| `Error::UrlParse(e)` | Malformed URL |
| `Error::Json(e)` | Response body failed to deserialize |

## Security notes

- The HTTP client is configured with `https_only(true)` — HTTP URLs are rejected at the client level.
- TLS certificate verification is always on (rustls defaults).
- `SfinError::message` originates from the financial institution. **Sanitize it before displaying to users.**
- Custom `currency` values may be URLs. Strings fetched from those URLs must also be sanitized.
- Store the Access URL at least as securely as you would store financial data.

## CLI usage

```sh
# First-time: claim a setup token
SIMPLEFIN_TOKEN=<base64-token> cargo run

# Subsequent runs — all flags are optional
SIMPLEFIN_ACCESS_URL=https://user:pass@bridge.simplefin.org/simplefin cargo run -- \
  --start 2025-01-01 \   # YYYY-MM-DD or UNIX timestamp, inclusive
  --end   2025-04-01 \   # YYYY-MM-DD or UNIX timestamp, exclusive
  --pending \            # include unposted transactions
  --account <id>         # repeatable; omit for all accounts
```

Get a test token from the SimpleFIN Bridge sandbox.

---
> Source: [cry/simplefin-rs](https://github.com/cry/simplefin-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
