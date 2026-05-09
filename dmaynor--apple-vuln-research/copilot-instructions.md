## apple-vuln-research

> Systematic security research of the MacBook Neo (A18 Pro) — the first A-series chip in a Mac laptop.

# Apple Silicon Vulnerability Research — A18 Pro (MacBook Neo)

## Project Purpose
Systematic security research of the MacBook Neo (A18 Pro) — the first A-series chip in a Mac laptop.
This is a dedicated R&D machine. All research is authorized.

## Research Approach
- Research drives skills, not the other way around. Extract skills at natural pause points.
- Software-first: exhaust OS-level research before hardware probing.
- Document everything. Every finding, dead end, and decision gets recorded.
- When we need a tool, build it first if practical. Register it in `tools/registry/TOOL_REGISTRY.md`.

## Repository Structure
```
findings/          — Research findings organized by layer
  hw/              — Hardware-layer findings
  os/              — OS-layer findings
  analysis/        — Cross-cutting analysis and diffs
artifacts/         — Raw dumps, binary artifacts, exported data
tools/             — Custom tools and tool registry
  registry/        — Tool documentation and metadata
  custom/          — Custom-built research tools
docs/              — Design docs, specs, research plans
scripts/           — Automation scripts (loop, orchestration)
```

## Research Loop
This project uses a restart-with-state research loop:
1. Read `RESEARCH_STATE.md` for current progress
2. Pick next pending task
3. Research, document findings, commit
4. Update `RESEARCH_STATE.md` and `PROGRESS.md`
5. Exit cleanly — loop restarts with fresh context

State files:
- `RESEARCH_STATE.md` — machine-readable task tracking (what's done, what's next)
- `PROGRESS.md` — human-readable summary of all work done
- `RESEARCH_PROMPT.md` — the prompt fed to each loop iteration

## Conventions
- Findings are markdown with structured JSON artifacts where machine-parseable data is needed
- Commit frequently with descriptive messages
- Use agents for parallel research — orchestration agent dispatches, document agent writes
- Skill family namespace: `apple-vuln-research-*` with `{chip}-{os}-{hw|os}` qualifiers
- Custom tools go in `tools/custom/<tool-name>/` with docs in `tools/registry/`

## Hardware
- **Target:** MacBook Neo (Mac17,5) — Apple A18 Pro (t8140), 6 cores (2P+4E), 8GB RAM, macOS 26 Tahoe (25D2140)
- **Baselines:** MacBook Air M5 (replacement unit — original bricked by TXM panic, returned to Apple; new unit acquired, not yet set up), MacBook Air M3
- **Lab equipment:** Saleae 16/16, Rigol oscilloscope, Tigard, Bus Pirate v3/v5, Dreamscope

## Cross-Platform Research Strategy
The A18 Pro (t8140) is identical silicon to iPhone 16 Pro. MacBook Neo serves as an iPhone research proxy:
- **Shared**: kernel, IOKit, DART, SEP, ANE, GPU, ISP silicon and drivers (42 iOS-shared kexts = 17%)
- **Mac advantages**: no app sandbox by default, dtrace/lldb/kperf available, SIP can be disabled, DriverKit debuggable in userspace
- **Porting barriers**: iPhone has stricter entitlement checks, app sandbox limits service reach, no permissive boot
- **Mac-only surfaces**: MediaTek MT7932 WiFi/BT (iPhone uses Broadcom)
- **Strategy**: develop and test IOKit/Mach/kernel primitives on Mac where you have visibility, then assess iPhone applicability

## Apple Bounty Submission Rule

**If we don't have a working exploit, Apple doesn't care.**

All 5 reports submitted to Apple have been dismissed as "not materially relevant to security" — including V-047 with a deterministic PoC, 12+ kernel panics, attacker data in registers, Ghidra RE, and PGZ-confirmed OOB across 6 zones. Apple's bounty program pays for demonstrated code execution, not bug reports. Do NOT submit to Apple until the finding includes a full chain from unprivileged to kernel code execution (or equivalent proven impact). Anything less will be dismissed regardless of technical depth.

**Alternatives for findings without full exploitation:**
- ZDI (Trend Micro) — pays for documented kernel bugs without requiring full exploit
- Black Hat / DEF CON — research talks, the story matters more than the shell
- Continue building toward full chain, then resubmit to Apple with exploit

## Reports Filed

5 reports submitted to Apple Security Research — **ALL DISMISSED**:
1. TXM kernel panic via AMFI sel 16 (CVSS 7.7) — bricked M5 Air — **DISMISSED**
2. BT DEXT kernel panic, 3 vectors (CVSS 7.5) — MT7932 only — **DISMISSED**
3. Apple Intelligence preference injection (CVSS 3.3) — debug keys in prod — **DISMISSED**
4. Siri XPC crash DoS, 3 services (CVSS 4.0) — **DISMISSED**
5. V-047 AppleJPEGDriver kernel panic + OOB (3 bug classes, deterministic PoC) — **DISMISSED**

