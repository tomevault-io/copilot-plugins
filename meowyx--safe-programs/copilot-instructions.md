## safe-programs

> >


# safe-programs

Three modes in one skill.

- **brainstorm** - design-time thinking partner. Surfaces architectural risks
  before code exists. Produces `design-notes.md` with captured decisions, open
  questions, and framework/testing choices.
- **build** - scaffold a new program (Anchor, Native Rust, or Pinocchio) with
  tests and a filled-out security checklist. Security baked in before the first
  line is written.
- **audit** - fan 8 parallel agents over existing program code, deduplicate,
  gate-validate, produce a findings report.

Same rules under the hood. Design decisions in brainstorm, enforcement in
build, hunting in audit.

---

## Pick the mode first

| Trigger | Mode |
|---|---|
| "thinking about building", "how would I design...", "should I use Anchor or Pinocchio for X", "what should I consider for Y", "brainstorm a solana program", "talk me through designing..." | **brainstorm** |
| "write a solana program", "scaffold", "build an anchor program", "create a native rust solana program", "help me write a program that does X on solana" | **build** |
| "audit this", "review for security", "check this program", "find the bugs", `/safe-programs`, `/safe-programs --deep`, `/safe-programs <file>` | **audit** |

If the ask is ambiguous, ask which one. Don't default.

The natural flow is **brainstorm → build → audit**. Brainstorm commits design
decisions, build scaffolds against them, audit hunts for what slipped through.
A user can enter at any mode.

---

# Brainstorm mode

Design-time thinking partner. Use this when the user is still figuring out what
to build, which framework to use, or whether a design will hold up. Surfaces the
architectural risks before code exists - the decisions that can't be patched
later by adding a line of code.

Produces a `design-notes.md` capturing decisions, alternatives, and open
questions. When the user is ready to scaffold, hand off to build mode with
those decisions pre-filled.

## Step 1 - understand the shape

Ask in one message, not a form. Just enough to know which risks apply:

- What does it do? (functionality, user flow)
- Scale - single user, multi-user, TVL potential
- Custody - holds SOL, holds tokens, holds NFTs, or no custody
- External integrations - CPIs to other protocols, oracles, token standards
- Time dynamics - time-locks, epochs, vesting, auctions, deadlines
- Constraints the user already knows about (team familiarity, audit budget, CU targets)

If the user's description already covers most of this, proceed and note any
assumptions.

## Step 2 - load references

Read `references/shared-base.md` fully. Brainstorm mode leans on the
architectural sections - decisions that are hard or impossible to undo post-deploy:

- §5.8 Defense-in-depth (global vault vs user-specific PDAs)
- §21 Reward accounting (especially §21.4 dead share price, §21.5 inflation attack, §21.7 reward source solvency)
- §22 Vault withdrawal paths
- §23 Token-2022 extension validation at init
- §24 Admin key rotation
- §25 BPF stack frame
- §26 State machine & lifecycle integrity
- §28 Bonding curve / AMM
- §29 Permissionless initialization
- §31.4 Treasury sweepability

If the user is leaning toward a specific framework, also load the matching
framework file (`references/anchor.md`, `references/native-rust.md`,
`references/pinocchio.md`). Don't load `references/litesvm.md` or the audit
references - those come later.

## Step 3 - classify risk tier and architectural category

Risk tier (same scale as build mode):

| Level | Criteria |
|---|---|
| 🟢 Low | no custody, no CPI, single user, read-heavy |
| 🟡 Medium | token transfers, basic CPI, multi-user state, PDAs |
| 🔴 Critical | vaults, multi-CPI chains, admin keys, large TVL |

Architectural categories (a program can hit multiple):

