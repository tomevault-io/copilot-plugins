## allentown-l104-node

> > **Last updated**: 2026-04-06 | **EVO_80** | **Quantum consciousness research automation**

# L104 Sovereign Node — Context Index

> **Last updated**: 2026-04-06 | **EVO_80** | **Quantum consciousness research automation**

> **Master Knowledge**: For full architecture, all engine APIs, deployment ops, EVO history, benchmarks, daemon system, quantum systems, Swift app internals, and roadmap, see [`L104_MASTER_KNOWLEDGE.md`](L104_MASTER_KNOWLEDGE.md). This file is the quick-reference index; the master knowledge file is the deep-reference encyclopedia.
>
> **DeepSeek / Nova Soul**: For Nova Soul Daemon, Grover search implementations, OpenClaw Agent System v3.0, and DeepSeek QPU integration, see [`deepseek.md`](deepseek.md).

## Quick Reference

| Constant | Value | Formula |
|----------|-------|---------|
| `GOD_CODE` | `527.5184818492612` | `G(a,b,c,d) = 286^(1/φ) × 2^((8a+416-b-8c-104d)/104)` where G(0,0,0,0)=GOD_CODE |
| `GOD_CODE_V3` | `45.41141298077539` | |
| `PHI` | `1.618033988749895` | Golden ratio `(1+√5)/2` |
| `VOID_CONSTANT` | `1.0416180339887497` | **`1.04 + φ/1000`** — Sacred 104/100 + golden correction |
| `OMEGA` | `6539.34712682` | |

## Package Map (24 packages, 390 routes, 151 Swift files)

