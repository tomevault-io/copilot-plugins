## hagukumi

> - **Name**: hagukumi (育み — continuative noun of 育む "to raise/nurture"; multi-generational nurturing echo: same verb for children and elders in Japanese)

# 20-actors/hagukumi — CLAUDE.md

## Identity

- **Name**: hagukumi (育み — continuative noun of 育む "to raise/nurture"; multi-generational nurturing echo: same verb for children and elders in Japanese)
- **DID**: `did:web:etzhayyim.com:hagukumi`
- **ADR**: ADR-2605261030 (R0 scaffold, 2026-05-26)
- **Parent ADR**: ADR-2605261000 (Liberation Ladder — L4 Care Tier gate)
- **Status**: R0 scaffold — all cells import-time RuntimeError
- **Cross-actor sibling**: mitate (ADR-2605260100; diagnosis ↔ care continuity pathway)

## Architecture

5 Pregel cells, all privacy-first, structurally encrypted:

```
child_daily_care ─────┐
elder_companionship ──┤
chronic_continuity ───┤── all encrypted XChaCha20 envelope per ADR-2605181100
respite_support ──────┘

meal_delivery (aggregate-only attestation; mitsuho-sourced)
```

Each cell = 1 Pregel graph. Cells communicate via lexicon records on MST (`com.etzhayyim.hagukumi.*`).

## Privacy Invariant (CRITICAL)

Care substrate handles the most sensitive observations in the religious-corp ecosystem.

**Structural enforcement** (not policy):
1. `careSessionAttestation` schema **requires** `encryptedPayloadCid` field; **rejects** any plaintext content field via JSON Schema additionalProperties=false.
2. R0 cells raise `RuntimeError` on import specifically to **prevent accidental plaintext data flow** before R1's encrypted-record framework is Council-attested production-ready.
3. Video stream firmware enforcement: Hitogata/Yutori never write video frames to disk; live telepresence transient-only.
4. Care-recipient identity uses 30-day rotating pseudonym DID per ADR-2605181200 (no stable identifier for adversary correlation).

## Robotics Fleet (R0–R1: human-only; R2+ robotics)

| Robot | Class | Function | Status |
|---|---|---|---|
| (human caregivers) | — | all R0–R1 delivery | live R1+ |
| Hitogata-A | gentle humanoid | optional R2+ subset | Council-attested gentle subclass only; never alone with recipient under G9 |
| Sukoyaka | cold-chain last-mile | meal delivery | yakushi inheritance, R2+ |
| Yutori (ゆとり) | companionship telepresence | R2+ | separate mech-design ADR (hanami precedent ADR-2605260230) |

## Constitutional Gates (G1–G14) — IMMUTABLE R0–R3

Key enforcement notes:

- **G2** (No video recording): Firmware-level. Hitogata/Yutori firmware spec includes signed attestation that no frame-write-to-disk code path exists. Council reviews firmware annually.
- **G3** (Per-session consent): `careSessionAttestation.consentRecordCid` REQUIRED schema field. Cell logic verifies consent timestamp within session window before proceeding.
- **G4** (Caregiver Council vetting): On-chain caregiver registry; sessions REJECT if caregiver DID not in attested registry with current `caregiverAttestation` record.
- **G9** (Human-in-loop): Yutori/Hitogata sessions require co-attestation by synchronous human caregiver DID at session start + every 30 min.
- **G10** (Caregiver work cap): On-chain caregiver work-log; cell logic rejects new session if caregiver shows >12 hr cumulative in past 24 hr or <12 hr recovery since last session.
- **G11** (Emergency escalation): mitate G5 emergency keyword set is shared lexicon (`com.etzhayyim.mitate.emergencyKeyword`); any care-session match triggers immediate mitate XRPC POST.
- **G14** (Multi-gen ratio): Quarterly Council audit; `silenCareReview.cohortRatio` field tracks ratio.

## Non-Goals (N1–N10) — EXCLUDED from R0–R3

- **N7** (No surveillance): Constitutional, not adjustable. No camera-in-home, no audio recording, no location tracking, no biometric monitoring outside immediate-safety mitate G5 trigger.
- **N8** (No guardian replacement): Caregiver is supplement to family/legal guardian, never substitute. Schema requires `guardianConsentCid` field linking to parent/legal-guardian attestation for any under-14 care-recipient.

## Lexicon Namespace

**App lexicon root**: `com.etzhayyim.hagukumi`

4 records (R0 stubs; full schemas R1+):

1. `caregiverAttestation` — onboarding: training + background + Council Lv6+ ≥3 vetting
2. `careSessionAttestation` — per-session; encrypted payload mandatory; aggregate-only public fields
3. `consentRecord` — care-recipient + guardian consent; revocable; on-chain
4. `silenCareReview` — Council attestation scope (privacy + Wellbecoming + multi-gen ratio)

## Pregel Cells (R0 stub bodies)

All R0 cells raise `RuntimeError("hagukumi R0 scaffold: activate via Council ADR + encrypted-record framework Council-attested production-ready")` on import.

### R1 activation triggers
1. ADR-2605261030 Council Lv6+ ratify
2. ≥1 pediatrician + ≥1 geriatrician on Council medical advisory
3. ADR-2605181100 encrypted-record framework production-deployed in CI
4. Caregiver onboarding pipeline (training curriculum + background-check workflow) Council-reviewed
5. Cross-actor mitate G5 emergency-keyword lexicon production-deployed

## Build & Deploy

**R0 status**: Scaffold only. All cells RuntimeError on import (intentional — prevents plaintext data flow).

**Smoke test** (import-only; R0 cells deliberately fail import):
```bash
cd 20-actors/hagukumi
python -c "import hagukumi.cells.child_daily_care" 2>&1 | grep "R0 scaffold"
python -c "import hagukumi.cells.elder_companionship" 2>&1 | grep "R0 scaffold"
python -c "import hagukumi.cells.chronic_continuity" 2>&1 | grep "R0 scaffold"
python -c "import hagukumi.cells.meal_delivery" 2>&1 | grep "R0 scaffold"
python -c "import hagukumi.cells.respite_support" 2>&1 | grep "R0 scaffold"
```

Expected: all 5 imports raise `RuntimeError` with "R0 scaffold" message.

## Related Files

- `/20-actors/hagukumi/manifest.jsonld`
- `/90-docs/adr/2605261030-hagukumi-care-tier-b-actor-r0.md` — Master ADR
- `/90-docs/adr/2605261000-labor-liberation-transition-mechanism.md` — L4 gate
- `/90-docs/adr/2605181100-mst-encrypted-records-signal-keywrap.md` — privacy envelope
- `/90-docs/adr/2605260100-mitate-diagnostic-routing-charter.md` — cross-actor sibling
- `/CLAUDE.md` — Religious-corp status table

---
> Source: [cloud-itonami/hagukumi](https://github.com/cloud-itonami/hagukumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