## ⚠️ Reports Pending Submission — SRD Prompt-Reporting Requirement

Two fully verified, high-severity reports exist but have NOT been submitted to Apple. Per Apple SRD program rules, confirmed vulnerabilities must be reported promptly. See `docs/srd-compliance.md` for full compliance tracking.

- **Report #5** (`reports/05-networksetup-dns-proxy-hijack.md`): V-036 networksetup confused deputy — unprivileged DNS/proxy/WiFi hijack (CVSS 7.1, High). E2E validated on Neo, 7/7 attack vectors confirmed with scutil --dns. **ACTION: SUBMIT TO APPLE.**
- **Report #6** (`reports/06-aes-accelerator-kernel-hang.md`): V-003 IOAESAccelerator uninterruptible kernel hang — unkillable zombie, SIGKILL fails, reboot required (CVSS 7.5, High). Confirmed on A18 Pro + M5. **ACTION: SUBMIT TO APPLE.**

## ⚠️ Third-Party Disclosure Required — MediaTek

SRD rules require reporting third-party bugs to the relevant third party. Report #2 (BT DEXT) was sent to Apple but the underlying MT7932 firmware has MediaTek-specific issues requiring disclosure to MediaTek PSIRT (psirt@mediatek.com):
- `jtag_detect_enable` function in production MT7932 firmware binary (JTAG backdoor risk)
- `test-driver` entitlement in production DEXT build
- No firmware image validation in `SendImage` download path (V-004 — no signature, no magic check)

**ACTION: File disclosure with MediaTek PSIRT referencing Apple Report #2.**
See `docs/srd-compliance.md` for full details.

## Phase Cap & Loop Status

- **Neo loop:** Phase 36+ of 40 (cap), verification/synthesis tasks in progress
- **Multi-model review:** `scripts/multi-model-review.sh` runs in tmux session `review` — Claude orchestrates Codex, Gemini, Copilot for 4-agent finding review. Both loops can run simultaneously.
- **Phase cap at 40:** After Phase 40, loop switches to verification/synthesis mode
- **Post-Phase-40:** Auto-generates RESEARCH_PLAN_V2.md (hindsight-optimized)
- **Quality:** 42% HIGH, 26% MEDIUM, 32% LOW in self-directed phases
- **Output:** 73 custom tools, 150+ findings files, 44 confirmed vulnerabilities (all manually verified)

## Priority Attack Surfaces (Updated 2026-03-16)

**Confirmed high-value (reports filed or bugs found):**
1. MediaTek MT7932 BT — 3 kernel panic vectors, firmware upload path (REPORTED to Apple; MediaTek disclosure PENDING)
2. Apple Intelligence preferences — 19 injectable prefs, debug keys in prod (REPORTED)
3. AMFI/TXM — sel 16 panic, bricked Air (REPORTED)
4. Siri XPC services — 3 crashable services (REPORTED)
5. networksetup confused deputy — unprivileged DNS/proxy hijack, no auth dialog (REPORT READY — `reports/05-networksetup-dns-proxy-hijack.md`, PENDING SUBMISSION)
6. IOAESAccelerator — unkillable zombie DoS, cross-platform confirmed (REPORT READY — `reports/06-aes-accelerator-kernel-hang.md`, PENDING SUBMISSION)

**Still under investigation:**
7. 240 unprotected UserClients (only 7 entitlements gate any)
8. Exclave framework (16+ services, alwaysonexclavesd unsandboxed root)
9. 21 DART units (all `dart,t8110` — single bug affects all coprocessors)
10. ANE DirectPathClient — any app access on Neo (entitlement-gated on Air)
11. **Rapport daemon + framework** (VS66.10) — trust/discovery substrate for Handoff, iPhone Mirroring, Continuity Camera, AirDrop, AppleWatch unlock, Apple Intelligence cross-device relays. Same daemon on iPhone/iPad/Mac/Watch. New macOS 26 device classes (RPRemoteDisplayDevice, DuetSync caps) unaudited. Cross-platform leverage: HIGH.

**Deprioritized (properly gated or limited impact):**
- ISP (67MB MMIO, but ExclusiveAccess locked by cameracaptured)
- MT7932 WiFi IOCTLs (kernel-gated by IOUserNetworkWLAN despite allow-any-userclient-access)
- Video codecs AVE2+AVD (limited surface from unprivileged)
- AGXDeviceUserClient (complex but well-validated)

## Safety Rules
- Don't brick the machine. Observe first, modify carefully.
- No SIP disabling, kernel modifications, or firmware writes without explicit human approval.
- Flag risky operations and skip them if running autonomously.
- **EXCLUDE /Volumes/r_d from all research.** That drive is for backups only — not a research target.

---
> Source: [dmaynor/apple-vuln-research](https://github.com/dmaynor/apple-vuln-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