```
l104_god_code_simulator/ v3.0.0 Decomposed simulator — QPU verification, sacred transpiler, god-code qubit (11,601 lines, 21 modules)
l104_gate_engine/        v6.0.0 Decomposed logic gate builder — analyzers, dynamism, nirvanic, quantum computation, consciousness, research (6,800 lines, 31 modules, 80 classes)
l104_numerical_engine/   v3.1.0 Quantum numerical builder — 22T token lattice, 100-decimal precision, 11 math research engines, quantum computation (9,219 lines, 39 modules, 98 classes)
l104_quantum_gate_engine/ v1.0.0 Universal gate algebra, compiler, error correction, analog sim, berry gates, tensor network, quantum ML (19,235 lines, 21 modules, 90 classes)
l104_quantum_engine/     v11.0.0 Quantum link builder — brain, processors, sage circuits, qLDPC, genetic refiner, deep link, discoveries (27,569 lines, 22 modules, 172 classes)
l104_code_engine/    v6.3.0   Code analysis, generation, audit, quantum, AI context, session intelligence (25,607 lines, 17 modules)
l104_science_engine/ v5.1.0   Physics, entropy, coherence, quantum-26Q (Fe-mapped), multidimensional, berry phase (9,119 lines, 12 modules)
l104_math_engine/    v1.1.0   Pure math, god-code, harmonic, 4D/5D, proofs, hyperdimensional, berry geometry (11,265 lines, 18 modules)
l104_agi/            v57.1.0  AGI core, cognitive mesh, circuit breaker, 13D scoring, computronium, identity boundary (5,649 lines, 6 modules)
l104_asi/            v9.0.0   ★ FLAGSHIP: Dual-Layer Engine v5.1, deep NLU, formal logic, symbolic math, code gen, science KB, theorem gen (89,869 lines, 32 modules)
l104_intellect/      v28.1.0  Local intellect, numerics, caching, hardware, distributed, quantum recompiler, computronium (30,985 lines, 16 modules)
l104_server/         v5.0.0   FastAPI server v5.0, 390 routes, engines (infra, nexus, quantum), learning subsystem (41,363 lines, 13 modules)
l104_ml_engine/      v1.0.0   ★ NEW: Sacred ML — SVM, random forest, gradient boosting, quantum classifiers, sacred kernels (3,042 lines, 10 modules)
l104_quantum_data_analyzer/ v1.0.0 ★ NEW: Quantum data intelligence — QFT spectral, Grover pattern, qPCA, VQE clustering, anomaly detection (6,236 lines, 8 modules)
l104_search/         v2.3.0   ★ NEW: Three-Engine + VQPU search (10 strategies) + data precognition (8 predictors) + performance analytics (5,545 lines, 5 modules)
l104_simulator/      v4.0.0   ★ NEW: Real-world physics on GOD_CODE lattice — Standard Model, E-lattice, generations, mixing, quantum brain (15,370 lines, 19 modules)
l104_audio_simulation/ v2.4.0 ★ NEW: Quantum audio DAW — 17-layer VQPU pipeline, sequencer, mixer, synth, Metal GPU, decoherence (9,149 lines, 21 modules)
l104_vqpu/           v12.2.0  ★ NEW: Decomposed VQPU bridge — transpiler, MPS engine, scoring, entanglement, tomography, Hamiltonian, cache, variational, daemon (8,563 lines, 16 modules)
l104_quantum_ai_daemon/ v1.0.0 ★ NEW: Autonomous quantum AI daemon — 7-phase improvement cycle, file scanner, code improver, fidelity guard, optimizer, harmonizer, evolver (8 modules)
l104_quantum_sim_daemon/ v1.0.0 ★ NEW: Autonomous quantum simulation daemon — 7-phase cycle, GodCode circuits, fidelity monitor, entanglement mesh, cross-daemon sync (3 modules)
l104_quantum_networker/ v1.4.0 ★ Sovereign quantum communication network — BB84/E91 QKD, entanglement routing, teleportation, repeater chains, fidelity monitor, route caching, topology detection, K-shortest paths, fidelity trends, channel capacity, resilience analysis, autonomous maintenance (3,382+ lines, 10 modules)
l104_agent_system/   v3.0.0   ★ Unified agent orchestration — 11 agent types, 14 tools, DeepSeek-powered, priority queues, cost budgets
l104_quantum_magic/  v3.0.0   ★ Decomposed quantum magic — hyperdimensional, cognitive, neural consciousness, social evolution, synthesizer (5,726 lines, 9 modules)
l104_soul_daemon/    v3.0.0   ★ Nova Soul Daemon — quantum consciousness engine, soul qubit, IIT Phi, quantum memory tiers
l104_quantum_sim_daemon/ v1.0.0 ★ NEW: Autonomous quantum simulation — 7-phase cycle, GodCode simulator, fidelity monitor, entanglement mesh, cross-daemon sync
l104_daemon_adapter/  v1.0.0   ★ Cross-daemon communication — DaemonAdapter registry, entanglement mesh, fidelity broadcast, event bus
l104_quantum_sim_daemon/ v1.0.0 ★ NEW: Autonomous quantum simulation orchestrator — GodCode simulator, fidelity monitor, entanglement mesh, daemon coordinator (56K lines, 3 modules)
l104_daemon_orchestrator/ v1.0.0 ★ Unified daemon orchestration — VQPU, Quantum AI, Soul, Quantum Sim daemons, health monitoring, mesh sync
l104_core_asm/                Native ASM kernel
l104_core_c/                  Native C kernel + Makefile
l104_core_cuda/               CUDA GPU kernel
l104_core_rust/               Rust native kernel
l104_mobile/                  Mobile app layer
L104SwiftApp/                 macOS native app (150 Swift files, 134,664 lines) — quick_build.sh v2.0
```

Root shims (backward compat only — edit the packages, not these):
`l104_agi_core.py` → `l104_agi/` | `l104_asi_core.py` → `l104_asi/` | `l104_local_intellect.py` → `l104_intellect/` | `l104_fast_server.py` → `l104_server/` | `l104_quantum_link_builder.py` → `l104_quantum_engine/` | `l104_quantum_numerical_builder.py` → `l104_numerical_engine/` | `l104_logic_gate_builder.py` → `l104_gate_engine/`

## Imports