| Category | Maps to |
|---|---|
| Share-based pool (stX / totalStaked) | §21.4, §21.5 |
| Reward / yield system | §21 (whole section) |
| PDA-controlled token vault | §22, §31.4 |
| Multi-CPI chain | §5 (especially §5.8 defense-in-depth) |
| Admin key / privileged control | §24, §29 |
| Token-2022 mint consumption | §7, §23 |
| Bonding curve / AMM | §28 |
| Time-gated logic | §16, §26.6 |
| Permissionless creation | §29 |
| Large account contexts | §25 |

State the tier and matching categories back to the user.

## Step 4 - walk through the architectural risks

For each category that applies, surface the risks that are **architectural** -
can't be patched later without a migration. Frame each as a decision the user
needs to make, not a lecture.

Examples of the shape this should take:

- **Share-based pool:** dead share price requires the exchange-rate update path exists at design time (§21.4). Inflation attack requires dead-shares / min-deposit / virtual balances (§21.5) - pick one now, rewriting later means a new pool.
- **Global vault vs user-specific PDAs:** this is the blast radius of a CPI exploit (§5.8). User-specific PDAs contain a breach to one user. Global vault risks everything. Retrofit is a migration.
- **Admin key model:** single immutable key vs two-step rotation (§24.2) vs multisig vs timelocked multisig. Each has different recovery paths. Starting with single-key then migrating is painful.
- **Reward source:** dedicated rewards vault vs same vault as principal (§21.7). Principal-funded rewards are structural insolvency from day one.
- **Token-2022 policy:** extension allowlist at init (§23) vs reject Token-2022 entirely. Allowlist after deploy requires migration.
- **Permissionless init:** if anyone can call `initialize`, identity must be decoupled from admin privilege or use two-step acceptance (§29.1).

Ask the user to commit. "You have three options here - which one makes sense for this protocol?"

## Step 5 - framework tradeoffs for this use case

Not a generic Anchor-vs-Native-vs-Pinocchio comparison. Specific to what they're
building:

- **High-throughput DEX / orderbook** → Pinocchio worth the unaudited-framework risk for CU savings
- **Critical TVL protocol (lending, large vault)** → Anchor (more auditor familiarity, larger security review pool)
- **Learning / exploration / MVP** → Anchor (faster iteration, more docs)
- **Simple program, tight CU budget** → Pinocchio
- **Token program with custom constraints** → Native Rust (full control, no framework layer)
- **Team unfamiliar with low-level Rust** → Anchor

If the user's leaning doesn't match the fit, say so explicitly. "You're leaning
Pinocchio but this is a 🔴 critical TVL protocol - unaudited framework is a
bigger risk here than the CU savings."

## Step 6 - design-time questions checklist

Before scaffolding, the user should have answers for the questions that apply
to their program. List only the ones that matter - don't ask every question for
a simple counter.

- Who can initialize? Deploy-time only? First-caller with a guard? Two-step accept?
- Is the admin key rotatable? Single key, multisig, or timelocked?
- What's the upgrade policy? Immutable after audit? Timelocked authority? Specific multisig?
- Where does yield come from? (reward programs) Separate rewards vault? External source?
- What happens on emergency? Pause switch? Guardian? Always-available withdrawal path?
- Token standard scope - SPL only, Token-2022 with extension allowlist, or reject Token-2022?
- User isolation - shared PDAs or per-user PDAs?
- Time units - slots or seconds? Consistent across every time-gated handler?
- Reward / fee source - tracked in accounting fields, or implied from raw balances?

## Step 7 - produce design-notes.md

Write `design-notes.md` next to where the code will live. Required sections:

- **What It Does** - one paragraph
- **Risk Level** - 🟢 / 🟡 / 🔴 with one-line justification
- **Architectural Categories** - bulleted list with section references to shared-base
- **Key Design Decisions** - for each: what was decided, what was chosen, alternatives considered, reason, relevant shared-base sections
- **Open Questions** - checkbox list of decisions not yet made
- **Framework Choice** - Anchor / Native Rust / Pinocchio, with the reasoning from Step 5
- **Testing Approach** - LiteSVM / framework default, with one-line reasoning
- **Known Risks Flagged at Design Time** - risks the user is accepting knowingly

