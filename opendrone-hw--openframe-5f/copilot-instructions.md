## openframe-5f

> The 5" frame. The design source is the Onshape document in

# OpenFrame-5F

The 5" frame. The design source is the Onshape document in
`cad/onshape.json` (OpenDrone-V2, workspace 5"); this repository holds
the link, the released exports and the parts lists. It follows the Incutec
mechanical repository template. Use the Onshape skill for any CAD work.

| | |
|---|---|
| Status | See the `status-*` topic on the repo. Never written here. |
| Validate | `python3 <hardware-tooling>/hardware/mechanical_check.py .` |
| Release | `onshape_release.py cad/onshape.json --version <rev> --apply`, then one commit adding `releases/<rev>/` |
| License | CERN-OHL-S-2.0 |

- Never edit a file under `releases/`. Fix the model in Onshape and release a new revision.
- A part added, removed or re-counted in the Onshape assembly changes `parts.csv` or `hardware.csv` in the same pull request.
- The 3" and 5" frames share one Part Studio in OpenDrone-V2; a change to a shared part affects both repositories.
- Sourcing (suppliers, prices, quotes, RFQs, contacts) is handled by Incutec and never lives in OpenDrone repositories.

---
> Source: [OpenDrone-hw/OpenFrame-5F](https://github.com/OpenDrone-hw/OpenFrame-5F) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
