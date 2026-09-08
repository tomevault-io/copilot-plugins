## doomfly

> - This repository contains the MaleCNS v1.0 / ViZDoom experiment only.

# DOOMFLY project constraints

- This repository contains the MaleCNS v1.0 / ViZDoom experiment only.
- Retain every released connection between the retained neuronal entries. Do not
  crop circuits, prune weak/self edges, or replace the network with a game policy.
- Keep live RGB input, modeled neural propagation, reinforcement, plasticity and
  fixed neuron-to-button decoding separate and traceable. Game-state telemetry
  must not select actions or secretly steer, aim or recover a stuck controller.
- Document measured circuitry, inferred mappings, chosen dynamics and unresolved
  mechanisms separately. A full retained connectome is not a literal living brain.
- Preserve failed experiments and controls. Changed weights and long survival
  alone do not establish learning; numerical tests are not biological validation.
- Store credentials and machine-specific origins only in ignored configuration.
- Preserve third-party notices. Do not bundle external research workbooks, papers,
  commercial game assets, dependency checkouts, or unrelated projects in archives.

## DOOMFLY launch documentation requirement

- User instruction (2026-09-05): before giving a launch green light, update and
  publish "How it works" to match the exact deployed model, including learning.
- Explain live pixels → sensory neurons → neural activity → fixed game controls,
  plus the reinforcement input, identified plastic connections, what changes
  during training, and what stays fixed. State whether the public stream is
  training, evaluating frozen learned weights, or running the original baseline.
- Include the actual connectome release, supporting sources, validation results
  and remaining assumptions. Do not describe an unvalidated candidate as proven
  fly learning. Keep the explanation brief, accessible and consistent with the
  site's pixel-art scientific/gamer voice and infographics.
- Check the published page against the release's code, configuration and evidence
  before declaring it ready; a draft or an explanation of an older model is not
  sufficient. This requirement does not mean the scientific launch gates passed.

- Spectator direction (user, 2026-09-05): plan for an immediately watchable shared
  training experiment, potentially spanning hours or days, with minimal required
  interaction and an entertaining but scientifically accurate progress story.
  Follow `docs/doomfly-spectator-experience.md`: distinguish wall/brain time and
  live training/evaluation/replay; preserve experiment continuity and controls.
  Long runtime and changed weights alone do not establish successful learning.

---
> Source: [nftechie/doomfly](https://github.com/nftechie/doomfly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