Every decision recorded should be traceable back to a shared-base section, even
if the user chose the lower-safety option - the decision being explicit is what
matters.

## Step 8 - offer to hand off to build mode

When the design decisions are captured and the user is ready, ask:

> "Ready to scaffold? I can switch to build mode with these decisions pre-filled - framework, testing, risk tier, and `design-notes.md` as context."

If yes: switch to build mode starting at **Step 5 (gather requirements)** since
framework, testing, and risk level are already decided. Reference
`design-notes.md` when producing `security-checklist.md` at the end - every
decision should appear in the checklist with the same reasoning.

If no (still thinking): leave `design-notes.md` as-is. The user can come back.

## Notes

- **Brainstorm is iterative.** The user may not have all answers in one session. Update `design-notes.md` across multiple sessions as decisions firm up.
- **Don't force a framework choice too early.** If the user is unsure, capture it in Open Questions.
- **Simple programs (counter, registry):** brainstorm mode is overkill. Suggest jumping to build mode directly.
- **Pure research questions** ("what are the risks of X?"): answer them, but skip `design-notes.md` unless the user is moving toward building.

---

# Build mode

## Step 1 - ask the framework

If the user hasn't said, ask exactly this (nothing else):

> "Anchor, Native Rust, or Pinocchio?"

Wait for the answer.

Pinocchio is Anza's zero-copy, zero-dependency framework. 88-95% CU reduction vs
Anchor. Best for high-throughput programs (DEXs, orderbooks, vaults). Unaudited,
so flag that in the checklist for 🔴 critical programs.

## Step 2 - ask about testing

Right after framework is picked, ask:

> "LiteSVM for tests (fast, in-process, no validator), or the framework default?"

| Option | Best for |
|---|---|
| **LiteSVM** | fast unit/integration, CI, time-lock testing, CU profiling, devnet account injection |
| **Framework default** | Anchor: TypeScript with `@coral-xyz/anchor`. Native/Pinocchio: `solana-program-test` |

Wait for the answer before continuing.

## Step 3 - load references

Once both answers are in, read these before writing any code:

1. **Always:** `references/shared-base.md`. Every rule here applies to every
   Solana program. Sections 1-20 cover foundational security. Sections 21-31
   cover vulnerability-derived rules from real protocol audits (reward
   accounting, vault architecture, Token-2022 extension validation, admin key
   rotation, BPF stack frame limits, lifecycle state machine integrity,
   slippage and fee ordering, bonding-curve AMM safety, initialization and
   namespace capture, config management, withdraw and drain safety, treasury
   sweepability).

2. **Then the framework file:**
   - Anchor → `references/anchor.md`
   - Native Rust → `references/native-rust.md`
   - Pinocchio → `references/pinocchio.md`

3. **If LiteSVM:** `references/litesvm.md`.

4. **Check `examples/`** for a matching reference program. If one exists, read
   it as a quality benchmark before writing.

Read every line. Don't skim. These are the rules.

## Step 4 - assess risk

Classify before gathering requirements. Determines how thorough the security
comments and "Known Limitations" section need to be.

| Level | Criteria | Examples |
|---|---|---|
| 🟢 Low | no custody, no CPI, single user, read-heavy | counter, registry, simple config |
| 🟡 Medium | token transfers, basic CPI, multi-user state, PDAs | staking, voting, simple escrow |
| 🔴 Critical | vaults, multi-CPI chains, admin keys, large TVL | AMM, lending, NFT launchpad, bridges |

State the level at the top of `security-checklist.md`. For 🔴 critical: add a
"High-Risk Decisions" section and flag every admin key, upgrade authority, and
irreversible state transition.

## Step 5 - gather requirements

In one message, ask for anything not already given:

- program name
- what it does (brief)
- accounts it needs
- instructions
- access control - who calls what? any admin roles?
- token standard - SPL, Token-2022, or none?
- external programs called (Metaplex, another protocol, etc.)