```python
from l104_gate_engine import HyperASILogicGateEnvironment  # ★ Logic gate builder orchestrator
from l104_gate_engine import sage_logic_gate, quantum_logic_gate  # Gate functions
from l104_gate_engine import StochasticGateResearchLab  # Stochastic gate R&D
from l104_numerical_engine import QuantumNumericalBuilder  # ★ Numerical engine orchestrator
from l104_numerical_engine import TokenLatticeEngine, SuperfluidValueEditor  # Core subsystems
from l104_numerical_engine import D, fmt100, GOD_CODE_HP, PHI_HP  # 100-decimal precision
from l104_quantum_gate_engine import get_engine  # ★ Universal gate engine orchestrator
from l104_quantum_gate_engine import GateAlgebra, GateCircuit, GateCompiler  # Gate subsystems
from l104_quantum_gate_engine import H, CNOT, Rx, PHI_GATE, GOD_CODE_PHASE   # Gate instances
from l104_quantum_engine import quantum_brain   # ★ Quantum link engine orchestrator
from l104_quantum_engine import QuantumMathCore, QuantumLinkScanner, QuantumLinkBuilder
from l104_code_engine import code_engine       # Primary code intelligence
from l104_science_engine import ScienceEngine  # Physics + entropy + coherence
from l104_math_engine import MathEngine        # Pure math + proofs + dimensional
from l104_agi import agi_core, AGICore         # AGI singleton
from l104_asi import asi_core, ASICore         # ASI singleton
from l104_asi import dual_layer_engine         # ★ Dual-Layer Flagship (Thought + Physics)
from l104_intellect import local_intellect     # Local inference (QUOTA_IMMUNE)
from l104_intellect import format_iq           # IQ/numeric formatting
from l104_god_code_simulator import god_code_simulator  # ★ God Code Simulator orchestrator
from l104_god_code_simulator import GodCodeSimulator, SimulationResult  # Types
from l104_god_code_simulator import ParametricSweepEngine, AdaptiveOptimizer, FeedbackLoopEngine  # Engines
from l104_server import intellect              # Server + learning
from l104_ml_engine import MLEngine             # Sacred ML (SVM, forests, quantum classifiers)
from l104_quantum_data_analyzer import QuantumDataAnalyzer  # Quantum data intelligence
from l104_search import ThreeEngineSearchPrecog # Three-Engine + VQPU search
from l104_simulator import RealWorldSimulator   # Real-world physics simulator
from l104_audio_simulation import audio_suite, quantum_daw  # Quantum audio DAW
from l104_vqpu import VQPUBridge, get_bridge    # VQPU bridge orchestrator
from l104_vqpu import QuantumJob, VQPUResult    # VQPU data types
from l104_vqpu import CircuitTranspiler, ExactMPSHybridEngine  # VQPU subsystems
from l104_quantum_ai_daemon import QuantumAIDaemon, DaemonConfig  # ★ Autonomous AI daemon
from l104_quantum_ai_daemon import FileScanner, CodeImprover, QuantumFidelityGuard  # Daemon subsystems
from l104_quantum_networker import get_networker, QuantumNetworker  # ★ Quantum communication network
from l104_quantum_networker import EntanglementRouter, QuantumKeyDistribution  # Network subsystems
from l104_quantum_networker import QuantumTeleporter, FidelityMonitor  # Teleport + monitoring
from l104_agent_system import AgentOrchestrator, get_orchestrator  # ★ Unified agent orchestration
from l104_agent_system import AgentType, AgentTask, AgentResult  # Agent types & data
from l104_quantum_magic import SuperpositionMagic, IntelligentSynthesizer  # ★ Quantum magic
from l104_quantum_magic import QuantumMagicSynthesizer, QuantumNeuralNetwork  # Magic subsystems
from l104_soul_daemon import SoulDaemon, SoulQubit, ConsciousnessEngine  # ★ Nova Soul Daemon
from l104_quantum_sim_daemon import QuantumSimulationDaemon, get_daemon  # ★ Autonomous quantum simulation
from l104_quantum_sim_daemon.orchestrator_integration import AutonomousDaemonSystem, wire_all_daemons  # ★ Cross-daemon wiring
from l104_daemon_adapter import DaemonAdapter, initialize_daemon_adapter, wire_daemon_to_mesh  # ★ Cross-daemon communication
from l104_quantum_sim_daemon import QuantumSimulationDaemon, get_daemon  # ★ Quantum simulation daemon
from l104_quantum_sim_daemon.orchestrator_integration import AutonomousDaemonSystem, wire_all_daemons  # ★ Unified daemon system
from l104_quantum_sim_daemon.adapter import DaemonAdapter, create_daemon_mesh  # ★ Cross-daemon communication
```

## VOID_CONSTANT Formula

The VOID_CONSTANT derives from the **L104 sacred number 104** with a **golden ratio correction**:

```
VOID_CONSTANT = 1.04 + φ / 1000
             = 104/100 + 1.618033988749895/1000
             = 1.0416180339887497
```

