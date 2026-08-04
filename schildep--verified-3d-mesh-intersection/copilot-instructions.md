## verified-3d-mesh-intersection

> A Lean 4 project for formally verified CSG / geometric algorithms.

# CSG — verified geometric algorithms

A Lean 4 project for formally verified CSG / geometric algorithms.

## Trust model

Correctness rests on **two things only**: the Lean type checker, and human review
of a *small* specification. It does **not** rest on trusting any AI agent.

A human reviewer of the kernel reads only the **specification files**.

The reviewed surface splits into **three independent parts**. Only the first is
the web application and the kernel its calls; the other two are standalone/legacy
surfaces relegated to `CSG/Legacy/`:

- the **web-application spec** — the files at the **top level of `CSG/`**:
  `DataStructures.lean`, `Def.lean` (ray-based mesh semantics), the mesh-facing
  statement files (`MeshIntersect.lean`, `WellFormedCheck.lean`,
  `MeshIntersectWithPreconditionCheck.lean`, `WellFormedCheckMsg.lean`) and
  `Web.lean` — plus `Example.lean`, optional sanity theorems that give a
  reviewer extra assurance about the semantics but are not needed to specify
  the algorithms;
- the standalone **general-position utility** — `CSG/Legacy/GenPosDef.lean` and
  `CSG/Legacy/GenPosCheck.lean` (the `meshIntersect` theorems carry no
  general-position hypothesis);
- the separate, **legacy chain-level spec** — `CSG/Legacy/ChainDataStructures.lean`
  and `CSG/Legacy/ChainDef.lean` (Feito–Rivero origin-cone winding) with
  `CSG/Legacy/ChainNormalCheck.lean`, `CSG/Legacy/ChainIntersectionAlgorithm.lean`
  and `CSG/Legacy/ChainIntersectionExistence.lean`.

The web-application spec depends on **nothing** under `CSG/Legacy/` — no top-level
`CSG/` spec file nor any runtime implementation imports a Legacy module, and the
mesh runtime never calls the chain or general-position algorithms — so a reviewer
of the application never has to read a Legacy file, and each surface is reviewed
on its own. (Untrusted *proofs* in `CSG/Proof/` may still reuse Legacy machinery;
that never enlarges the reviewed surface or the runtime.)

## File roles

| File | Reviewed by humans? | Mathlib? | Role |
| --- | --- | --- | --- |
| `DataStructures.lean` | yes | no | The mesh data types in the algorithm interface (shared by all reviewed surfaces). |
| `Def.lean` | yes | yes | The ray-based mesh semantics used to state the web-application theorems. |
| web-application statement files | yes | yes | `MeshIntersect.lean`, `WellFormedCheck.lean`, `MeshIntersectWithPreconditionCheck.lean`, `WellFormedCheckMsg.lean` — theorem statements and algorithm signatures at the top level of `CSG/`. The *meaning* (type) of each depends **only** on `DataStructures.lean` and `Def.lean`. |
| `Example.lean` | yes (optional) | yes | Sanity theorems exercising the `Def.lean` semantics (∀/∃ agreement for `solid`, admissible-ray existence, a concrete well-formed cube whose solid is exactly the open unit cube). Pure assurance: nothing in the spec or runtime depends on it. |
| `CSG/Legacy/*.lean` | yes | yes | The **general-position utility** (`GenPosDef.lean`, `GenPosCheck.lean`) and the **legacy chain-level spec** (`ChainDataStructures.lean`, `ChainDef.lean`, `ChainNormalCheck.lean`, `ChainIntersectionAlgorithm.lean`, `ChainIntersectionExistence.lean`) — two *separate* reviewed surfaces the web application does **not** depend on. Namespaces `CSG.Legacy.*`. |
| `CSG/Impl/*.lean` | no | no | Executable definitions of the application (the algorithms). |
| `CSG/Impl/Legacy/*.lean` | no | no | Executable definitions used only by the Legacy surfaces / by proofs (never by the app runtime). Namespaces `CSG.Impl.Legacy.*`. |
| `CSG/Proof/*.lean` | no | yes | Formal proofs of the specifications (app and legacy alike). |
| `Web.lean` | yes | no | WASM entry point: a `String → String` wrapper over the exported algorithms, `@[export]`ed. |

**Repository layout.** The reviewed *application* surface is easy to find: its
specification files (data/semantics + the statement files above) sit directly at
the **top level of `CSG/`**. Everything else lives in subfolders — executable app
code in **`CSG/Impl/`** (namespaces `CSG.Impl.*`), proofs in **`CSG/Proof/`**
(namespaces `CSG.Proof.*`), informal proof-strategy notes in
**`CSG/ProofPlanning/`**, and the standalone/legacy surfaces (general-position +
chain-level, spec and impl) under **`CSG/Legacy/`** and **`CSG/Impl/Legacy/`**
(namespaces `CSG.Legacy.*` / `CSG.Impl.Legacy.*`). So the trust boundary is
visible both in the folder tree and in every fully-qualified name: the trusted
*application* spec is exactly the top level of `CSG/`; `CSG.Legacy.*` is a
separate reviewed surface the app does not depend on; `CSG.Impl.*`/`CSG.Proof.*`
are untrusted.

