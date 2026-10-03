## krobot

> This file is the operating contract for agents working in this repository or using the Krobot CLI. Read it before changing code or calling Kroger APIs.

# Krobot agent guide

This file is the operating contract for agents working in this repository or using the Krobot CLI. Read it before changing code or calling Kroger APIs.

## Mission

Krobot helps a user turn grocery intent into a reviewed Kroger pickup cart through Kroger's documented public APIs. Optimize for correctness, clear user control, minimal API usage, and secure authentication.

The intended workflow is:

```text
grocery request
  -> select store
  -> search candidates
  -> resolve ambiguity with the user
  -> present a cart plan
  -> receive approval
  -> add approved UPCs
  -> emit a structured browser handoff
  -> continue in Kroger with computer-use
  -> receive final order approval
  -> place the order and report confirmation
```

Krobot's public-API layer does not place orders. An agent may complete the unsupported pickup steps in Kroger's visible website using computer-use, provided it follows the browser-handoff contract below. Final order submission always requires explicit user approval immediately before the click.

## Agent access boundary

The intended shopping-agent workspace is this Krobot directory, not the user's
home directory or the parent containing other projects. Keep grocery lists, cart
plans, and other task files inside this directory; use their absolute paths when
calling the CLI. Use the checkout-local executable. Do not read or write outside
this workspace, follow links outside it, select external state paths, import
legacy state, or change global commands/settings as part of normal shopping.
Ask the user when the task requires additional access.

This guide is an operating policy, not an enforced sandbox. The agent runner
must enforce filesystem and tool permissions. Merely starting in this directory
or reading AGENTS.md does not restrict a process. Krobot currently does not
install or configure an agent sandbox.

Keep secrets outside the agent-readable workspace in secure OS storage
(currently macOS Keychain). Let Krobot retrieve them internally; never inspect
the vault directly, dump environment variables, extract browser cookies, or
request credentials in chat. Credential setup belongs in a user-controlled
terminal; shopper sign-in/MFA belongs in the user's browser. Do not inject secrets
into the agent's environment. Local state in `.krobot/` contains references and
metadata, not secret values, and should still be treated as private user data.

Folder restrictions alone do not restrict credential-store APIs, network access,
or browser tools. The runner must explicitly permit the capabilities needed for
Krobot's official API requests, native vault access, loopback OAuth callback, and
user-approved browser handoff. Do not grant arbitrary shell/network access on
the assumption that filesystem restrictions protect credentials.

For a hard guarantee that an agent cannot retrieve secrets, run credential-bearing
operations behind a separately enforced boundary with only approved operations
exposed to the agent. The agent must not be able to modify that trusted executable
or its launch configuration, or bypass it with shell access under the credential
owner's identity. That isolated execution mode is not implemented by Krobot;
do not claim that this guide or the current CLI provides it. Developing Krobot
source and operating a protected shopping installation are distinct roles.

## Command interface

Use the single `krobot` command:

Krobot is a native Rust executable. Build from the repository root with
`cargo build --manifest-path src/krobot-cli-rust/Cargo.toml --release --locked`.
The source-build executable is `./src/krobot-cli-rust/target/release/krobot`;
release bundles use `./bin/krobot`. Either can be invoked by absolute path.
The examples below use `krobot` as shorthand for that executable, or a
user-configured PATH entry. Do not assume an older `bin/krobot` was refreshed
by a source build; only release packaging copies the new binary there.

- `krobot credentials ce|prod`: securely configure that profile in macOS Keychain.
- `krobot use ce|prod`: choose the default profile for subsequent commands.
- `krobot --profile ce|prod COMMAND`: choose a profile for one command without changing the default.
- `krobot COMMAND`: run against the current default profile.

Never ask the user to paste a client secret, OAuth token, password, verification code, or cookie into chat.

Use `krobot --help` for the complete CLI contract, `krobot help workflow` for the end-to-end flow, and `krobot COMMAND --help` for command-specific syntax and boundaries.

## Environment separation

Use Certification first for development and non-production verification:

```sh
krobot use ce
krobot COMMAND
```

Use Production only when the user explicitly wants real account or cart effects:

```sh
krobot use prod
krobot COMMAND
```

Agents should prefer explicit `--profile` on every stateful or remote command instead of relying on a mutable default. Humans may prefer `krobot use` for shorter interactive sessions.