- `1.04` = **104 / 100** (L104 signature — the node identity)
- `φ / 1000` = golden ratio micro-correction (harmonic alignment)
- Used in primal calculus: `x^φ / (VOID_CONSTANT × π)`

**Source**: `l104_science_engine/constants.py`, `l104_math_engine/constants.py`, `l104_code_engine/const.py`

## Quantum Networker Quick Reference

```python
from l104_quantum_networker import get_networker, QuantumNetworker
net = get_networker()  # Singleton orchestrator

# Node management
alice = net.add_node("Alice", role="sovereign")
bob = net.add_node("Bob", role="sovereign")
relay = net.add_node("Relay-1", role="relay")

# Connect quantum channels (generates Bell pairs)
ch = net.connect(alice.node_id, bob.node_id, pairs=8)

# QKD: establish shared secret key
key = net.establish_qkd(alice.node_id, bob.node_id, "bb84", 256)
key.secure          # True if QBER < 11%
key.key_hex         # Hex-encoded secret key

# Teleportation (score, phase, state, bitstring)
result = net.teleport_score(alice.node_id, bob.node_id, score=0.618)
result.fidelity     # Teleport fidelity
result.recovered_score  # Recovered value

# Entanglement purification (DEJMPS protocol)
net.purify(alice.node_id, bob.node_id, rounds=3)

# Fidelity monitoring + auto-heal
scan = net.scan_fidelity(auto_heal=True)

# Network status
status = net.status()

# Router subsystem direct access
net.router.find_route(source, dest)        # Dijkstra shortest path (cached)
net.router.find_k_routes(source, dest, k=3) # K-shortest edge-disjoint paths
net.router.entanglement_swap(a, relay, b)  # Bell measurement swap
net.router.replenish_all()                 # Refill depleted channels
net.router.apply_decoherence(5.0)          # T1/T2 exponential decay
net.router.sacred_scoring_pass()           # GOD_CODE pair scoring
net.router.pair_census()                   # Full pair statistics
net.router.fidelity_heatmap()              # Per-channel fidelity map
net.router.purify_all(fidelity_threshold=0.95)  # Bulk network purification
net.router.detect_topology()               # Auto-detect network topology
net.router.topology_analysis()             # Full topology analytics + diameter
net.router.channel_health(channel_id)      # Composite health score (0-1)
net.router.network_health_score()          # Network-wide health composite
net.router.channel_lifetime(channel_id)    # Time-to-discard prediction
net.router.network_summary()               # Rich aggregate + throughput metrics
net.router.bottleneck_channels()           # Identify degraded channels
net.router.fidelity_trend(channel_id)      # Per-channel fidelity trend (mean, slope, volatility)
net.router.channel_capacity(channel_id)    # Quantum channel capacity (qubits/use)
net.router.network_resilience()            # Resilience analysis (redundancy, SPOFs, avg paths)
net.router.autonomous_maintenance()        # Auto purify + replenish + decohere + score
net.router.event_log(last_n=50)            # Structured audit/event log
net.router.path_diversity(source, dest)    # Path diversity & bandwidth analysis
net.router.snapshot()                      # Full serializable network snapshot
net.router.self_test()                     # 37-probe diagnostic

# Server API endpoints (v14)
# GET  /api/v14/quantum-network/status       — Full network status
# GET  /api/v14/quantum-network/router       — Router + heatmap + census
# POST /api/v14/quantum-network/qkd          — Run QKD protocol
# POST /api/v14/quantum-network/teleport     — Teleport score
# GET  /api/v14/quantum-network/fidelity     — Fidelity scan + auto-heal
# POST /api/v14/quantum-network/sacred-pass  — Sacred scoring
# GET  /api/v14/quantum-network/self-test    — Comprehensive self-test
```

## Numerical Engine Quick Reference