The specification files delegate their definitions to the `CSG/Impl/` files and
their proofs to the `CSG/Proof/` files. So a reviewer never needs to read a file
under `CSG/Impl/` or `CSG/Proof/`: they are machine-checked, not trusted. The one
exception is `Web.lean`: its `String → String` (de)serialization is trusted glue
with no correctness proof, so it is kept minimal and is reviewed.

## Adding to the specification (for specification agents)

When introducing a new user-facing definition or theorem, keep the trusted surface
minimal and self-contained:

1. **Put each piece in the right file** (see *File roles*):
   - a new data type → `DataStructures.lean` (keep it **Mathlib-free**: `Batteries`/core only);
   - a new semantic definition a statement needs → `Def.lean` (Mathlib allowed);
   - a new theorem statement or algorithm signature → its own statement file at the
     top level of `CSG/`, say `CSG/Foo.lean`.

2. **Statements depend only on `DataStructures.lean` and `Def.lean`.** The *meaning*
   of a statement (its type) must be expressible from those two files alone — never
   from a `CSG/Impl/` or `CSG/Proof/` file. A statement file may *import*
   `CSG.Impl.Foo` / `CSG.Proof.Foo` only to borrow the implementation body or the
   proof term; it must not mention their internals in what it claims. Concretely:
   - algorithm: `def foo (…) : T := Impl.Foo.foo …`, then `theorem foo_spec … := Proof.Foo.foo_spec …`;
   - pure theorem: state it in `CSG/Foo.lean`, prove it by `:= Proof.Foo.foo …`.
   The delegation type-checks by definitional unfolding, so `CSG/Foo.lean` stays
   `sorry`-free while the real work can later be filled in to the untrusted
   `CSG/Impl/` / `CSG/Proof/` files.

3. **`sorry` first, prove later.** As the *first* step, create the matching
   `CSG/Impl/Foo.lean` (`def … := sorry`) and `CSG/Proof/Foo.lean` (`… := by sorry`)
   so the whole project compiles and the **specification can be reviewed on its own**,
   before any implementation or proof exists. Prover agents fill the `sorry`s
   afterwards without touching a spec file.

4. **Nothing extra in the reviewed files.** Put *only* what the statement itself
   requires into the user-facing files — no helper lemmas, no intermediate
   constructions, no implementation detail. Everything auxiliary belongs in
   `CSG/Impl/` / `CSG/Proof/`. The smaller the reviewed surface, the stronger the guarantee.

5. **Make the statement provable and non-trivial before writing it.** Prefer
   *extensional* specifications that pin down the intended observable behaviour (e.g.
   the result a construction must produce), so that only a genuine construction can
   satisfy them — regardless of how it is built — yet nothing degenerate can. If you
   suspect a formulation is unprovable, vacuous, or trivially satisfiable, surface it
   during planning, before writing the spec or the proofs. After writing the specification
   you should always run a review agent that critically searches for any such
   issues. Give it a focused charter (the specific files/statements changed).

## Hard rule for prover agents

- **Prover and implementation agents never modify the specification files**
  (`DataStructures.lean`, `Def.lean`, and the statement files) when filling
  in implementations or proofs. Changing a statement would invalidate the
  human review.
- Prover agents work **only** inside the `CSG/Impl/` and `CSG/Proof/` files,
  which start as `sorry` and must be completed there.
- Prover agents should never add any `.lean` file outside `CSG/Impl/` or `CSG/Proof/`.
- If a prover agent has strong evidence that a statement in a specification file is
  unprovable, even though this was critically reviewed earlier, it should flag this
  issue rather than changing the specification. Files under `CSG/Impl/` and
  `CSG/Proof/` on the other hand can be freely changed if not stated otherwise.
- Prover agents should strive for a clean architecture, code reuse etc. within the
  `CSG/Impl/` and `CSG/Proof/` files.
- Prover agents should always write an informal but mathematically rigorous proof
  in a markdown file in `CSG/ProofPlanning/` before writing the formal proof. This
  file should not be concerned with lean but informally prove the theorem in detail.
  After the file is written the prover agent should start a review agent that
  reviews the informal proof and that it tracks with statements that needs to be proven
  on the lean side. Give the reviewer a focused charter (the specific claims).
  It should iterate on the file until no issues are found and
  until it is clear that this formalizes well. Only then should the prover agents start
  writing the lean proof. The formal proof should then follow the informal proof. In case
  the prover agents needs to change the proof strategy or need a more detailed strategy,
  they immediately need to go back to the informal proof markdown and adapt the proof
  strategy there, because it is important to first plan out proofs as informal proofs
  before formalization, since this leads to the best results. It is important that the
  markdown file tracks the current proof strategy at every point in time. Note that in
  case the proof strategy is adapted or details are added this again must be reviewed (one
  focused review agent per round) for the above mentioned issues and iterate on that until
  no issues are found. And make sure the reviewer also checks that the reported missing
  details were adequately provided.

