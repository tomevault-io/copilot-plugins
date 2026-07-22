## envops

> - `docs/2026-07-15-postmortem-remote-set-stdin.md` — remote `set` silently wrote empty values: a preliminary ssh call inherited and drained the caller's stdin pipe. Read it before touching `RemoteFile._ssh`, stdin handling in any command, or the fake-ssh test shim (which must keep emulating real ssh's stdin draining).

@README.md

# Development Guide

## Postmortems (`docs/`)

- `docs/2026-07-15-postmortem-remote-set-stdin.md` — remote `set` silently wrote empty values: a preliminary ssh call inherited and drained the caller's stdin pipe. Read it before touching `RemoteFile._ssh`, stdin handling in any command, or the fake-ssh test shim (which must keep emulating real ssh's stdin draining).

## Commands

- Run tests: `uv run pytest`
- Run the CLI directly: `./envops.py show <file>` (plain `python3` shebang, stdlib-only)
- Build distributions: `uv build`; publish to PyPI: `uv publish` (needs a PyPI token)

## Workflow: fixing detection bugs (false positives / negatives)

Misdetection reports ("key X should/shouldn't be masked") follow this TDD loop:

1. **Diagnose first** — run `uv run python scripts/diagnose_false_positives.py <env-file>` against the reported file to see exactly which rule(s) each masked key triggered. Never guess from the masked output alone; several rules can overlap on one key.
2. **Add regression cases before fixing** — extend `TestSecretDetection` in `tests/test_envops.py`. It holds two dicts: `NON_SECRETS` (must print in clear) and `SECRETS` (must stay masked). Copy real non-secret values verbatim; for real secrets, craft fake values with the same shape (length, charset, prefix) — never commit a real credential. Run pytest to confirm the new cases fail (red).
3. **Fix** `looks_like_secret` / the regexes in `envops.py`, then get all tests green.
4. **When loosening a rule, add a tightening case too** — every exemption needs a paired `SECRETS` case proving real secrets of that shape still mask (e.g. `SLACK_WEBHOOK_URL` guards the `_URL` suffix exemption).
5. **Verify end-to-end** on the real .env file(s) with `./envops.py show` — tests use synthetic data; the report came from a real file.
6. **Sync the README** — the "Secret detection" section must describe the current rules. README is both user docs and project instructions (imported above).

## Diagnostic scripts (`scripts/`)

- `scripts/diagnose_false_positives.py <env-file>` — for every masked key, prints which rules matched (`value-prefix`, `url-password`, `key-name`, `url-run-entropy`/`run-entropy` with per-run score/length). Prints key names and rule metadata only, never values.
- `scripts/check_db_url.py <env-file> <KEY>` — dissects a URL-shaped value without printing it: scheme, whether userinfo exists, whether it contains a `:password`, path length/charset. Use it to decide if a masked URL is a real secret (has password / embedded token) or a false positive.

Conventions for these and any new diagnostic script:

- `envops.py` is a plain module at the repo root; scripts insert the repo root into `sys.path` and `import envops as mod`. Reuse that pattern.
- **Value-safe output only**: print booleans, lengths, entropy scores, charsets — never the raw value. A diagnostic that prints values defeats the point of the tool.
- Keep diagnostics in sync with `looks_like_secret` — the reasons they report must mirror the real rule order, or the diagnosis lies.

## Debugging safety rules

- Inspect files under investigation with `./envops.py show` (masked), not `cat`.
- Only use `unsafe-read-value` on keys already proven non-secret (e.g. by `check_db_url.py` or the diagnose script), typically to copy exact values into `NON_SECRETS` test cases.

## Detection design invariants (`looks_like_secret`)

Rule order matters; keep these when changing detection:

1. Positive value checks run first and are never exempted: URL-shaped values (`URL_SPLIT_RE`) are masked per segment by `mask_url` before `looks_like_secret` is even consulted — the userinfo password and random-looking runs always mask; known credential prefixes (`SECRET_VALUE_RE`) mask the whole value. This is why the password in `DATABASE_URL=postgres://u:p@h/db` stays masked even though the key ends in `URL`.
2. Key-name matching (`SECRET_KEY_RE`) is gated by `NONSECRET_KEY_RE`: a trailing config word (`URL`, `ENDPOINT`, `HOST`, `NAME`, `TELEMETRY`, ...) exempts the key-name rule **only** — never the entropy net. A `_URL` key whose value embeds a random token must still mask.
3. The entropy net (`entropy_looks_random`) works on **unbroken alphanumeric runs** (≥20 chars, mixes letters and digits, Shannon entropy ≥3.5) — not on the whole value. Separators (`-` `.` `_` `/` `:`) cut structured values (URLs, hostnames, bucket names, db names) into short runs that pass; real tokens (hex, base62, webhook secrets) survive as one long run. Do not revert to whole-value entropy: readable-but-varied strings like `campuswatch-demo-snapshot` or `oss-cn-beijing.aliyuncs.com` exceed 3.5 as a whole and will false-positive.
4. Prefer generic patterns over enumerations. URL scheme is "reasonable-length word + `://`" (`[a-z][a-z0-9+.-]{0,31}://`), not a list of known schemes — enumerations always miss one.

Lessons already paid for (don't re-learn):

- Char-frequency entropy cannot distinguish human-readable-but-varied strings from random ones; structure (run length, letter+digit mix) is the discriminating signal.
- `AUTH` as a key substring is too broad on its own (`BETTER_AUTH_URL`, `BETTER_AUTH_TELEMETRY`); the suffix gate exists for this.
- A URL being masked doesn't mean it's a true positive — check for an actual password/token in it (`check_db_url.py`) before assuming the mask is correct.

---
> Source: [reorx/envops](https://github.com/reorx/envops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
