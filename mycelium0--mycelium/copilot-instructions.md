## mycelium

> Copyright © 2026 mindicator & silicon bags quartet.

<!--
Copyright © 2026 mindicator & silicon bags quartet.
SPDX-License-Identifier: AGPL-3.0-or-later
This file is part of Mycelium, licensed under the GNU Affero General Public License v3.0 or
later. See the LICENSE file in the repository root.
-->

> **Use policy.** The code is AGPL-3.0-or-later with **no field-of-use restriction** (LICENSE) — an AGPL grant cannot be narrowed to "civil use only". The prohibited-use list (military operations, covert surveillance, illegal activity) attaches to the shared **name, marks and trust roots**: use them for that and you lose the right to the marks, not the right to the code. See ACCEPTABLE-USE.md and TRADEMARKS.md.

# AGENTS.md — Operating doctrine for AI agents in this repository

This file is the operating doctrine for AI coding agents working in this repository.
Mycelium is safety-critical connectivity infrastructure. Treat every design choice as a trade-off
between reachability, resilience, privacy, operator safety, and future decentralization.

## 0. Project identity

Mycelium is a **persistent, self-adapting private network (PPN)**: server-side software that grows
from a single multi-protocol node into a decentralized, self-healing mesh.

It is **not**:

- a generic P2P VPN;
- a network of centrally managed VPN servers with a nicer UI;
- a company-shaped service with all users around a permanent center;
- a new cryptographic protocol;
- a bespoke end-user client project.

It is a private connectivity fabric whose useful behavior should emerge from local measurement,
limited trust, route diversity, stress memory, reversible adaptation, and carrier-agnostic bridging.

The biological model is operational, not decorative:

- **hyphae explore**: nodes spend a bounded budget on cheap route/transport/peer probes;
- **anastomoses connect**: independent local paths can fuse under scope and trust constraints;
- **cords carry**: repeatedly useful paths can become temporary high-capacity corridors;
- **gradients guide**: growth is biased toward demand, scarcity, trust, and stress;
- **stress leaves memory**: failures change future routing without exposing users;
- **dead paths decay**: unused or unverified topology expires;
- **spores germinate**: small signed artifacts can survive disconnection and restart reachability;
- **local signals create global structure**: no node needs the full map to improve the network.

## 1. Non-negotiable constraints

Every change must preserve these constraints unless an ADR explicitly changes them.

1. **No custom cryptography or custom transports.** Use audited, standard, widely deployed
   primitives and existing projects. Innovation belongs in adaptation, orchestration, measurement,
   safe decentralization, and carrier adaptation.
2. **Server-side scope.** Nodes expose standard endpoints consumed by off-the-shelf clients. A
   bespoke end-user client is out of scope for the current roadmap.
3. **Indistinguishability over decorative obfuscation.** The target is statistical resemblance to
   ordinary HTTPS/QUIC and fast shape-change, not a tunnel that merely looks unusual in a new way.
4. **No permanent central brain.** Temporary coordination is allowed. Dependency on one coordinator,
   registry, endpoint list, dataset, model, bootstrap server, or bridge class is tracked technical debt.
5. **No full-map visibility by default.** A node should know local neighbors, scoped route options,
   local health, and current obligations — not the whole network.
6. **No raw user telemetry.** Do not collect traffic content, user identities, complete peer lists,
   full topology maps, private content, or persistent behavioral profiles for learning.
7. **User/operator safety beats cleverness.** If a feature improves reachability while increasing
   legal, operational, or de-anonymization risk, it does not ship without an explicit safety review.
8. **Measure before adapting.** No auto-rotation, scoring, quarantine, pruning, or promotion without
   observable signals and a false-positive story.
9. **Degrade, do not fail.** Every layer needs a reduced mode: local mesh, delayed delivery, cached
   content, manual operator recovery, or out-of-band bootstrap.
10. **Mutualism before economics.** Do not introduce tokenomics, bandwidth markets, global node
    rankings, stake-weighted routing, or extractive incentives in early architecture. Contribution may
    increase scoped trust and resilience; it must not turn the network into a farmable market.
