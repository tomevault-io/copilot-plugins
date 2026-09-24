## verl-vla

> This file records model-layer decisions that every verl-vla integration must

# Model Integration Agent Guide

This file records model-layer decisions that every verl-vla integration must
preserve. It applies to all code under `src/verl_vla/models/`.

## Model Ownership

- A trainable model wraps exactly one upstream-native policy. The wrapper may
  additionally own verl-vla training state such as critics, value heads,
  Flow-SDE parameters, or runtime adapters.
- Keep those two configuration domains separate. The upstream policy config
  describes the native model; adapter, critic, embodiment, dataset, and
  workflow settings belong to verl-vla configuration. Do not use the native
  config as a transport for framework settings or machine-local paths.
- Preserve the upstream policy implementation and loading API. Add explicit
  builders and adapters around it instead of converting it to a verl-vla model
  format or registering it through an unrelated AutoClass.

## Checkpoint and Export Contracts

There are two intentionally different artifacts:

1. The full verl checkpoint is the resumable training artifact. Its sharded
   model state represents the complete trainable wrapper, including the native
   policy and any critic or other auxiliary parameters. Optimizer, scheduler,
   RNG, and extra trainer state remain in their normal verl locations.
2. `actor/huggingface/` is the deployable upstream-native policy artifact. It
   contains only the policy weights and artifacts required by the upstream
   loader, such as its native config, processor, tokenizer, statistics, or
   normalization data. It must not contain critics, verl-vla adapter settings,
   workflow state, or runtime paths.

The supported export path is the verl checkpoint lifecycle:

- Include `hf_model` in the actor checkpoint `save_contents`.
- Let the VLA FSDP checkpoint manager gather the full wrapper state.
- Let the trainable model extract the native policy state and delegate artifact
  writing to the upstream policy's native save API.
- When LoRA is enabled, the full checkpoint may retain a sibling
  `actor/lora_adapter/`, while `actor/huggingface/` contains the merged native
  policy expected by deployment.

Do not add a second workflow-specific exporter or rely on the generic
Transformers model merger for a VLA wrapper. The checkpoint manager owns
distributed state gathering; the model integration owns the mapping from
wrapper state to native policy state; the upstream policy owns its exported
file format.

## Native Artifact Purity

- Treat every filename inside `actor/huggingface/` as upstream-owned. In
  particular, never write verl-vla runtime metadata to `adapter_config.json`;
  Transformers reserves that filename for PEFT and may misidentify the entire
  directory as a PEFT adapter.
- Do not export auxiliary wrapper state such as `critic.pt` beside the native
  policy. It already belongs to the full verl checkpoint and is restored from
  there.
- Make the generic verl config-save hook safe by construction. Prefer exposing
  the native policy config when the upstream model has one. If an integration
  uses a framework config at `model.config`, its hook must not claim ownership
  of files in the native export directory; the integration's native export
  method remains responsible for the real model config.
- Do not solve artifact contamination by renaming files during load, deleting
  stale files after export, or temporarily mutating native config fields. Fix
  the producer and ownership boundary instead.

## Required Validation

For a new model integration or checkpoint-format change, validate the real
artifact round trip rather than only calling `save_pretrained()` directly:

1. Save through the verl checkpoint manager with `hf_model` enabled.
2. Confirm the full checkpoint contains all enabled wrapper parameters,
   including auxiliary training state.
3. Confirm `actor/huggingface/` contains no verl-vla adapter, critic, or runtime
   metadata.
4. Load the exported policy and processor through the upstream implementation.
5. For LoRA, verify the sibling adapter is resumable and the Hugging Face
   artifact contains the merged native policy.

---
> Source: [verl-project/verl-vla](https://github.com/verl-project/verl-vla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
