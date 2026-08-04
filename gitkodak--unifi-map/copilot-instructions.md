## unifi-map

> Pulls the UniFi topology from a controller's JSON API and renders it as vector

# CLAUDE.md: unifi-map

Pulls the UniFi topology from a controller's JSON API and renders it as vector
diagrams and editable draw.io files, using real Ubiquiti product artwork. See
`README.md` for usage; this covers what's easy to get wrong when changing it.

This is intended to be published publicly, so keep it site-agnostic and
non-identifying: no real hostnames, subnets, SSIDs, device addresses or
site-specific defaults in code, tests, docs or fixtures. Test data should look
like a plausible generic network, not like anyone's actual one.

## Commands

```bash
make check     # ruff format --check, ruff check, pytest (run before committing)
make map       # fetch + render against the live controller
make sane      # render in the readable (non-UniFi) layout
make offline   # builtin icons, no network access
make demo      # render the shipped demo dataset, no controller needed
make test      # pytest only
```

Single test: `.venv/bin/python -m pytest tests/test_assets.py::TestCatalog`

Tests never touch the network. Fixtures in `tests/conftest.py` are synthetic
payloads with invented MACs; `tests/test_assets.py` writes a catalog straight
into a temp cache so `AssetStore` reads from disk.

## Pipeline

Each stage owns one concern; nothing downstream of `model.py` sees raw
controller JSON.

1. **`config.py`** is the only module that reads `os.environ`. Accepts `UNIFI_*`
   and `UDM_*` names. Keep it that way: it's what makes a future Vault/OpenBao
   backend a single-file change. Credentials are `UNIFI_HOST` plus
   `UNIFI_API_KEY`, and nothing else.
2. **`client.py`** is the only module that talks to the controller. Auth is an
   `X-API-KEY` header set once in the constructor; there is no login, session or
   CSRF token. Network application paths are prefixed `/proxy/network`. `unwrap()` absorbs both the v1 `{"data": [...]}`
   envelope and bare v2 lists, returning `[]` on anything unexpected so a
   controller upgrade thins the diagram instead of raising.
3. **`model.py`** normalizes into `Topology`. All schema quirks land here.
4. **`assets.py`** is the only module that fetches artwork. Cached under
   `--asset-cache` (default `cache/assets`), deliberately separate from the
   snapshot cache so `--cache-dir examples/demo` doesn't get downloads written
   into it.
5. **`layout.py`** is the only module that shells out to Graphviz (`dot`,
   `unflatten`).
6. **`render_dot.py` / `render_drawio.py` / `svg_post.py`** are pure functions
   from `Topology` to text. `theme.py` holds every colour, shape and label.

## Artwork constraints

- **Never vendor Ubiquiti artwork into the repo.** It is their IP. It is fetched
  at runtime and cached under `cache/` (gitignored). `--icons builtin` must stay
  a fully working, network-free path.
- **Match devices on `sysid`, not `model`.** The controller's `model` string does
  not reliably match the catalog's `shortnames` (`USWED72` vs `USPH24P`).
  Catalog sysids are hex strings, the controller reports decimal ints; all 1178
  catalog values are unambiguously hex, so strict base-16 parsing is correct.