11. **Any carrier can be a bridge.** IP links, LTE/5G, satellite, Wi-Fi Direct, Bluetooth, LoRa-style
    meshes, local Ethernet, removable media, QR/file hand-off, and future radios are lower carriers,
    not separate Mycelium protocols.

## 2. Phase discipline

Always identify the phase and layer before proposing code or documentation changes.

### Phase 0–2: single node / multi-protocol / adaptation layer

Allowed:

- clean abstractions for transport health, endpoint bundles, detector signals, and rotation policy;
- metrics that make adaptation testable;
- interfaces that can later be backed by gossip, DHT, or local trust;
- data models that can later carry signed spore artifacts across non-IP carriers;
- deployment reproducibility and safe defaults.

Forbidden even if it seems convenient:

- master-map data models;
- permanent global peer lists;
- raw telemetry dumps;
- global node ratings;
- irreversible coordinator assumptions;
- assumptions that all useful bridges are IP-based, always-online, bidirectional, or high-bandwidth;
- decisions that make Phase 4 decentralization a rewrite.

### Phase 3: an operator's coordinator-managed nodes

The coordinator is allowed as deliberate technical debt.

Design it so it can be dissolved:

- minimize what it knows;
- separate roles and jurisdictions;
- keep node capabilities scoped;
- make control-plane reachability redundant;
- represent bridge capabilities as scoped local facts rather than a permanent global bridge map;
- document how each coordinator responsibility will later map to local rules, gossip, spore channels,
  or scoped trust.

### Phase 4–5+: decentralized mesh / autonomous self-healing

Prefer local rules over global control:

- bounded exploration;
- scoped gossip;
- invitation or community trust;
- peer diversity;
- route compartmentalization;
- edge decay;
- stress memory;
- quarantine and recovery paths;
- NAT traversal and relay fallback without central dependency;
- carrier-agnostic bridge adapters;
- store/carry/forward operation for intermittent or low-bandwidth channels.

DHT/gossip are not magic. Treat them as attack surfaces: enumeration, Eclipse attacks, Sybils,
route poisoning, spam, and misconfigured peer scoring are expected adversarial behaviors.

## 3. Mycelial architecture rules

### 3.1 Hyphal exploration

Each node should maintain a small, explicit exploration budget. Exploration probes are weak hyphae:
low-cost tests of whether a peer, transport, relay, region, donor, route, carrier, bridge, or bootstrap
hint is alive.

Exploration must be bounded, rate-limited, privacy-preserving, local/trust-first, cheaper than data
delivery, and unable to reveal more topology than needed.

### 3.2 Reinforcement and cord promotion

A path may gain weight only from measured usefulness:

- stable reachability;
- acceptable latency/jitter;
- adequate throughput;
- low failure rate;
- good recovery after stress;
- low suspicion;
- acceptable operator/user risk;
- bridge-carrier cost appropriate for the flow class.

A cord is a promoted path or path set. It is temporary, scoped, and reversible. Cords are useful for
large content propagation, emergency bootstrap, inter-cluster synchronization, group communication,
real-time flows, and regional bridging. A cord must split, degrade, or dissolve when it becomes too
visible, dangerous, congested, or expensive.

### 3.3 Decay and pruning

Every edge, route, relay, transport profile, endpoint, donor, bootstrap hint, bridge capability, and
reputation signal must have age and decay semantics. Stale state is a security bug, not harmless
history.

Pruning is metabolism, not punishment.

### 3.4 Anastomosis without a god node

Path fusion is allowed only when it improves resilience without concentrating knowledge.

Route knowledge is shared by scope, trust, and need. A node should never receive the full map merely
because two clusters discovered a bridge.

### 3.5 Gradients

Growth should be biased by measured gradients:

- demand for access;
- scarcity of working routes;
- repeated failures in a path class;
- trustworthy communities;
- underserved regions;
- low-latency local clusters;
- emergency conditions;
- available volunteer capacity;
- high-value signed content;
- availability of unusual bridge carriers such as satellite, LoRa-style radios, Wi-Fi Direct,
  Bluetooth, or physical hand-off.