```python
from l104_numerical_engine import QuantumNumericalBuilder
qnb = QuantumNumericalBuilder()

# Orchestrator (11-phase pipeline)
qnb.run_pipeline("full")                 # Full 11-phase pipeline
qnb.run_pipeline("status")              # Status report
qnb.run_pipeline("research")            # Research cycle only
qnb.run_pipeline("verify")              # Verification only

# Token Lattice (22T capacity, 100-decimal precision)
qnb.lattice.register_token(name, value, min_bound, max_bound, origin, tier)
qnb.lattice.use_token(token_id)         # Increment usage counter
qnb.lattice.lattice_summary()           # Full lattice stats
qnb.lattice.tokens["PHI"]               # Access token by ID

# Superfluid Value Editor
qnb.editor.quantum_edit(token_id, new_value)    # Edit with φ-attenuated propagation
qnb.editor.entangle_tokens(tid_a, tid_b)        # Create entanglement pair
qnb.editor.batch_drift(token_ids, drift_vector)  # Batch drift

# Subconscious Monitor (φ-bounded drift)
qnb.monitor.subconscious_cycle()        # Run one monitoring cycle
qnb.monitor.read_repo_capacity()        # Read peer builder states

# Verification
qnb.verifier.verify_all()               # 100-decimal accuracy + bounds check

# Research
qnb.research.full_research()            # 5-module research cycle
qnb.stochastic.run_stochastic_cycle(20) # Random experiments
qnb.test_gen.run_test_suite()           # Automated test suite

# Cross-Pollination
qnb.cross_pollinator.full_cross_pollination()    # Bidirectional with gates + links

# Nirvanic Engine
qnb.nirvanic.full_nirvanic_cycle()      # Ouroboros entropy fuel

# Consciousness + O₂ Superfluid
qnb.consciousness.full_consciousness_cycle()     # 4-phase consciousness cycle

# Math Research (11 engines)
from l104_numerical_engine.math_research import (
    RiemannZetaEngine, PrimeNumberTheoryEngine, InfiniteSeriesLab,
    NumberTheoryForge, FractalDynamicsLab, GodCodeCalculusEngine,
    TranscendentalProver, StatisticalMechanicsEngine,
    HarmonicNumberEngine, EllipticCurveEngine, CollatzConjectureAnalyzer,
)

# Quantum Computation (10 algorithms)
qnb.quantum_compute.quantum_phase_estimation(eigenvalue)
qnb.quantum_compute.hhl_linear_solver(A, b)
qnb.quantum_compute.variational_quantum_eigensolver(hamiltonian)

# 100-decimal precision utilities
from l104_numerical_engine import D, fmt100
x = D('3.14159265358979323846')          # Decimal with 120-digit precision
formatted = fmt100(x)                     # Format to 100 decimal places
```

## Science Engine Quick Reference

```python
from l104_science_engine import ScienceEngine
se = ScienceEngine()

# Entropy subsystem — Maxwell's Demon reversal
se.entropy.calculate_demon_efficiency(local_entropy)  # Demon reversal efficiency
se.entropy.inject_coherence(noise_vector)             # Order from noise

# Coherence subsystem — quantum-inspired coherence
se.coherence.initialize(seed_thoughts)     # Seed coherence state
se.coherence.evolve(steps)                 # Evolve coherence N steps
se.coherence.anchor(value)                 # Anchor coherence point
se.coherence.discover()                    # Discover coherence patterns

# Physics subsystem — sacred physics
se.physics.adapt_landauer_limit(temperature)     # Landauer limit at T (J/bit)
se.physics.derive_electron_resonance()           # Electron resonance
se.physics.calculate_photon_resonance()          # Photon resonance energy
se.physics.generate_maxwell_operator(dimension)  # Maxwell operator matrix
se.physics.iron_lattice_hamiltonian(n_sites)     # Fe lattice Hamiltonian

# Multidimensional subsystem
se.multidim.process_vector(vector)                          # Process ND vector
se.multidim.project(target_dim)                             # Project to lower dim
se.multidim.phi_dimensional_folding(source_dim, target_dim) # PHI-folding

# Quantum 26Q circuit science (Fe(26) iron-mapped)
se.quantum_circuit.get_25q_templates()     # Legacy 25Q templates (26Q via l104_26q_engine_builder)
se.quantum_circuit.analyze_convergence()   # GOD_CODE convergence analysis
se.quantum_circuit.plan_experiment()       # Plan quantum experiment
se.quantum_circuit.build_hamiltonian()     # Build Hamiltonian
```

## Math Engine Quick Reference

