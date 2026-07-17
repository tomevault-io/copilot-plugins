## minecraft-pvp-robo

> Repository guidance for agents working on the PVPRobo open core.

# AGENTS.md

Repository guidance for agents working on the PVPRobo open core.

## Project Purpose

Build and maintain a Windows-first, no-render Minecraft PvP reinforcement-learning environment:

```text
PaperMC --nogui server
PVPRobo Paper plugin
Mineflayer headless bot controller
Python Gymnasium-compatible environment and training tools
```

The public core starts with privileged structured observations and high-level actions. It keeps clean interfaces for future RL algorithms, self-play, behavior cloning, benchmarks, and visual distillation research. Do not implement visual rendering in this phase.

## Hard Constraints

- Use only private local/offline research servers controlled by the user.
- Do not implement public-server cheating, anticheat bypassing, stealth automation, or evasion.
- Efficiency is the primary design goal.
- Do not use a Minecraft graphical client in the training/runtime stack.
- Do not add screenshots, OpenGL, Selenium, browser automation, or computer-vision dependencies for Route A.
- The hot loop must not use HTTP per step.
- The hot loop must not write to disk per step.
- Use persistent localhost connections for plugin, bot, and Python communication.
- The main Gymnasium API must expose flat `np.float32` observations and simple integer action arrays.
- Rich dictionary snapshots are allowed only in `info`, debug paths, tooling, and tests.
- Reset episodes in place. Do not restart the Minecraft server per episode.
- All ports, versions, paths, `action_repeat_ticks`, timeouts, and worker counts must be configurable.
- Windows 10 support is explicit: provide PowerShell scripts, avoid Unix-only shell assumptions, and keep Python paths cross-platform with `pathlib` where practical.
- Do not silently accept the Minecraft EULA. Provide an explicit script flag or documented manual step.

## Public/Private Boundary

Keep protocol, observations, actions, model formats, policy-pool manifests, and duel manifests in the open core.

Keep authenticated entry, cloud deployment, private maps, skins, signed textures, leaderboards, monitoring, live server handoffs, and operator automation out of the open core. Private extensions should layer on top through configs, launchers, optional plugin extensions, and released model files without forking the training protocol.

## Preferred Layout

```text
server-plugin/        Paper plugin
bot-controller/       Node.js/TypeScript Mineflayer controller
python/               Python package and tests
scripts/              Windows PowerShell setup/build/run/train/eval tools
configs/              Public local defaults and examples
docs/                 Public docs
runs/                 Generated worker state, gitignored
downloads/            Downloaded third-party artifacts, gitignored
```

## Required Tests

Maintain tests for:

- Protocol parsing and validation.
- Action encoding.
- Observation flattening.
- Python environment behavior against mock plugin/bot servers.
- Public manifest tooling.
- Smoke path that can execute random actions for at least 100 steps once integration components exist.

## Configuration

The following must be configurable, not hard-coded:

- Paper/Minecraft version.
- Plugin version and build paths.
- Java, Gradle, Node, Python, and virtualenv paths.
- Server root and worker run directories.
- Hostnames and ports for plugin, bot controller, and Python clients.
- Bot usernames and counts.
- Number of workers.
- `action_repeat_ticks`.
- Reset and connection timeouts.
- Arena/world paths and coordinates.

## Definition Of Done

The open core is in a usable state when:

- The plugin can be built.
- Node and Python dependencies can be installed.
- A local Paper server starts with `--nogui`.
- Two headless bots connect.
- A 1v1 arena resets in place.
- Random actions can execute for at least 100 steps.
- The Python API returns Gymnasium-compatible `obs`, `reward`, `terminated`, `truncated`, and `info`.
- Tests cover protocol parsing, action encoding, observation flattening, and Python env behavior against mock servers.
- PowerShell scripts and docs explain how to run on Windows 10.

---
> Source: [KyaKayKya/Minecraft-PVP-ROBO](https://github.com/KyaKayKya/Minecraft-PVP-ROBO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