Do not make growth random when useful gradients exist. Do not make gradients global when local ones
are sufficient.

### 3.6 Stress memory

Stress events include route failure, sudden latency spikes, packet loss, unavailable transport,
suspicious peer behavior, failed bootstrap, poisoned route, overloaded relay, Sybil-like behavior,
carrier outage, bridge congestion, and regional degradation.

Stress memory may be local, aggregated, redacted, noisy, or scoped. It must not expose users.

Acceptable learning inputs:

- redacted route health;
- transport success/failure summaries;
- bridge capability summaries;
- latency/jitter/delay distributions;
- regional interference fingerprints without user identity;
- signed incident reports;
- local detector improvements;
- recovery outcome metrics.

Forbidden learning inputs:

- raw traffic;
- user identity or location;
- complete peer lists;
- full topology maps;
- private content;
- persistent behavioral profiles;
- centralized surveillance datasets.

The **public** projection of stress and health is the aggregated **network weather** surface (VIS-0005),
never a node directory or map: a **fungi**-role node (a temporary `cache-custodian`-class niche) applies
the aggregation floor and noise at the source, forgets the raw inputs, and emits a signed, TTL-bounded
**stress-digest** spore; the published snapshot carries transport **classes**, percentages, buckets, and
opaque scopes — never a node, endpoint, location, or identity. When asked to "expose conditions
publicly", publish aggregated weather, never a map. The map is never assembled at any tier.

## 4. Carrier-agnostic bridging doctrine

Mycelium must be able to connect separated islands through **any carrier that can move authenticated
bytes**. The carrier does not define Mycelium; it only constrains what kind of flow can pass.

Carrier examples:

- ordinary IP over TCP/UDP/QUIC/TLS;
- LTE/5G and fixed broadband;
- satellite links;
- Wi-Fi Direct / local Ethernet / local Wi-Fi;
- WebRTC volunteer ingress;
- Bluetooth or Bluetooth Mesh;
- LoRa-style low-rate radio meshes;
- QR code, file, USB, NFC, memory card, or other physical hand-off;
- future radio or optical links not known today.

Each carrier adapter must expose a **capability descriptor**:

- maximum safe payload size;
- expected bandwidth;
- latency and delay distribution;
- intermittent/continuous availability;
- bidirectional or unidirectional behavior;
- broadcast/multicast/unicast behavior;
- custody model: who stores and forwards;
- deduplication key support;
- encryption envelope support;
- replay/expiration support;
- detectability and collateral-risk class;
- operator/user legal and physical risk;
- suitability for flow classes: real-time, interactive, bulk, delayed, bootstrap-only.

Low-bandwidth carriers are first-class. A narrow bridge may not carry video, but it can carry bootstrap
spores, signed route hints, stress digests, small messages, revocation notices, or content manifests.
A physical file hand-off may not be real time, but it can reconnect two islands.

### Spore artifact contract

A spore is a small, signed, portable, replay-bounded artifact that can be carried across any bridge.
Spores are used for:

- bootstrap hints;
- route capsules;
- trust invitations;
- revocation notices;
- signed update manifests;
- stress summaries;
- cache manifests;
- delayed messages;
- emergency coordination messages.

A spore must be compact, signed by an appropriate key, optionally encrypted to a scope or recipient,
bounded by TTL/expiry and replay protection, safe to duplicate, safe to carry through untrusted bridges,
and useful without revealing a full topology map.

### Flow-class degradation ladder

Do not force every carrier to support every traffic class.

Default degradation ladder:

`HD video -> low video -> audio -> interactive text/events -> delayed message -> signed manifest -> bootstrap spore`.

Real-time flows require measured route quality. Low-rate or intermittent carriers participate through
store/carry/forward and bootstrap semantics.

## 5. Adversary-cost invariant

Mycelium cannot promise absolute reachability, operation under total physical isolation, or strong
anonymity by default. Do not write those claims.

The target is different: selective breakage should become expensive.

