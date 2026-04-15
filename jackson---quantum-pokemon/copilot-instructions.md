## quantum-pokemon

> To build a **Hybrid Quantum-Classical Reinforcement Learning (QRL) Agent** capable of playing Pokémon Red (Game Boy).

# Project Context: Quantum Red (Hybrid QRL Agent)

## 1. Project Objective
To build a **Hybrid Quantum-Classical Reinforcement Learning (QRL) Agent** capable of playing Pokémon Red (Game Boy).
* **Goal:** The agent must navigate the world (movement) and handle battles.
* **Key Constraint:** The high-level decision-making (Policy Network) must be executed by a **Variational Quantum Circuit (VQC)**, not purely classical logic.
* **User Persona:** Full Stack Python/JS Engineer. Experienced in distributed systems. New to Quantum.

## 2. Technical Stack
* **Language:** Python 3.10+
* **Emulator:** `PyBoy` (headless Game Boy emulator).
* **RL Framework:** `Gymnasium` (formerly OpenAI Gym) for wrapping the emulator.
* **Classical ML:** `PyTorch` (for the Convolutional Neural Network feature extractor).
* **Quantum ML:** `PennyLane` (for the Quantum Circuit and gradients).
* **Backend:** `default.qubit` (Simulator) initially, upgradable to `qiskit.ibmq` later.

## 3. Architecture: The "Hybrid Sandwich"
Current Quantum hardware cannot process raw pixels (160x144 inputs). We will use a Hybrid architecture:

1.  **The Eye (Classical):** A PyTorch CNN (Convolutional Neural Network) takes the raw screen buffer. It compresses the visual data into a small vector of **4 to 8 features**.
2.  **The Brain (Quantum):** These features are encoded as rotation angles into a Quantum Circuit (using PennyLane). The circuit creates entanglement (CNOT gates) and rotations to calculate the "Q-Values" (expected reward).
3.  **The Hands (Action):** The Quantum measurement is decoded into a discrete action: `[UP, DOWN, LEFT, RIGHT, A, B, START, SELECT]`.

## 4. Game Memory Map (Pokémon Red)
*Crucial RAM addresses for the Reward Function:*
* **X Coordinate:** `0xD362`
* **Y Coordinate:** `0xD361`
* **Map ID:** `0xD35E`
* **Party Count:** `0xD163`
* **Pokemon 1 HP:** `0xD16C` (Current) / `0xD16E` (Max)
* **Enemy HP:** `0xCFE6` (Current)
* **Badges:** `0xD356`

## 5. Development Phases

### Phase 1: The Environment (Class: `PokeGlass`)
* Create a custom `gym.Env` wrapper around PyBoy.
* Implement `step()`, `reset()`, and `render()`.
* **Critical Task:** Implement a `get_reward()` function that reads RAM to reward the agent for:
    1.  Discovering new Map IDs (Exploration).
    2.  Winning battles (Enemy HP = 0).
    3.  Leveling up.

### Phase 2: The Classical Baseline
* Build the pipeline using *only* PyTorch first.
* Ensure the agent can learn basic movement (walking out of the house).
* *Why:* To prove the Environment and Reward function work before introducing Quantum complexity.

### Phase 3: The Quantum Brain (Class: `QuantumNet`)
* Replace the final Linear Layer of the PyTorch model with a `PennyLane` QNode.
* **Encoding:** Use `AngleEmbedding` to map the 4 Classical Features to 4 Qubits.
* **Ansatz:** Use `StronglyEntanglingLayers` or `BasicEntanglerLayers` for the decision logic.
* **Measurement:** Measure `PauliZ` on all qubits to determine Action Probabilities.

## 6. Commands & Rules for Gemini
* **Code Style:** Modular Python. Separate `environment.py`, `agent.py`, and `circuit.py`.
* **Quantum Logic:** Explain *why* specific gates (Hadamard, CNOT) are being used when generating circuit code.
* **Debugging:** If the loss doesn't decrease, suggest checking the "Learning Rate" first, then the "Entanglement Depth."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackson-)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jackson-)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
