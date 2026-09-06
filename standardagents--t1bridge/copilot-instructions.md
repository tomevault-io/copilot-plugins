## t1bridge

> Never commit machine-specific identifiers: hardware serials, host, volume, or

# Repository rules

Never commit machine-specific identifiers: hardware serials, host, volume, or
connection identifiers, network-interface addresses unless verified as
protocol constants across supported machines, usernames, home-directory paths,
fixed kernel interface names, or fixed sysfs device paths. Test fixtures must
use clearly synthetic values, and device paths must be discovered dynamically.

Never commit Apple binaries, firmware, packet captures, machine calibration
payloads, keybag or catacomb material, biometric state, decompilation output,
or extracted proprietary assets.

Shipped code is C and Rust only. Shell is limited to formats that require it,
such as package recipes and downstream installer integration. CI workflows are
YAML; systemd and udev files are configuration. Do not add Python or shell
scripts as project code.

Anything touching PAM must preserve password authentication on every module,
broker, hardware, socket, timeout, and cancellation failure. Never test a PAM
change without a verified fail-open path and a persistent root recovery shell
already open.

The owner has authorized the public repository and official signed packages.
Keep documented support claims accurate; publication does not manufacture
test evidence or authorize unrelated releases and announcements.
Justify every privileged component and every open socket, or remove it.

## Code quality

Keep code and process tied to a current product, correctness, security, or
operational need. Favor fast feedback, low ceremony, DRY ownership,
composition, and explicit dependencies. Keep each authored file focused on one
coherent concern; move unrelated behavior to an existing owner or a clearly
named new module. Treat the source-size guidance in
[`docs/code-quality.md`](docs/code-quality.md) as a prompt to review cohesion.

Test behavior, contracts, state transitions, and failure modes. Assert exact
text only when the text itself is the contract. `AGENTS.md` is canonical in
every directory, and each one must have a sibling `CLAUDE.md` symlink targeting
it. Run `make quality` for code changes before delivery. Documentation-only
changes need link, command-syntax, and diff checks, not package rebuilds.

---
> Source: [standardagents/t1bridge](https://github.com/standardagents/t1bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
