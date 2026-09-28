## aha-3d

> User instructions govern the task. Keep these rules in working context; load

# aha3d agent contract

User instructions govern the task. Keep these rules in working context; load
only the current stage's references from [docs/INDEX.md](docs/INDEX.md).
Keep docs concise; link operation-specific details instead of repeating them.
Keep hashes needed for input identity, cache validity and evidence binding; avoid
redundant large-file hashing. Hash agreement does not establish source fidelity,
geometry or output quality.

1. **Protect shared work.** Inspect Git status and `python3 tools/task_claim.py
   list --compact`; claim intended writable paths before edits or jobs. One writer
   per file/output `.blend`; keep claims while queued/running writers exist.
   Follow [coordination](docs/COORDINATION.md) for claims and recovery.
2. **Record the actual scope.** Before reconstruction work, claim the scene and
   delivery paths, then initialize/bind [acceptance](docs/WORKFLOW_ACCEPTANCE.md)
   to this task. Preserve user corrections as blocking findings. Resume established
   scope; ask only for missing decisions. Code/documentation tasks stay separate.
3. **Preserve inputs.** Open saved source scenes in background Blender and write
   distinct outputs. Keep source materials, requested camera/timing and existing
   local data/environments. Pi3X geometry is reference material, not the authored room.
   Persist outputs needed downstream. If a required result was not saved, rerun
   its producer and save it; do not substitute sparse or incomplete caches.
4. **Run appropriately.** Follow [MACHINE.md](MACHINE.md): run locally on the single
   GPU, one GPU job at a time. Use installed configured environments; do not modify
   another project's environment. Read setup/install guides only when configuring
   or troubleshooting a runtime.
5. **Wait efficiently and finish.** Follow [job waiting](docs/RENDER_EXECUTION.md):
   prefer scripts; use low-frequency checks when needed and continue through delivery.
6. **Prove completion.** Generated files, successful exits and historical reports
   are not acceptance. Inspect actual representative views and full video
   decoding/timing. Keep automated checks, visual review and selection distinct;
   [results](docs/RESULTS.md) uses the exact accepted selected manifest, never a
   filename or modification time. Rebuild/check the claimed Gallery for delivery.
7. **Complete the full requested pipeline.** Full source-human reconstruction includes
   scene/camera reuse or reconstruction, all-person tracking and pose estimation,
   validated common-scene position anchoring, v2, human-object and foot-ground
   contact refinement, collision/slip/source-projection checks, and final scene
   visualization/delivery. Intermediate caches or diagnostic previews are not
   completion. Repair failed stages and continue; narrow to a stage/debug result
   only when the user explicitly requests it.
8. **Finish the authorized work.** Preserve unfinished shared edits. Complete and
   validate the requested changes. Commit and push only within the current task's
   explicit or established authorization, using its agreed branch; do not default
   to pushing `main`. Reviews do not create commits. Follow [Git workflow](docs/GIT_WORKFLOW.md).

Task-specific rules live at their point of use:

- Source people: [motion controls](docs/MOTION_CONTROLS.md). Default GVHMR, fixed
  whole-clip rigid initial alignment at scale 1; full-pipeline requests include
  scene anchoring and contact refinement. Separate trajectory experiments require
  an explicit request. Kimodo is for new/changed actions or documented fallback.
  Default to v2 world post-optimization before scene contact refinement. Smooth
  whole-body translation may improve contact and penetration. Do not modify joint
  poses to remove collisions or run human 6D leg repair/leg IK. Check continuity,
  foot slip and source alignment after translation.
- Furniture/materials: search [assets](assets/INDEX.md) before authoring; use
  [Blender RoomKit](.agents/skills/blender-roomkit/SKILL.md) for stable-ID replacement,
  orientation and material preservation. Build/check the catalog after changing
  assets, skills, stages or demo metadata.
- Put references in `references/`, scene work in `scenes/<id>/`, outputs in `runs/`
  via [results](docs/RESULTS.md); reserve `runs/_tools` for cross-scene checks.
- Write new docs/filenames in English and hand off evidence, job/log paths and next
  actions. Keep reference attribution and [distribution terms](DISTRIBUTION.md).

---
> Source: [KevinXu02/aha-3d](https://github.com/KevinXu02/aha-3d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