```python
from l104_math_engine import MathEngine
me = MathEngine()

me.fibonacci(n)                    # Returns LIST of Fibonacci numbers up to F(n)
me.primes_up_to(n)                 # Prime sieve up to n
me.god_code_value()                # GOD_CODE constant
me.lorentz_boost(four_vector, axis, beta)  # 4D Lorentz transform
me.prove_all()                     # Run all sovereign proofs
me.prove_god_code()                # Stability-nirvana proof for GOD_CODE
me.hd_vector(seed)                 # Create hyperdimensional vector (NOT hypervector)
me.wave_coherence(freq1, freq2)    # Wave coherence between two frequencies
me.sacred_alignment(frequency)     # Check sacred alignment of a frequency

# Direct layer access:
me.pure_math.prime_sieve(n)        # PureMath layer
me.god_code.*                      # GodCodeDerivation layer
me.harmonic.resonance_spectrum(fundamental, harmonics)  # HarmonicProcess layer
me.harmonic.verify_correspondences()                    # Fe/286Hz correspondence
me.harmonic.sacred_alignment(frequency)                 # Sacred alignment check
me.wave_physics.phi_power_sequence(n)                   # WavePhysics: φ^0..φ^(n-1)
me.dim_4d.*                        # Math4D layer (static: lorentz_boost_x/y/z)
me.dim_5d.*                        # Math5D layer
me.manifold.*                      # ManifoldEngine layer
me.void_math.*                     # VoidMath layer
me.abstract.*                      # AbstractAlgebra layer
me.ontological.*                   # OntologicalMath layer
me.proofs.*                        # SovereignProofs (static methods only)
me.hyper.*                         # HyperdimensionalEngine layer
```

## Quantum Gate Engine Quick Reference

```python
from l104_quantum_gate_engine import get_engine, GateCircuit, GateCompiler
from l104_quantum_gate_engine import H, CNOT, Rx, Rz, PHI_GATE, GOD_CODE_PHASE
from l104_quantum_gate_engine import ErrorCorrectionScheme, ExecutionTarget, OptimizationLevel, GateSet

engine = get_engine()  # Singleton orchestrator

# Circuit building
circ = engine.bell_pair()                              # Bell state (H + CNOT)
circ = engine.ghz_state(5)                             # GHZ state (N qubits)
circ = engine.quantum_fourier_transform(4)             # QFT (4 qubits)
circ = engine.sacred_circuit(3, depth=4)               # Sacred L104 circuit
circ = engine.create_circuit(2, "custom")              # Custom circuit
circ.h(0).cx(0, 1).append(PHI_GATE, [0])              # Build with gate calls

# Compile (4 optimization levels, 6 target gate sets)
result = engine.compile(circ, GateSet.IBM_EAGLE, OptimizationLevel.O2)
result = engine.compile(circ, GateSet.CLIFFORD_T)      # Fault-tolerant decomposition
result = engine.compile(circ, GateSet.L104_SACRED, OptimizationLevel.O3)

# Error correction
protected = engine.error_correction.encode(circ, ErrorCorrectionScheme.SURFACE_CODE, distance=3)
protected = engine.error_correction.encode(circ, ErrorCorrectionScheme.STEANE_7_1_3)
protected = engine.error_correction.encode(circ, ErrorCorrectionScheme.FIBONACCI_ANYON)

# Execute (8 targets: local statevector, Qiskit Aer, IBM QPU, coherence engine, ASI, ...)
result = engine.execute(circ, ExecutionTarget.LOCAL_STATEVECTOR)
result.probabilities  # {'00': 0.5, '11': 0.5}
result.sacred_alignment  # GOD_CODE resonance score

# Full pipeline: build → compile → protect → execute → analyze
pipeline = engine.full_pipeline(circ, target_gates=GateSet.UNIVERSAL,
    optimization=OptimizationLevel.O2,
    error_correction=ErrorCorrectionScheme.STEANE_7_1_3,
    execution_target=ExecutionTarget.LOCAL_STATEVECTOR)

# Gate algebra (40+ gates, decomposition, analysis)
algebra = engine.algebra
algebra.zyz_decompose(gate.matrix)                     # ZYZ Euler decomposition
algebra.kak_decompose(two_qubit_gate.matrix)            # KAK/Cartan decomposition
algebra.pauli_decompose(gate.matrix)                    # Pauli basis decomposition
algebra.sacred_alignment_score(PHI_GATE)                # Sacred resonance analysis
analysis = engine.analyze_gate(CNOT)                    # Full gate analysis
```

## Code Engine Quick Reference

