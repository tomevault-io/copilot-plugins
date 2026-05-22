## simball-simple

> <!-- .github/copilot-instructions.md

<!-- .github/copilot-instructions.md
Guidance for AI coding agents working on the simball_simple repository.
Keep this file short, factual and tightly tied to discoverable code and scripts.
-->

# Quick context (why this repo exists)
- Purpose: train football-playing AIs (PPO + imitation learning) and expose HTTP team APIs for a browser-based simulator.
- Primary languages/tools: C# (.NET 8), JSON for model persistence, shell scripts for local training orchestration.

## Big-picture architecture (read these files early)
- Entry / runtime: `Program.cs` and `FootballInstructor.csproj` (root of `FootballInstructor/`).
- AI core: `Domain/AI/` — key files: `PPOFootballAI.cs`, `PPONetworks.cs`, `PPOExperience.cs`, `PPOBuffer.cs`, `GameStateEncoder.cs`, `RewardCalculator.cs`, `SimpleNeuralNetwork.cs`.
- Player services: `Domain/PlayerService.cs`, `Domain/PPOPlayerService.cs`, `Domain/RLPlayerService.cs` — these expose HTTP endpoints consumed by the simulator.
- Config & running: `appsettings.Development.json`, `start_training.sh`, `reset_training.sh`.
- Models: stored under the repo `models/` directory (e.g. `rl_model_instance_4.json`).

## Contract (short) for AI modules
- Inputs: game state JSON from simulator (`setup` GET and `update` POST). See `README.md` for field units: cm / s.
- Outputs: player instruction list (HTTP JSON) per `PlayerController` behavior.
- Success modes: training runs without exceptions, models saved periodically, the simulator accepts the HTTP responses.
- Error modes: NaNs in networks, model save failures, port conflicts.

## Developer workflows & exact commands (discoverable / reproducible)
- Start local training (recommended): run the provided script from repo root.
  ```bash
  ./start_training.sh
  ```
  It launches three instances (ports 5004, 5002, 5003). The scripts assume you run them from the repository root.
- If you need to run `dotnet` directly prefer absolute paths (see `AGENTS.md`). Example:
  ```bash
  dotnet run --project /home/joakim/Projects/simball_simple/FootballInstructor/FootballInstructor.csproj --urls=http://localhost:5004
  ```
- Force-save model via HTTP during debugging:
  ```bash
  curl http://localhost:5004/Player/force-save-model
  ```

## Project-specific conventions and patterns
- Simplicity-first: follow AGENTS.md rules (KISS, YAGNI). Keep functions small (<=60 lines) and total additions minimal.
- NaN protection: `PPONetworks.cs` and `SimpleNeuralNetwork.cs` contain explicit guards; preserve them when editing numerical code.
- Model serialization: networks are serialized to JSON using jagged arrays (2D -> jagged). When changing serialization, keep backward-compatible shape handling.
- Imitation blend: teacher actions are mixed into training (default 90% teacher / 10% PPO). Look at `PPOFootballAI.cs` and config keys under `AISettings`.

## Integration points and external behavior
- Simulator endpoints: the repo implements two endpoints the simulator calls — `setup` (GET) and `update` (POST). See `Controllers/PlayerController.cs` for exact routes and payload shapes.
- Ports used by training scripts: 5004 (instance 4), 5002 (instance 2), 5003 (heuristic teacher). Tests and runs assume these ports; avoid changing them without updating scripts.
- Model persistence: models saved automatically (every N games / minutes). Models live in `models/` and are named per instance (e.g. `rl_model_instance_4.json`).

## Where to make changes (and what to avoid)
- Tuning hyperparameters: change `appsettings.Development.json` keys under `AISettings` (PPOGamma, PPOLambda, PPOClipEpsilon, PPOBatchSize, etc.).
- Network structure/weights: `PPONetworks.cs` and `SimpleNeuralNetwork.cs`. When modifying shapes, update serialization/deserialization code.
- Reward shaping: `RewardCalculator.cs`. The repository currently uses sparse rewards (goals only) — avoid reintroducing dense rewards without careful testing (risk of reward hacking).
- Logging & metrics: follow existing log messages (examples in README). Don’t change metric names that monitoring or scripts parse.

## Quick pointers for common tasks / troubleshooting
- Ports already in use: use `lsof -i :5004` (or `pkill -f "dotnet run --urls"`) — scripts in README show common fixes.
- Models not saving: check `models/` directory permissions and console logs for `Model saved:` messages. Use `curl` force-save endpoint to verify.
- Strange actions early in training: expected due to exploration / imitation; monitor `Temperature` and action ratio tags `[TEACHER]` / `[PPO]` in logs.

## Examples from the codebase (where to look)
- HTTP controller: `Controllers/PlayerController.cs` shows how `setup` and `update` are wired.
- Game -> vector encoding: `Domain/AI/GameStateEncoder.cs` — use it when creating features for networks.
- PPO training loop & buffer: `Domain/AI/PPO/PPOFootballAI.cs`, `PPOBuffer.cs`, and `PPOExperience.cs`.

## Small checklist for PRs made by an AI agent
- Keep changes minimal and targeted. Mention exact files changed.
- Preserve NaN guards and serialization compatibility.
- Update `appsettings.Development.json` only for experiments — don't change defaults in PRs without note.
- If changing network shapes, add a migration path for existing model files.

---
If anything in here is unclear or you'd like more detail (example payloads, sample logs, or a short dev-run checklist), tell me which part to expand and I will iterate.

---
> Source: [jockesv/simball_simple](https://github.com/jockesv/simball_simple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