## Subagent worktrees

If you want to run parallel subagents that can change and build lean code independently,
create (or reuse if already present) git worktrees whose build folders symlink to the
Mathlib build.

## Proof iteration: use the lean-lsp MCP server, not `lake build`

The project configures a `lean-lsp` MCP server (`.mcp.json`); its tools are named
`mcp__lean-lsp__*` (in agent contexts load them via ToolSearch). A full or per-module
`lake build` as an edit-check probe re-pays Mathlib import loading plus whole-file
re-elaboration on every call. The LSP keeps the file loaded and re-elaborates
incrementally, so:

- **Iterate with the LSP tools**: `lean_goal` (proof state at a position — the main
  tool), `lean_diagnostic_messages` (errors/warnings for the edited file),
  `lean_multi_attempt` (try several tactics at a position without editing),
  `lean_hover_info`/`lean_completions`, and the Mathlib search tools
  (`lean_leansearch`, `lean_loogle`, `lean_state_search`, `lean_hammer_premise`).
- **Run `lake build` only** (a) as a targeted `lake build CSG.<Module>` once a lemma
  family is believed closed — before every commit, since downstream modules need the
  `.olean`s and the tree must stay green — and (b) after import changes (the LSP may
  need `lean_build`/restart to pick those up).
- If the lean-lsp tools are not available in the current session (server connects at
  session start), fall back to *targeted* `lake build CSG.<Module>` probes — never
  repeated full `lake build`s as an iteration loop.

**The cold build is the only completion gate.** The LSP's warm incremental state (and a
subagent's self-report) can show "green" while a **cold `lake build` fails** — e.g. on
elaboration timeouts a fresh process hits but the cached one never re-pays. So before
committing, run a cold `lake build CSG.<Module>` *yourself* (`rm` that module's `.olean`)
plus `#print axioms` (expect `[propext, Classical.choice, Quot.sound]`, no `sorryAx`). Fix
timeouts **structurally, not with `maxHeartbeats`**.

## WebAssembly runtime layer (must stay Mathlib-free)

The intersection algorithm implementations compile to a **minimal WASM module** that
must **not depend on Mathlib**. The runtime closure is `Web.lean → CSG/Impl/*.lean → DataStructures.lean`,
and every file in it imports only `Batteries`/core — never `Mathlib`. So:

- **`DataStructures.lean` and every `CSG/Impl/` file (and `Web.lean`) must never
  import Mathlib.** Use `Rat`/`Int` and `Batteries`, not `ℚ`/`ℤ` notation or
  Mathlib lemmas. A single stray Mathlib import pulls all of Mathlib into the WASM
  closure and breaks the size budget.
- The Mathlib-using files (`Def.lean`, the statement files, `CSG/Proof/*.lean`) are
  **not** part of the WASM closure, so they may use Mathlib freely. `Def.lean`
  gives the data types their geometric semantics (e.g. as a Mathlib vector space)
  without the runtime layer ever importing Mathlib.

Build the web app (requires the Lean v4.15.0 toolchain, emscripten, zstd, and
node/npm; `wasm-opt` optional):

`build_web_demo.sh` builds only the deployable web demo, not the Lean
development (that is `lake build`, above):

```
LEAN_CC=./emcc-wasm.sh LEAN_SYSROOT=$PWD/.wasm-toolchain lake build wasmClosure
./build_web_demo.sh   # full pipeline: closure → link → wasm-opt → size gate
                      #   → web/public/lean_app.{js,wasm} (gitignored)
                      #   → npm run build → docs/ (committed; GitHub Pages)
```

`docs/` is the only committed artifact copy — GitHub Pages serves it from
`main`, so deploying is `./build_web_demo.sh` + commit + push. For UI-only
changes, `cd web && npm run build` reuses the wasm already in `web/public/`
without re-running the Lean stage.

`build_web_demo.sh` fails if the gzipped wasm exceeds `MAX_WASM_GZIP_BYTES`; a
sudden jump almost always means a Mathlib dependency leaked into the runtime
closure.

## Verify

```
lake build
```

When proofs are complete, confirm no hidden assumptions remain by printing the
axioms of each top-level specification theorem:

```lean
#print axioms <SpecTheorem>       -- expect: [propext, Classical.choice, Quot.sound]
```

---
> Source: [schildep/verified-3d-mesh-intersection](https://github.com/schildep/verified-3d-mesh-intersection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
