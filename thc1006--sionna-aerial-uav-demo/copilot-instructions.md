## sionna-aerial-uav-demo

> > **Copy this file to `CLAUDE.md` and customize for your environment.**

# UAV-ACAR-Sionna – Claude Code Instructions

> **Copy this file to `CLAUDE.md` and customize for your environment.**
> `CLAUDE.md` is git-ignored so your local settings won't be committed.

You are **Claude Code** assisting with the UAV-ACAR-Sionna pipeline project.

---

## 1. Environment Information

**Fill in your machine details:**

- Machine name: `YOUR_HOSTNAME`
- GPU: `YOUR_GPU_MODEL` (e.g., RTX 4090, A100)
- Password (if needed for sudo): `YOUR_PASSWORD`
- NGC API Key: `YOUR_NGC_API_KEY`

---

## 2. Project Status

**Phase 0-2: ✅ Complete**
- Sionna channel simulation (Rayleigh fading + AWGN)
- ML models (FCRegressor R²=0.89, CNN1D architecture)
- FastAPI REST API (/health, /emulate, /results)
- Dataset structure (summary.csv, npz, h5 stubs)
- 14 tests passing

**Phase 3: 🔄 Ready to implement**
- Real MATLAB/Aerial mcore integration
- Real cuBB/testMAC/RU emulator containers
- Multi-host distributed deployment

---

## 3. Key Commands

```bash
# Setup environment
./scripts/setup.sh           # Full setup
./scripts/setup.sh --quick   # Quick (no GPU deps)
./scripts/setup.sh --phase3  # With Docker/NGC checks

# Activate venv
source .venv/bin/activate

# Run tests
pytest tests/

# Train ML models
uav-train --model fc --epochs 100 --device cuda

# Start API server
uav-api

# Phase 3: NGC login
./scripts/ngc_login.sh

# Phase 3: Docker
cd docker && docker-compose up -d
```

---

## 4. Golden Rules

1. **Log shell commands** - Use `scripts/run_and_log.sh` or log to `logs/commands.log`
2. **Run tests frequently** - After non-trivial changes, run `pytest -q`
3. **Small incremental changes** - Edit one module at a time
4. **Keep concerns separated**:
   - `uav_acar_sionna/orchestrator/` - adapters for Sionna/MATLAB/cuBB
   - `uav_acar_sionna/ml/` - PyTorch models and training
   - `uav_acar_sionna/api/` - FastAPI endpoints

---

## 5. Files to Know

| Path | Description |
|------|-------------|
| `src/uav_acar_sionna/orchestrator/sionna_adapter.py` | Sionna → NPZ conversion |
| `src/uav_acar_sionna/orchestrator/matlab_adapter.py` | NPZ → H5 TestVector (stub) |
| `src/uav_acar_sionna/orchestrator/cubb_adapter.py` | H5 → throughput/BLER (stub) |
| `src/uav_acar_sionna/ml/models.py` | FCRegressor, CNN1D models |
| `src/uav_acar_sionna/ml/train.py` | Training loop with CLI |
| `src/uav_acar_sionna/api/main.py` | FastAPI endpoints |
| `tests/conftest.py` | XLA_FLAGS configuration |
| `docker/docker-compose.yml` | Phase 3 container setup |

---

## 6. Phase 3 TODO

- [ ] Replace `matlab_adapter.py` stub with real MATLAB/Aerial mcore calls
- [ ] Replace `cubb_adapter.py` stub with real Docker container orchestration
- [ ] Add SionnaRT ray-tracing scenarios (currently using Rayleigh fading)
- [ ] Multi-host deployment configuration
- [ ] Real channel data for CNN model training

---

## 7. Troubleshooting

**TensorFlow XLA error:**
```bash
pip install nvidia-cuda-nvcc-cu12
```

**Docker permission:**
```bash
sudo usermod -aG docker $USER && newgrp docker
```

**NGC login:**
```bash
./scripts/ngc_login.sh
# Or manually: docker login nvcr.io -u '$oauthtoken' -p <api-key>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thc1006) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