If the user already covered most of this, proceed and note assumptions.

## Step 6 - write the program

### 6a. Security pre-check (internal)

Run through shared-base.md and the framework file. Flag which rules apply to
this program's design. Note inherent risks in the design itself.

### 6b. Full project scaffold

Not just `lib.rs`. The whole thing, ready to build.

**Anchor:**
```
<program-name>/
├── Anchor.toml
├── Cargo.toml
├── programs/<program-name>/
│   ├── Cargo.toml
│   └── src/lib.rs
└── tests/
    └── <program-name>.ts           # framework default
    └── <program-name>_tests.rs     # LiteSVM
```

**Native Rust / Pinocchio:**
```
<program-name>/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── instruction.rs
    ├── processor.rs
    ├── state.rs
    └── error.rs
tests/
    └── <program-name>_tests.rs     # LiteSVM
```

### 6c. The program code

- Compiles without warnings
- Every account validated (ownership, type, signer, writable as applicable)
- No unchecked math on any financial value
- PDAs with canonical bumps stored and reused
- No logic after CPI that reads stale state
- Descriptive custom errors per program
- Inline security comments on every non-obvious decision

Header block at the top of `lib.rs`:

```rust
// ============================================================
// Program:    <ProgramName>
// Framework:  <Native Rust | Anchor | Pinocchio>
// Testing:    <LiteSVM | solana-program-test | TypeScript/Anchor>
// Risk Level: 🟢 Low | 🟡 Medium | 🔴 Critical
// Security:   See accompanying security-checklist.md
// ============================================================
```

### 6d. Tests

Always ship a test file. The shape depends on Step 2.

**If LiteSVM:**

Follow `references/litesvm.md`.

Required structure:
- `setup()` that loads the `.so`, airdrops SOL, returns `(LiteSVM, Keypair)`
- `send_tx()` helper wrapping message/transaction build, calls `expire_blockhash()` after each send
- PDA derivation helpers matching on-chain seeds exactly

Happy path (implement fully):
- End-to-end success flow with full state assertion (lamports, token balances, account data fields)
- Account closure verification (lamports=0, data.len()=0, owner=system_program)
- CU consumption logged to the `CU_RESULTS` static for `zz_cu_summary`

Security / edge case (implement or stub with `TODO` + why):
- Wrong signer → `assert!(result.is_err())`
- Reinitialization attempt → `assert!(result.is_err())`
- Before-deadline action → `assert!(result.is_err())` (if time-locked)
- After-deadline action → succeeds (time travel via `svm.set_sysvar(&clock)`)
- Over-limit / zero-amount arithmetic → `assert!(result.is_err())`
- Any program-specific edge cases flagged in the checklist

Mandatory closing test:

```rust
#[test]
fn zz_cu_summary() { /* print CU table */ }
```

**If framework default:**

- Anchor: TypeScript via `@coral-xyz/anchor`
- Native Rust: Rust integration tests via `solana-program-test`

Cover the same happy path + security/edge matrix. Mark unimplemented tests with
`TODO` and an explanation comment.

## Step 7 - ship the checklist

Always produce `security-checklist.md`. Every rule applied, every assumption,
every known limitation. Risk level at the top.

## Examples

`examples/` has complete reference programs written to this skill's standard.
Before writing, check if a similar one exists. Use it as a quality benchmark.
Don't copy-paste.

| Example | Framework | Testing | Risk | Demonstrates |
|---|---|---|---|---|
| `examples/nft-whitelist-mint/` | Anchor | TS | 🔴 Critical | MintConfig PDA, per-user WhitelistEntry PDA, double-mint guard, Metaplex CPI with program ID verification, SOL balance check around CPI, Token-2022 compatible mint, safe account close |

Each folder: `lib.rs` + `security-checklist.md`.

## Edge cases and gotchas

- **Simple programs (counter, hello world):** every check still applies.
  Simplicity isn't an excuse for insecure patterns.