CE and Production use separate API hosts, developer credentials, OAuth tokens, selected-store configs, and usage counters. Never copy a token between environments. A CE cart should not be expected to appear at `www.kroger.com/cart`.

Krobot loads the selected profile's Keychain entries automatically. Profile-specific variables (`KROGER_CE_CLIENT_ID`/`KROGER_CE_CLIENT_SECRET` or `KROGER_PROD_CLIENT_ID`/`KROGER_PROD_CLIENT_SECRET`) are available for trusted noninteractive automation, not for injecting secrets into an agent's environment. Do not use generic credential variables because they weaken environment separation.

## Authentication model

There are two distinct authentication layers:

1. Developer client authentication uses `client_credentials`. It supports generalized Locations and Products calls without a shopper login.
2. Shopper authentication uses OAuth authorization code, state, and PKCE. It is required for `profile` and `cart add`.

The shopper signs in only on Kroger's page. The localhost callback is `http://127.0.0.1:8765/callback` unless `KROGER_REDIRECT_URI` overrides it. Krobot must never collect the shopper's Kroger password.

Relevant scopes:

- `product.compact`
- `profile.compact`
- `cart.basic:write`

## Standard agent workflow

### 1. Check state without changing Kroger

```sh
krobot --profile ce limits status
krobot --profile ce auth status
krobot --profile ce locations current
```

`auth status` and `limits status` are local checks. `locations current` is also local after a store has been selected.

### 2. Find and select a store

With confidential-client app credentials, finding stores does not require
shopper login. Public clients without a secret must sign in first:

```sh
krobot --profile ce locations find --zip 45202 --limit 10
krobot --profile ce locations find --zip 45202 --radius 20 --chain Kroger --json
```

Present the store name and address to the user before selection when more than one plausible location exists. Then select by the exact returned ID:

```sh
krobot --profile ce locations select 01400513
krobot --profile ce locations current
```

For real shopping, use `prod` and the user's ZIP code. Do not infer a home ZIP code or precise location.

### 3. Search products

Use curbside pickup filtering for pickup carts:

```sh
krobot --profile ce products search "whole milk" --fulfillment csp --limit 5
krobot --profile ce products search "whole milk" --fulfillment csp --limit 5 --json
```

Inspect a specific product with location-aware details:

```sh
krobot --profile ce products show 0001111041700 --json
```

Resolve up to 50 known 13-digit product IDs in one official request:

```sh
krobot --profile ce products lookup 0001111041700 0001111042315 --json
```

Batch ID lookup is catalog-oriented. Kroger's specification says the product-ID filter causes other filters to be ignored, so do not assume batch results contain location-specific price or inventory.

### 4. Search a grocery list

Input is a plain-text file with one query per line. Quantities may use `N x item`:

```text
2 x whole milk
eggs
bananas
```

Search it with:

```sh
krobot --profile ce list search /absolute/path/to/groceries.txt --json
```

Use the JSON results to construct candidates. Do not silently choose a match when brand, size, variety, dietary requirements, unit price, or substitution intent is ambiguous.

### 5. Present a cart plan

Before writing to a cart, summarize at least:

- requested grocery and quantity;
- chosen product description, size, and UPC;
- regular and promotional price when available;
- pickup availability and stock level;
- substitutions or unresolved assumptions;
- known subtotal, explicitly labeled as an estimate.

Never claim the estimate is the final total. The public API cannot read the cart, taxes, fees, clipped coupons, or the final checkout price.

### 6. Authenticate the shopper only when needed

```sh
krobot --profile ce auth login
krobot --profile ce auth status
krobot --profile ce profile
```

The browser flow must finish before the three-minute callback timeout. Explain that login grants limited API access to the shopper profile ID and cart write; it does not grant checkout or payment access.

### 7. Add only approved products

Cart writes change external state. Show the exact proposed items and obtain user approval immediately before adding them.

Single item:

```sh
krobot --profile ce cart add 0001111041700 --quantity 1 --modality PICKUP
```

Approved batch:

```sh
krobot --profile ce cart add \
  --item 0001111041700:1 \
  --item 0001111042315:2
```

Resolved JSON file:

```sh
krobot --profile ce cart add-list /absolute/path/to/cart.json --modality PICKUP
```

Do not repeatedly call `cart add` to check whether it worked. The endpoint is write-only, and retries can create duplicate quantities.

### 8. Continue with the browser handoff

Cart-add JSON includes a `next.handoff` object. After every approved item has been added in Production, request the same handoff directly:

```sh
krobot --profile prod handoff pickup --json
```

This command does not open a browser by default. It returns a stable, machine-readable contract containing the starting URL, selected-store context, ordered browser steps, and confirmation boundary. The agent should pass `startUrl` to its computer-use/browser tool so that subsequent UI observations and actions remain in the same agent task.

If a human is operating the browser instead, open it explicitly:

```sh
krobot --profile prod handoff pickup --open
```

Do not attempt a storefront handoff from Certification. Certification carts are isolated and do not appear on `www.kroger.com`.

### 9. Browser checkout protocol

Once computer-use opens the handoff's `startUrl`, the agent may:

1. Verify that Kroger is signed in to the intended account. If sign-in, MFA, or a CAPTCHA is required, pause for the user.
2. Verify the selected store, pickup modality, items, quantities, substitutions, availability, and prices against the approved cart plan. Treat the website as authoritative because the Cart API is write-only.
3. Resolve material differences with the user. Never silently accept a materially different product, price, store, pickup day, or substitution rule.
4. Apply available coupons, choose substitution preferences, use a saved payment method, and select a pickup slot when these choices follow the user's stated preferences. Ask when they do not.
5. Reach the final order-review screen and report the store, pickup time, unavailable/substituted items, discounts, fees, taxes, payment summary, and final total.
6. Obtain explicit user approval immediately before activating the final place-order control.
7. After approval, activate that control once. Report the confirmation number and pickup details shown by Kroger. Do not retry an ambiguous submission; inspect the page or order history first.

Never expose, transcribe, or store passwords, complete payment-card details, cookies, or session tokens. Keep the browser visible throughout purchase review. A prior approval to add items to the cart is not approval to place the order.

## Machine-readable operation

Agents should pass `--json` for locations, products, list searches, profile data, and limits whenever subsequent reasoning depends on the output. Human-readable output is for terminal inspection only.

Use `handoff pickup --json` for computer-use transitions. Do not scrape prose from `browse cart`, and do not launch a shell-controlled browser when the computer-use tool needs to own the tab.

Prefer one larger supported request over repeated calls:

- Request enough location results in one call, up to the documented limit.
- Use product `--limit` thoughtfully; do not retrieve 50 results when five candidates are sufficient.
- Use `products lookup` for known IDs rather than one catalog call per ID.
- Cache reasoning within the current task instead of repeating identical searches.

Do not parse the human-readable pipe-delimited display when `--json` is available.

## API limits

Krobot locally enforces the downloaded official limits:

| Bucket | Calls per UTC day |
| --- | ---: |
| Products | 10,000 |
| Cart | 5,000 |
| Identity | 5,000 |
| Locations | 1,600 |
| Chains | 1,600 |
| Departments | 1,600 |

Inspect usage before large jobs:

```sh
krobot --profile ce limits status --json
```

Failed calls count conservatively. Never bypass the local limiter, delete usage state to gain more calls, or automatically retry HTTP `429`. Respect `Retry-After` when present. Remember that local counters cannot see calls made outside this installation; Kroger's server remains authoritative.

## Supported capability boundary

The downloaded public contracts support:

- OAuth client and shopper authentication;
- profile ID;
- locations, chains, and departments;
- product search and details;
- regular and promotional product prices;
- coarse inventory and fulfillment flags;
- write-only cart additions.

They do not support:

- reading, updating, or removing cart contents;
- a complete deals feed or structured offer terms;
- browsing or clipping coupons;
- substitutions or shopper notes;
- pickup-slot scheduling;
- addresses or payment methods;
- checkout, order submission, order status, or order history through the public API. These are browser-handoff capabilities only.

Do not represent an unsupported feature as implemented.

## Undocumented storefront endpoints

Requests such as `/atlas/v1/product/v2/products` observed in browser developer tools are not included in the official public specifications. They may reveal useful concepts, but agents must not build execution paths around them, replay user cookies, reverse-engineer private authentication, or assume their stability.

Use the supported overlap instead. For example, public `filter.productId` supports a batch of up to 50 known IDs, and product details expose nutrition, price, fulfillment, and coarse stock when Kroger supplies them.

## Security practices