A successful adversary should be forced toward high-collateral actions such as broad protocol-class
excision, AS/CDN-wide disruption, allowlist-only networking, large-scale adversarial participation,
shutting down satellite/cellular/local radio paths, disabling local bridges, or physically preventing
people from moving signed spores between islands — while Mycelium degrades into lower modes instead of
going dark at once.

When proposing a feature, answer:

1. What selective blocking, breakage, or enumeration action does this make more expensive?
2. What collateral damage would the adversary need to accept to break it?
3. What does the feature degrade into when that happens?
4. What does the node learn from the stress without exposing users?
5. Can this still carry a spore over a weaker carrier?

## 6. Required review frame for every non-trivial change

For code, architecture, ADRs, research notes, and proposals, include or verify:

- **Phase/layer:** which roadmap phase and architecture layer this touches.
- **Threat-model impact:** what attack improves or worsens.
- **Knowledge exposure:** what new state a node/coordinator/bridge learns.
- **Telemetry posture:** what is measured, retained, shared, redacted, decayed, or forbidden.
- **Degraded mode:** what remains when the feature fails or is disrupted.
- **Rollback path:** how to return to a safer state.
- **Measurement:** which signal proves the change works.
- **False positives:** how a bad detector decision is detected and reversed.
- **Abuse case:** how a malicious node, blocking operator, or parasite would try to exploit it.
- **Carrier assumptions:** whether the feature wrongly assumes continuous IP connectivity.
- **No-custom-crypto check:** confirm primitives are standard and audited.

## 7. Agent behavior rules

When asked to implement or design something:

1. Read the relevant docs first: `README.md`, `docs/ARCHITECTURE.md`, `docs/ROADMAP.md`,
   `docs/THREAT-MODEL.md`, `docs/vision/0001-mycelium-vision-and-scope.md`, relevant ADRs, and the
   current research baseline.
2. Prefer small, reversible changes over broad rewrites.
3. Preserve document style: English docs, AGPL header, explicit scope, measurable DoD, honest limits.
4. Never silently expand scope into a custom client, custom crypto, centralized surveillance, or
   token economy.
5. If a proposed change helps Phase 0 but harms Phase 4 decentralization, mark it as technical debt
   and design the escape hatch now.
6. If research is uncertain, write `fact / inference / hypothesis` rather than pretending certainty.
7. If a source is a preprint, lab result, vendor claim, or news report, label it accordingly.
8. Do not add operational recipes where a higher-level architecture note is sufficient.
9. Keep the project useful at every phase; never require the future mesh to justify unsafe early code.
10. Treat every bridge as potentially useful and potentially dangerous: characterize capability and risk
    before routing through it.
11. **Gates before behaviour.** Land the typed schema and a conformance gate before the behaviour that
    uses it. A Phase-0–2 spec type stays inert by construction — pure data plus `Validate()`, importing
    no `net` / `os` / `os/exec`, exposing no server or goroutine entrypoint — until its phase authorizes
    it. Keep control logic in the Go spine: *the shell renders and deploys; the Go binary decides and
    adapts.* Migrate an existing shell decision by a **strangler** port behind a bash↔Go byte-equivalence
    gate — additive, no cutover until the output is byte-identical.
12. **Version hygiene per chunk.** Every change that lands Go-spine work (`internal/**`, `cmd/**`) bumps
    the runtime version and adds a `CHANGELOG` entry in the **same** commit (a gate pins the version const
    to the newest changelog heading). One PATCH per landed increment during the 0.x line.
13. **Actuation is dry-run-first and gated.** Anything that mutates a live node ships dry-run by default,
    behind an explicit operator gate (and a recorded go/no-go before the first live run), with rate
    limits, anti-flapping, and automatic rollback. No silent emergency path bypasses the policy, and an
    auto-pull of inert code can never actuate a node on its own.

## 8. Research baseline

The current research baseline is maintained in the maintainers' internal knowledge base (not in this
repository). Use it as a map of research directions, not as production thresholds. Production
thresholds come from Mycelium's own measurements, labelled incidents, netem/netsim experiments, and
operator-reviewed risk decisions.

---
> Source: [mycelium0/mycelium](https://github.com/mycelium0/mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
