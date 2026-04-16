## self-driving

> This project is a Donkey Car simulation setup running on Linux with the Unity Simulator.

# Project Status: Self Driving Simulation

## Context
This project is a Donkey Car simulation setup running on Linux with the Unity Simulator.
The goal is to train an autonomous driving agent (`mypilot.h5`).

## Environment
- **Path**: `~/mysim`
- **Python**: Managed via `uv` (Python 3.11).
- **Dependencies**: `donkeycar[pc]`, `gym==0.22.0`, `gym-donkeycar` (git version).
- **Simulator**: `/home/igtm/projects/DonkeySimLinux/donkey_sim.x86_64`

## Key Configurations (`myconfig.py`)
- **Simulator Integration**: `DONKEY_GYM = True`.
- **Simulator Path**: Hardcoded in `myconfig.py`.
- **Startup Delay**: `GYM_CONF['start_delay'] = 20` (increased to prevent connection refused errors).
- **Joystick**: `USE_JOYSTICK_AS_DEFAULT = False` (explicitly disabled to prevent "ASRock LED Controller" from sending 0-values and overriding web control).
- **Drive Train**: `DRIVE_TRAIN_TYPE = "MOCK"` (safeguard, though Gym overrides this).

## Code Modifications
- **`manage.py`**:
    - `add_simulator` call moved to the end of the `drive()` pipeline (after `add_user_controller` and `DriveMode`).
    - This ensures `DonkeyGymEnv` receives the *current frame's* steering/throttle inputs, reducing latency issues.
    - `ThrottleFilter` is active.
- **`dgym.py`**: Debug prints were added and then removed.

## Workflow
1.  **Drive/Collect**: `uv run python manage.py drive` -> Web UI (User Mode) -> Record.
2.  **Train**: `uv run python train.py --tubs=data --model=models/mypilot.h5`.
3.  **Autonomous**: `uv run python manage.py drive --model=models/mypilot.h5` -> Web UI (Local Pilot Mode).

## Next Steps
- Collect more data (reverse track, recovery from corners) to improve model stability.
- Investigate why `gym-donkeycar` requires git version vs pip version (dependency conflict resolution).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/igtm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