- **Inherent design risks (admin key with no timelock, missing upgrade
  authority check):** flag in the checklist under "High-Risk Decisions" or
  "Known Limitations."
- **Token-2022 features (transfer hooks, confidential transfers,
  interest-bearing):** flag as needing extra manual review. Validate extensions
  at `initialize` per shared-base §23.
- **Programs with `remaining_accounts`:** same ownership, signer, and type
  checks as named accounts. Flag in checklist.
- **Upgrade authority:** always note if the program is upgradeable and who
  holds the authority. Recommend timelock or multisig for 🔴 critical.
- **Staking / yield:** shared-base §21. Every reward payout path must update
  `reward_debt`. Retroactive rate application and partial-unstake rounding are
  the top criticals in this category.
- **Share-based pools (stX/totalStaked):** §21.4 (dead share price) and §21.5
  (inflation attack). Architectural, not line-level. Can't be patched easily
  after deploy.
- **Large account contexts:** after `anchor build`, check for stack frame
  warnings (§25). Box the largest fields if the warning appears.
- **AMM / bonding-curve:** §26 (lifecycle state machine), §27 (slippage and
  fee), §28 (AMM safety), §30 (withdraw and drain). Completion-threshold
  capping, post-cap slippage recheck, and terminal-state solvency are the top
  findings.
- **Admin config instructions:** §29. Every write path validates all fields.
  Partial updates use patch semantics so unrelated fields don't get silently
  zeroed.
- **Fee treasury or protocol wallet:** §31.4. Treasury authority must be
  sweepable (valid signer path to move tokens out). Validate the ATA at config
  time, not at withdrawal.
- **User-supplied metadata (name, symbol, URI):** §18 hygiene. Non-empty,
  explicit length bounds, URI scheme allowlists, enforced at instruction
  processing.
- **LiteSVM for RPC-dependent tests:** LiteSVM doesn't support every RPC
  method. If the program needs wallet integration or real validator behavior,
  note in the checklist that those tests belong to `solana-test-validator`
  separately.

---

# Audit mode

8 parallel agents plus an optional 9th protocol agent in deep mode.

## Scope selection

**Exclude:** `tests/`, `test/`, `migrations/`, `scripts/`, `target/`,
`node_modules/`, and files matching `*_test.rs`, `*_tests.rs`, `test_*.rs`,
`tests.rs`, `mod.rs` (unless it contains instruction handlers).

- **Default (no args):** scan all `.rs` files in the program directory with the
  exclude pattern. Use Bash `find`, not Glob.
- **`<filename> ...`:** scan the specified file(s) only.

**Flags:**

- `--file-output` (off by default): also write the report to a markdown file,
  path per `references/report-formatting.md`. Never write the file unless this
  is set.
- `--deep`: also spawn the Solana protocol analysis agent (agent 9, opus model).
  Use for thorough reviews of DeFi protocols. Slower and more costly.

## Orchestration

### Turn 1 - discover

One message, parallel tool calls:

a. Bash `find` for in-scope `.rs` files per scope selection
b. Glob for `**/references/attack-vectors/attack-vectors-1.md`. Two levels up
   from the match is `{refs}`.
c. ToolSearch `select:Agent`
d. Read the local `VERSION` file from the skill directory
e. Bash `mktemp -d /tmp/audit-XXXXXX` → store as `{bundle_dir}`

### Turn 2 - prepare

One message, parallel: read `{refs}/report-formatting.md` and `{refs}/judging.md`.

Then build all bundles in a single Bash command using `cat` (no shell variables,
no heredocs):

1. `{bundle_dir}/source.md` - every in-scope `.rs` file, each preceded by a
   `### path` header and wrapped in a fenced code block.
2. Agent bundles = `source.md` + agent-specific files:

| Bundle | Appended (relative to `{refs}`) |
|---|---|
| `agent-1-bundle.md` | all 5 `attack-vectors/attack-vectors-*.md` + `hacking-agents/vector-scan-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-2-bundle.md` | `hacking-agents/math-precision-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-3-bundle.md` | `hacking-agents/access-control-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-4-bundle.md` | `hacking-agents/economic-security-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-5-bundle.md` | `hacking-agents/execution-trace-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-6-bundle.md` | `hacking-agents/invariant-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-7-bundle.md` | `hacking-agents/periphery-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-8-bundle.md` | `hacking-agents/first-principles-agent.md` + `hacking-agents/shared-rules.md` |

Print line counts for every bundle and `source.md`. Never inline file content
into agent prompts.

### Turn 3 - spawn

One message, all 8 agents as parallel foreground Agent calls. Prompt template
(substitute real values):

```
Your bundle file is {bundle_dir}/agent-N-bundle.md (XXXX lines).
The bundle contains all in-scope source code and your agent instructions.
Read the bundle fully before producing findings.
```

If `--deep`: also spawn agent 9 (protocol analysis) with `model: "opus"`. Agent
9 receives the in-scope `.rs` file paths and the instruction: your reference
directory is `{refs}`. Read `{refs}/hacking-agents/solana-protocol-agent.md`
for full instructions.

### Turn 4 - deduplicate, validate, output

Single pass. Deduplicate all agent results, gate-evaluate, produce the final
report in one turn. No intermediate dedup list - go straight to the report.

1. **Deduplicate.** Parse every FINDING and LEAD from all agents. Group by
   `group_key` (`Program | handler | bug-class`). Exact-match first, then
   merge synonymous bug_class tags on the same program and handler. Keep the
   best version per group. Number sequentially. Annotate `[agents: N]`.

   Check **composite chains**: if finding A's output feeds B's precondition AND
   combined impact is strictly worse than either alone, add "Chain: [A] + [B]"
   at confidence = min(A, B). Most audits have 0-2.

2. **Gate evaluation.** Run every deduplicated finding through the four gates
   in `judging.md`. Don't skip or reorder. Evaluate each finding exactly once.
   Don't revisit after verdict.

   **Single-pass protocol:** evaluate every relevant code path once, in fixed
   order (initialize → deposit/stake → process/swap → withdraw/unstake →
   claim → close). One-line verdict per path: `BLOCKS`, `ALLOWS`, `IRRELEVANT`,
   or `UNCERTAIN`. Commit after all paths. `UNCERTAIN = ALLOWS`.

3. **Lead promotion and rejection guardrails.**
   - Promote LEAD → FINDING (confidence 75) if: complete exploit chain traced
     in source, OR `[agents: 2+]` demoted (not rejected) the same issue.
   - `[agents: 2+]` does NOT override a concrete refutation. Demote to LEAD if
     refutation is uncertain.
   - No deployer-intent reasoning. Evaluate what the code **allows**, not how
     the deployer **might** use it.

4. **Fix verification** (confidence ≥ 80 only): trace the attack with the fix
   applied. Verify no new DoS, CPI failures, or broken invariants. List every
   location if the pattern repeats. If no safe fix exists, omit with a note.

5. **Format and print** per `references/report-formatting.md`. Exclude rejected
   items. If `--file-output`: also write to file.

## Banner

Before doing anything else in audit mode, print this exactly:

```
               __                                                            
   _________ _/ __/__        ____  _________  ____ __________ _____ ___  _____
  / ___/ __ `/ /_/ _ \______/ __ \/ ___/ __ \/ __ `/ ___/ __ `/ __ `__ \/ ___/
 (__  ) /_/ / __/  __/_____/ /_/ / /  / /_/ / /_/ / /  / /_/ / / / / / (__  ) 
/____/\__,_/_/  \___/     / .___/_/   \____/\__, /_/   \__,_/_/ /_/ /_/____/  
                         /_/               /____/                             
```

---
> Source: [meowyx/safe-programs](https://github.com/meowyx/safe-programs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
