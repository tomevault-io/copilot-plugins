## project-doberman

> **Status**: NIGHTHAWK HiRes mega dataset is actively generating on the GPU server (GPUs 2-7). Runtime: several days. Completion target: mid next week.

# ⚠️ NIGHTHAWK MEGA DATASET IS BEING BUILT

**Date**: 2026-04-11
**Status**: NIGHTHAWK HiRes mega dataset is actively generating on the GPU server (GPUs 2-7). Runtime: several days. Completion target: mid next week.

## What this means for this module

- **DO NOT** run SAM3 / CUT / generation on GPUs 2-7 until NIGHTHAWK finishes — you will thrash the disk and kill both jobs.
- **Use GPUs 0 or 1** for any training during this window.
- **Output path**: `/mnt/train-data/datasets/nighthawk_mega_highres/` — do NOT write into this path; NIGHTHAWK agents own it.
- **Future asset**: once complete, the HiRes mega pool (~2.75M augmented frames across day2night/day2fog/day2rain/day2dusk/rgb2thermal) will be available as a shared training source. Your module may benefit for finetuning.
- **No dataset duplication**: if your module needs aerial RGB or augmented night/thermal UAV data, check `/mnt/train-data/datasets/nighthawk_mega_highres/` when the flag file `DONE.flag` appears. Until then, use your own data.

## Key references

- Module path: `/mnt/forge-data/modules/07_night_augmentation`
- Script running: `scripts/build_mega_hires.sh`
- Monitor: `tail -f /mnt/artifacts-datai/logs/nighthawk_mega_hires/build.log`
- Input inventory: `/mnt/forge-data/modules/07_night_augmentation/inputs.md`

## Shenzhen deferred additions

After Apr 23, NIGHTHAWK will get a HiRes extras pass with +250K new UAV source frames (UAVDT, UAVid, UAVDark135, Anti-UAV, IndraEye, MAFS, UDD6, MM-UAV, AWMM-100K, CST Anti-UAV, EV-UAV) pulled over the Shenzhen fast link. Expect the pool to roughly double.

**If you need nighthawk data and the DONE.flag is missing, WAIT — do not trigger your own augmentation pipeline in parallel.**

---
> Source: [RobotFlow-Labs/project_doberman](https://github.com/RobotFlow-Labs/project_doberman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