- The controller does **not** serve device *images* locally (verified on Network
  10.5.67: every plausible path under `/proxy/network/manage/angular/<hash>/`
  returns the SPA's HTML 404). It DOES serve the icon font, and it serves the
  fingerprint database at `/proxy/network/v2/api/fingerprint_devices/0`.
- **Client artwork is `static.ui.com/fingerprint/0/{dev_id}_{size}.png`**, keyed
  on the fingerprint `dev_id` in `stat/sta` (`dev_id_override` wins). Only
  257x257, 129x129 and 101x101 exist; any other size 302s to ui.com, so treat a
  redirect as "absent" and do not follow it. This is `staticFingerprintOld` in
  the Network UI config.
- **Two frontends exist. Read the right one.** `/manage/` is the legacy Angular
  app; its `getIconClassName` resolves clients to just four icon-font glyphs, so
  reading it will convince you no client artwork exists. The app the browser
  actually loads is the React one served from the UniFi OS root (`/275.*.js`,
  `/main~2.*.js`), and that is where the real image URLs live. When hunting an
  asset, find it in the bundle the browser loads rather than inferring from a
  failed guess.
- Artwork must degrade: no network, no Pillow, or unknown hardware all fall back
  to the shape renderer rather than failing the run.

## Rendering constraints

- **Don't switch `--layout sane` edges to `splines=ortho`.** It looks tidier but
  Graphviz cannot place edge labels on orthogonal routes, so port numbers drift
  far from their link and float beside unrelated nodes. `--layout unifi` *does*
  use ortho, and deliberately suppresses port labels for exactly this reason.
- **Edges are emitted parent → child, the reverse of how they're stored**, so the
  root lands at the top (TB) or left (LR) rather than trailing at the far end.
- **`--layout unifi` omits the title block and legend.** A graph label sets a
  minimum canvas width, which pads a tall narrow map with dead space on both
  sides. The UniFi UI has neither, so dropping them is faithful *and* tighter.
- **Stagger once, before rendering.** `_write_outputs()` applies `unflatten`
  then feeds the *same* DOT to the SVG render and the draw.io coordinate pass.
  Different DOT means draw.io positions disagree with the SVG.
- **`unflatten` reformats the file.** It re-tabs and drops trailing semicolons,
  so `sed`-style patches against generated `.dot` files silently no-op. Change
  `render_dot.py` instead.
- **Graphviz identifiers cannot contain a raw MAC.** DOT reads `:` as a port
  specifier, so `_node_id()` strips colons; `render_drawio.py` reuses it so
  layout lookups line up.
- **Graphviz `<IMG SRC>` needs a filesystem path**, not a data URI.
  `svg_post.inline_svg_images()` rewrites those paths into data URIs afterwards,
  restricted to the icon cache dir so a crafted device name cannot pull in
  arbitrary files.
- **draw.io wants `data:image/png,<base64>`**: comma, *not* `;base64,`.
- **`mxGeometry` needs `as="geometry"`.** `as` is a Python keyword and cannot be
  a `SubElement` kwarg; `_geometry()` sets it afterwards. Without it draw.io
  silently ignores every position and piles all shapes at the origin.
- **Size icon cells to the real aspect ratio** via `IconAsset.display_size()`.
  Rack switches are wide and short; a square cell letterboxes them into a thin
  strip surrounded by dead space.
- **Colour is never the only channel.** The accent palette is Okabe-Ito and every
  distinction is also carried by artwork, shape or line style. Don't add a
  red/green pair that carries meaning alone.
- **Never invent a product match.** `AssetStore.sysid_for_name()` is how UniFi
  hardware appearing as a client gets artwork, and it returns a match only when
  exactly one catalogue entry matches. `g3-flex` genuinely matches both
  `UVC-G3-FLEX` (Protect camera) and `UA-G3-Flex` (Access reader), so ties are
  broken with a device type from another app (Protect's camera list), never by
  preference or ordering. Ambiguous stays ambiguous and falls back to the glyph.
- **Never invent topology.** Clients whose uplink the controller doesn't report
  get anchored to `UNKNOWN_UPLINK_ID`. Don't guess a plausible parent switch.
- **`Topology.infrastructure` includes `Kind.UNKNOWN`** so that placeholder
  survives per-network filtering. Removing it re-orphans those clients.

## Defaults reproduce the UniFi web view

`--icons unifi --layout unifi --theme light` is chosen so the tool matches what
the console shows out of the box. Don't change a default to something "better
looking" without a reason; the point is fidelity first, with `sane` available
for readability.

The single deliberate exception is `--show-offline no`: the UI offers no way to
hide stale hardware, which was specifically wanted. `build_topology()` still
defaults `include_offline=True` (a library shouldn't drop data silently); only the
CLI flips it.

When excluding offline devices, they are left out of `device_macs` too, so the
uplink pass must skip any device not in `topo.nodes`; indexing it directly was a
real KeyError.

## `--layout unifi` is an approximation, and the docs say so

It is deliberately not claimed to be pixel-identical to the controller UI:
Graphviz owns the layout, so sibling order and spacing are its decisions, link
routing differs in its corners and channels, typography and label content are
ours, clients fall back to shapes because the client fingerprint icon database
is not reachable, and the output is static. The README has a section spelling
this out. Keep improving fidelity if you like, but do not let the documentation
start implying an exactness that is not there.

## Whether `unifi` layout is narrower than `sane` is data-dependent

It is on a real network with many sibling clients (1305pt vs 4648pt observed),
and inverts on a small fixture where tree depth dominates. Don't assert it.

## Demo dataset

`examples/demo/` is generated by `scripts/make_demo_snapshot.py`; edit the
script, not the JSON. MACs use the locally-administered `02:` prefix and
addresses are RFC 1918, but the **sysids are real** because that is the artwork
join key; fake ones would leave the demo unable to show icons. `tests/test_demo.py`
enforces both of those properties. The dataset intentionally includes an offline
device, four VLANs, and an unplaceable client so those behaviours are visible.

## Overrides are a stub

`overrides.py` has a working, tested schema and loader; `apply()` raises
`NotImplementedError` and nothing in the render path calls it. See
`docs/overrides.md` for the spec and remaining work. When implementing, an
unmatched or ambiguous selector must be a loud error (a typo that silently does
nothing is worse than a failed run), and user-asserted links must stay visually
distinguishable from what the controller reported.

## Open work

- **ISP logo on the Internet node.** `wan_info()` reads `isp_name` from
  `stat/health`, so the node is already labelled ("Carl's Discount Internet & Tackle"). The logo is
  real: the UniFi infrastructure view renders a brand mark beside each WAN entry,
  a brand mark for the primary on WAN1 and another for the backup on WAN2 (in
  testing, Carl's Discount Internet & Tackle and Cruelty Cable Co.). Both are present, so
  expect a systematic lookup keyed on something like the ISP name or ASN, not a
  sparse hand-maintained table.

  Where it is *not*: searched `/275`, `/905`, `/989`, `/main~0`, `/main~2` and the
  legacy `/manage/` bundles for `isp*Logo|Icon|Image|Brand`, `/isp` URL
  templates, and carrier/provider logo identifiers. Only hits were
  `${NCA}/network-cloud/v2/isp-metrics`, an `/isp-viewer` route, and a legacy
  `ispThroughput.pug`. None of them build an image URL.

  What `images.svc.ui.com` turned out to be: a generic image resizing proxy,
  `https://images.svc.ui.com/?u=<source-url>&w=<px>&q=<quality>`, built by a
  shared `<img>` component in `main~2` that also takes `srcFallbackOffline` and
  `srcFallbackBundled`. It is not ISP-specific and resolves nothing on its own;
  the real logo URL has to arrive as data in the `u` parameter.

  The only ISP data reference in the bundles is
  `${NCA}/network-cloud/v2/isp-metrics`, a **cloud** endpoint, so the logo URL
  may well be cloud-provided. But the local `stat/health` WAN subsystem does
  carry an **`asn`** alongside `isp_name`, `isp_organization` and `wan_ip`, and
  an ASN is exactly the sort of key a brand-logo service keys on. That is
  available from a local API key, so do not write this off as cloud-only without
  testing it.

  Concrete next step: take the ASN from `stat/health` (a real one, not a
  documentation value) and see whether any
  Ubiquiti-hosted path resolves a logo from it, then feed whatever URL that
  yields through the `images.svc.ui.com/?u=...` proxy. If nothing local resolves
  it, only then treat cloud as the answer.

  Do not repeat: greps for `isp*Logo|Icon|Image|Brand`, `/isp` templates or
  carrier/provider identifiers across `/275`, `/905`, `/989`, `/main~0`,
  `/main~2` and the legacy `/manage/` bundles. Also do not look for a webpack
  chunk id-to-hash map; the React bundles do not expose one in the usual form.
- **Infrastructure view.** A rack/cabling-style view of gateway, switches, APs
  and uplinks, separate from the client topology. `--no-clients` approximates it.
- **Apply overrides** (see above).
- **Obfuscation is built** (`--obfuscate`, `obfuscate.py`). Notes for changing it:

  - It runs on the model before anything is drawn, so a renderer cannot leak a
    value that is already gone. Keep it that way rather than filtering output.
  - Node ids are derived from MAC addresses and appear in DOT identifiers and
    draw.io cell ids, so ids are remapped too, not just labels.
  - **Artwork is resolved before scrubbing**, and the icon dictionary is remapped
    through `id_map()`. UniFi hardware appearing as a client is matched on its
    hostname, so obfuscating first silently lost the camera's artwork. That is
    why `id_map()` is public.
  - A client's `detail` is the SSID when it has no fingerprint, and a catalogue
    product name when it does, so `detail` is dropped only when `dev_id` is None.
  - Pseudonyms come from a fixed ordering, never from a hash of the real name,
    which for a short hostname is reversible.
  - The Internet node's label is the ISP name and its ip is the WAN address.
    Both are replaced. An early version kept the label by mistake.
  - `test_nothing_leaks_into_any_output_format` renders SVG, DOT and draw.io and
    asserts no original value survives in any of them. Extend it when adding a
    format; a partially clean export is worse than none.
  - **Known limit, documented in the README:** artwork still shows what a device
    is, brand marks included. `--icons builtin` is the stronger option, and
    `--title` is passed through untouched.

- **CI is built** (`.github/workflows/ci.yml`). Two jobs: `check` runs
  `ruff format --check`, `ruff check` and `pytest` across Python 3.11, 3.12 and
  3.13, then renders the demo dataset plainly and obfuscated to exercise
  Graphviz end to end; `policy` enforces what the test suite cannot, namely the
  that no snapshot, render, icon font or artwork is committed.

  It needs no secrets, because nothing in the suite touches the network and the
  demo renders use `--icons builtin --offline`. **Do not add a job that talks to
  a real controller**; there is nothing to gain and it would mean an API key in
  repository secrets.

- **GitHub community standards.** The repo passes on README and LICENSE and is
  missing the rest of GitHub's checklist. None of it is urgent, all of it is
  cheap, and two are worth actual thought rather than boilerplate:

  Done: `CODE_OF_CONDUCT.md` (Contributor Covenant, added via the web UI).
  Remaining:

  - **Repository description.** Not a file: it is a GitHub setting. Keep it
    consistent with the `description` in `pyproject.toml`.
  - **`SECURITY.md`.** Worth writing rather than pasting a template. The useful
    content is that the tool is read only against a controller, that it wants an
    API key and what that key can reach, that snapshots under `cache/` are a
    MAC, hostname and IP inventory that should not be shared, and how to report
    something privately.
  - **`CONTRIBUTING.md`.** Should say `make check` is the gate, that tests never
    touch the network, that fixtures must stay non-identifying, and that the code
    is largely AI authored under review, since a contributor deserves to know
    that before reading it.
  Also done: `SECURITY.md`, `CONTRIBUTING.md`, and the issue and pull request
  templates under `.github/`. The issue forms ask for console model, Network
  version and site count, since those three explain most of what could go wrong
  and are exactly what the caveats section admits are untested.

## API key auth only

Verified 2026-07-29: an `X-API-KEY` header reaches `stat/device`, `stat/sta`,
`rest/networkconf`, `stat/health`, `v2/api/fingerprint_devices/0` and the web
app's static assets, so a key covers the whole tool including the icon font.
Password auth was removed; do not add it back.

## Observed versus assumed

Everything here was developed against one console: a UDM Pro Max on Network
10.5.67 with a **single site**. Keep the docs honest about that boundary rather
than letting confident prose creep in.

Specifically not verified, and currently caveated in the README:

- Multi-site controllers. The claim that a hand-created site's internal name is
  an opaque string is general UniFi knowledge, not an observation. This is why
  the docs point at the URL and `GET /proxy/network/api/self/sites` instead of
  describing what the value looks like.
- That Lucid imports the `.drawio` output. draw.io itself is confirmed working.

If any of these gets verified, tighten the README instead of leaving a hedge in
place. If one turns out broken, it is a bug, not a documented limitation.

## Data hygiene

`cache/` and `out/` are gitignored; snapshots are written `0600`. A snapshot is a
full MAC/hostname/IP inventory. Never commit one or paste it into an issue.

---
> Source: [gitkodak/unifi-map](https://github.com/gitkodak/unifi-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