- Keep developer secrets in macOS Keychain through `krobot credentials ce|prod`.
- Never commit `.env`, config, usage, OAuth tokens, cookies, HAR files containing headers, or customer data.
- Never print secrets in diagnostics or enable shell tracing around credential setup.
- Use absolute file paths for agent-generated grocery or cart files.
- Treat product, price, inventory, and nutrition data as potentially incomplete or stale.
- Treat allergy and dietary matching as high consequence: verify product labels and flag uncertainty.
- Keep CE and Production isolated.
- Require explicit approval immediately before external cart mutations.
- Use only the visible Kroger website for checkout; never call or reverse-engineer undocumented checkout interfaces.
- Require a fresh, explicit user approval at final review before activating the place-order control.

## Development rules

The implementation lives in `src/krobot-cli-rust`; its public README describes
storage, security, and verification. Keep the executable native and checkout-local.
Compiled Rust dependencies are allowed; no separately installed language runtime
is required by users. Preserve the public CLI/JSON contracts and synthetic fixtures.
The legacy implementation has been removed by user request; do not restore it
or add a JavaScript wrapper.

For the Rust implementation, default runtime state to the checkout's ignored
`.krobot/` directory, resolving the project marker and executable location rather
than an unrelated working directory. Standalone binaries require an explicit
`--data-dir` or `KROBOT_DATA_DIR`; never silently fall back to the home directory.
Keep secrets in secure OS storage with exact, installation/profile-scoped entry
ownership. Preserve legacy Keychain entries; credential replacement and legacy
state import must be explicit. Setup must not modify global shell configuration,
Keychain settings, unrelated credentials, or other checkouts. These boundaries
apply to all setup, import, account, and shopping commands.
Rust status uses local token metadata without accessing Keychain; it does not
verify the remote session. Rust legacy import is explicit and source-preserving.
Rust OAuth login, token refresh, HTTP transport, and profile reads are implemented.
Shopping CLI dispatch, JSON/terminal formatting, and browser handoffs are now
implemented and covered by frozen compatibility contracts. Offline acceptance
and live CE application authentication/store/product checks passed. Shopper
login, secure token writes/refresh, cart writes, and browser checkout remain
unverified with a live shopper account; do not claim otherwise.
Rust validates entire grocery/cart input files before remote work and bounds
them to 1 MiB. Grocery quantities and cart quantities must be 1–99. A batch with
any DELIVERY items returns `next.available: false` instead of a pickup handoff.
`--quantity` applies only to the positional UPC; repeated `--item` entries encode
their own quantities as `UPC:QUANTITY`. Rust rejects a standalone `--quantity`
with only `--item` entries instead of silently ignoring it.
Rust `browse --json` returns `{ "opened": true, "url": "..." }` only after the
explicitly requested browser launch succeeds. Prefer `handoff pickup --json`
for agents so no separate browser window is opened.
The Rust README documents how to run verification. Automated tests
must continue using fake credentials; real legacy import and live smoke tests
are separate, explicit operations. Never treat a CE app credential as proof
that a shopper's normal login works in Kroger's Certification site.

The checked-in OpenAPI files under `openapi-official-specs/` are the contract source of truth. When they are updated:

1. Review changed versions, paths, scopes, parameters, schemas, and limits.
2. Update the implementation and public capability guidance in this file and README.md if necessary.
3. Run contract and unit tests.

Required verification after code changes:

```sh
cargo fmt --manifest-path src/krobot-cli-rust/Cargo.toml --check
cargo clippy --manifest-path src/krobot-cli-rust/Cargo.toml --all-targets --locked -- -D warnings
cargo test --manifest-path src/krobot-cli-rust/Cargo.toml --locked
cargo build --manifest-path src/krobot-cli-rust/Cargo.toml --release --locked
```

Tests must use mocked HTTP and temporary config paths. They must never require real credentials, consume real API quota, open a browser, or mutate a real cart.

Keep builds and runtime state out of commits. Release packaging must use an
explicit public-file allowlist, never archive the entire checkout. Do not
publish, replace global commands, or change shell configuration as a side effect
of a build. Only a user-requested installation may change a global command.

## Related documentation

- `README.md`: installation and user-facing command reference.
- `src/krobot-cli-rust/README.md`: Rust setup, implementation status, and verification.
- `openapi-official-specs/`: downloaded Kroger contracts.

Keep internal planning, research, and verification notes in the Git-ignored
`private-docs/` directory. Keep user-facing instructions in the public READMEs
and this guide; they must remain usable without any private documents.

---
> Source: [zj3500/krobot](https://github.com/zj3500/krobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