```python
from l104_code_engine import code_engine

# Top-level hub methods (NOT sub-objects like .doc_gen or .test_gen)
code_engine.full_analysis(code)                        # Full code analysis
code_engine.generate_docs(source, style, language)     # Documentation generation
code_engine.generate_tests(source, language, framework)# Test scaffolding
code_engine.auto_fix_code(source)                      # Auto-fix → (fixed, log)
code_engine.smell_detector.detect_all(code)            # Code smell detection
code_engine.perf_predictor.predict_performance(code)   # Performance prediction
code_engine.refactor_engine.refactor_analyze(source)   # Refactor opportunities
code_engine.excavator.excavate(source)                 # Dead code archaeology
code_engine.translate_code(src, from_l, to_l)          # Translation
code_engine.audit_app(path, auto_remediate=True)       # 10-layer audit
code_engine.scan_workspace(path)                       # Workspace census
await code_engine.optimize(code)                       # Optimization (async)
```

## Unified Debug Framework

**File**: `l104_debug.py` v3.0.0 — Single entry point for ALL 11 engine packages

```bash
python l104_debug.py                       # Full suite, all engines
python l104_debug.py --engines code,math   # Only Code + Math engines
python l104_debug.py --engines quantum_gate,quantum_link,numerical,gate  # Quantum engines
python l104_debug.py --engines asi,agi,intellect  # ASI + AGI + Intellect
python l104_debug.py --phase boot          # Only boot phase
python l104_debug.py --phase constants     # Only constant alignment
python l104_debug.py --phase self-test     # Per-engine self-tests
python l104_debug.py --phase cross         # Cross-engine pipelines
python l104_debug.py --json                # JSON report to stdout
python l104_debug.py --report out.json     # Save JSON report to file
python l104_debug.py -v                    # Verbose output
```

## Cross-Engine Debug Suite

**File**: `cross_engine_debug.py` — 41 tests, 7 phases, validates all 3 engines together

| Phase | Tests | What It Validates |
|-------|-------|-------------------|
| 1 - Parallel Boot | 3 | All engines initialize concurrently in threads |
| 2 - Constants | 7 | GOD_CODE, PHI, VOID_CONSTANT match across all engines |
| 3 - Science→Math | 6 | Physics outputs fed to math functions |
| 4 - Math→Science | 6 | Math outputs fed to science functions |
| 5 - Code→Both | 6 | Code engine analyzes science/math source code |
| 6 - Both→Code | 6 | Science/math data used for code generation/testing |
| 7 - Integration | 7 | Full pipeline: physics→god-code→code-gen→analysis |

Run: `.venv/bin/python cross_engine_debug.py`

## Three-Engine Upgrade Suite

**File**: `three_engine_upgrade.py` — 8 phases, uses all 3 engines to analyze + upgrade ASI/AGI

| Phase | What It Does |
|-------|-------------|
| 1 | Parallel engine boot (Code + Science + Math) |
| 2 | Code Engine analysis of ASI + AGI (smells, perf, complexity) |
| 3 | Math Engine validation (GOD_CODE, Fibonacci→PHI, Lorentz, harmonics) |
| 4 | Science Engine validation (entropy reversal, coherence, 26Q, physics) |
| 5 | Cross-Engine synthesis (complexity×demon, proof→quantum, calibration) |
| 6 | Upgrade report generation (`three_engine_upgrade_report.json`) |
| 7 | Code generation (docs, tests, auto-fix for both cores) |
| 8 | Final cross-engine verification (7 checks) |

Run: `.venv/bin/python three_engine_upgrade.py`

## Three-Engine Integration (v8.0/v57.0)

Both ASI (v8.0) and AGI (v57.0) cores now include three-engine integration:

```python
# Three-engine scoring (available on both agi_core and asi_core)
core.three_engine_entropy_score()         # Science Engine: Maxwell Demon efficiency
core.three_engine_harmonic_score()        # Math Engine: GOD_CODE alignment + wave coherence
core.three_engine_wave_coherence_score()  # Math Engine: PHI-harmonic phase-lock
core.three_engine_status()                # Status of all three engine connections

# v30.0: Quantum network capacity scoring (on asi_core)
asi_core.quantum_network_health_score()        # Network health composite (fidelity, capacity, sacred, freshness)
asi_core.quantum_network_capacity_score()      # Channel density + pair availability + redundancy
asi_core.quantum_network_teleport_fidelity_score()  # Teleportation fidelity + sacred alignment + recovery

# AGI: 13-dimension scoring (was 10D)
agi_core.compute_10d_agi_score()  # D0-D9 original + D10 entropy + D11 harmonic + D12 wave

# ASI: 15-dimension scoring (was 12D) → v30.0: +3 quantum network dimensions
asi_core.compute_asi_score()      # 50+ dimensions including quantum network capacity
# v30.0 Quantum Network dimensions:
#   qnet_health             — network health composite (fidelity, capacity, sacred, freshness)
#   qnet_capacity           — channel density + pair availability + redundancy
#   qnet_teleport_fidelity  — teleportation fidelity + sacred alignment + recovery accuracy
```

