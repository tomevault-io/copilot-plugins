## pingrb

> A tiny push notification app to get alerts on anything. Webhooks (Stripe, Cal.com, Hatchbox, StatusCake), generic `curl` POSTs, and a CLI for Claude all turn into clean push notifications on the user's iPhone (Android soon).

# pingrb

A tiny push notification app to get alerts on anything. Webhooks (Stripe, Cal.com, Hatchbox, StatusCake), generic `curl` POSTs, and a CLI for Claude all turn into clean push notifications on the user's iPhone (Android soon).

Domain: pingrb.com. Repo: `ruby-native/pingrb` on GitHub.

## Stack

- Rails 8 (SQLite, Solid Queue/Cache/Cable, Hotwire/Turbo, Tailwind v4)
- iOS app via Ruby Native (`gem "ruby_native"`, Hotwire Native shell)
- Push delivery via `gem "action_push_native"` → APNs
- Hatchbox deploy on Hetzner CPX31 (shared with Joe's other side projects)
- Postmark for transactional email
- Fathom Analytics

## Brand

- Primary color: emerald-700 `#047857`
- App icon: white `❯` chevron on solid emerald
- Font: IBM Plex Mono everywhere
- Style: terminal-mono, lowercase headlines, terse copy. No em-dashes.
- Headline: "A tiny push notification app to get alerts on anything."
- Tagline (saved for elsewhere, not the homepage): "If it can POST, pingrb can push."

## Sources (parsers)

- **Stripe** (HMAC) — payments, refunds, disputes, subscriptions
- **Cal.com** (HMAC) — bookings, reschedules, cancellations
- **Hatchbox** (token-in-URL, form-encoded) — failed deploys
- **StatusCake** (token-in-URL, form-encoded) — site down/up
- **Custom** (token-in-URL, JSON) — passthrough `{title, body, url}` for anything that can POST
- **CLI** (token-in-URL, JSON) — same payload shape as Custom, but the user-facing artifact is a token used by the `pingrb` CLI binary. Setup never exposes the URL.

Honeybadger and UptimeRobot were both removed (webhooks paywalled). StatusCake covers the uptime use case.

### Adding a new parser

1. `app/parsers/<name>_parser.rb` inheriting from `Parser`
2. Add to `Source::PARSER_TYPES` and (usually) `Source::CREATABLE_PARSER_TYPES`
3. Setup + test partials at `app/views/sources/_setup_<name>.html.erb` and `_test_<name>.html.erb`
4. Tests at `test/parsers/<name>_parser_test.rb`
5. Fixture at `test/fixtures/sources.yml`
6. Seed fixtures at `db/seeds/webhooks/<name>/*.{json,txt}`

The parser handles auth via class methods on the parser itself (`self.verify`, `self.requires_signing_secret?`). `WebhooksController` doesn't case-switch on `parser_type` — each parser is self-describing.

## Push notification payload convention

Ruby Native handles push taps based on the payload's `data`. Pass via `ActionPushNative.with_data(...)`:

- **`path:`** — internal route (e.g. `/sources/42`). Loaded in-app.
- **`url:`** — full external URL. `http`/`https` opens in an embedded browser inside the app. Other schemes (`mailto:`, `tel:`, `maps:`, third-party app schemes) open the relevant app if installed.

When both are present, `url` wins. Malformed URLs are dropped silently.

`WebhooksController#deliver_push` passes both `path: source_path(source)` and `url: notification.url`. Since `url` wins when present, a tap opens the parser's external link (Stripe dashboard, GitHub issue, Cal.com booking) in the in-app browser; notifications with no `url` (e.g. Hatchbox) fall back to the in-app source page.

## Deploy + Hatchbox gotcha

- Hatchbox auto-deploys `main` on push.
- **Solid Queue must run inside Puma** in production. Pingrb's `config/puma.rb` gates this on `SOLID_QUEUE_IN_PUMA=true`. Without that env var on the Hatchbox app, push delivery jobs enqueue but never execute.
- iOS app builds via `bundle exec ruby_native deploy`. The build pipeline pulls the **latest `v*` tag** from `ruby-native/native` — commits to main alone are invisible. To ship iOS-side changes, tag a release on the native repo.

## Dev

- Server on port 3010 (`bin/dev`)
- Cloudflare named tunnel at `pingrb.rubynative.com` → `localhost:3010` for receiving real webhooks in dev
- `tmp/public_host` contains the tunnel URL (read by `public_webhook_url` helper for setup partials)
- `bin/ci` runs: setup, rubocop, gem audit, brakeman, tests, seeds replant. Use it before pushing.

## Logging

`WebhooksController` tags every request with `[webhook] [parser_type]` and logs one of:

- `accepted source=<id> title=<title>`
- `rejected: source not found token=<prefix>`
- `rejected: invalid signature source=<id> body_bytes=<n>`
- `rejected: JSON parse failed source=<id> message=<msg>`

Grep `[webhook]` in Hatchbox logs to triage failures.

## Pricing (planned)

- Free during alpha (current state).
- $10 one-time paid download for iOS, $10 one-time for Android. Paid downloads, **not** IAP.
- Web app + Custom source URLs + parsers stay free forever. The paid app is what receives the actual pushes.

## Tunnel + dev DNS gotcha

macOS aggressively caches NXDOMAIN for fresh `*.trycloudflare.com` hostnames. The named tunnel at `pingrb.rubynative.com` sidesteps this for dev. If you ever switch to a quick tunnel, query 1.1.1.1 directly or set the system DNS to 1.1.1.1 to avoid resolver weirdness.

---
> Source: [ruby-native/pingrb](https://github.com/ruby-native/pingrb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