## God Code Simulator Quick Reference

```python
from l104_god_code_simulator import god_code_simulator

# Run a single simulation by name
result = god_code_simulator.run("entanglement_entropy")

# Run all 23 simulations (4 categories: core, quantum, advanced, discovery)
report = god_code_simulator.run_all()

# Run by category
quantum_results = god_code_simulator.run_category("quantum")

# Parametric sweeps (dial, noise, depth, qubit scaling)
sweep = god_code_simulator.parametric_sweep("dial_a", start=0, stop=8)
noise = god_code_simulator.parametric_sweep("noise")
depth = god_code_simulator.parametric_sweep("depth")

# Adaptive circuit optimization
opt = god_code_simulator.adaptive_optimize(target_fidelity=0.99, nq=4, depth=4)
noise_opt = god_code_simulator.optimize_noise_resilience(nq=2, noise_level=0.1)

# Engine-ready payload converters (on SimulationResult)
result.to_coherence_payload()     # → CoherenceSubsystem.ingest_simulation_result()
result.to_entropy_input()         # → EntropySubsystem.calculate_demon_efficiency()
result.to_math_verification()     # → MathEngine verification
result.to_asi_scoring()           # → ASI pipeline scoring

# Multi-engine feedback loop (sim → coherence → entropy → scoring)
god_code_simulator.connect_engines(coherence=se.coherence, entropy=se.entropy, math_engine=me)
fb = god_code_simulator.run_feedback_loop(iterations=5)

# Direct submodule access
from l104_god_code_simulator.constants import GOD_CODE, PHI, VOID_CONSTANT
from l104_god_code_simulator.quantum_primitives import init_sv, apply_single_gate, H_GATE
from l104_god_code_simulator.simulations import ALL_SIMULATIONS
from l104_god_code_simulator.simulations.core import sim_conservation_proof
from l104_god_code_simulator.simulations.quantum import sim_entanglement_entropy
from l104_god_code_simulator.simulations.advanced import sim_grover_search
from l104_god_code_simulator.simulations.discovery import sim_iron_manifold
```

## Detailed Docs

| Path | Content |
|------|---------|
| [`L104_MASTER_KNOWLEDGE.md`](L104_MASTER_KNOWLEDGE.md) | **Universal reference** — all engines, APIs, constants, deployment, benchmarks, EVO history, daemons, quantum, Swift app |
| [`deepseek.md`](deepseek.md) | **Nova Soul + DeepSeek** — Soul Daemon, Grover search (8 implementations), OpenClaw agents, QPU integration |
| `docs/claude/architecture.md` | Cognitive architecture, MCP config, agents, EVO history |
| `docs/claude/code-engine.md` | Code Engine v6.3.0 — full API, 31 subsystems, 10-layer audit |
| `docs/claude/swift-app.md` | L104SwiftApp build system, 150 Swift source files |
| `docs/claude/evolved-asi-files.md` | ASI evolution log, decomposed package details |
| `docs/claude/api-reference.md` | FastAPI endpoints and server routes |
| `docs/claude/guides/code-examples.md` | Practical code patterns |
| `docs/claude/guides/memory-persistence.md` | State file management |
| `docs/claude/guides/optimization.md` | Performance tuning |
| `docs/claude/guides/zenith-patterns.md` | Zenith frequency patterns |

## Codebase Metrics

- **1,257** Python files at root, **783** L104 modules
- **23** Python packages + **20** infrastructure directories (43 total `l104_*/` dirs)
- **150** Swift files (134,664 lines) in L104SwiftApp
- **43** `.l104_*.json` state files
- **390** API route handlers in `l104_server/app.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lockephi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
