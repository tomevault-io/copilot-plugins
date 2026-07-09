## golfwebgui

> Use this repo as an ARC/NeuroGolf solving workbench, not as a place to hand-write ONNX files directly.

# NeuroGolf Lab Agent Notes

Use this repo as an ARC/NeuroGolf solving workbench, not as a place to hand-write ONNX files directly.

## Primary Workflow

1. Inspect task JSON in `client/dist/tasks/` or `client/public/tasks/`.
2. Form a transformation hypothesis.
3. Build a graph using the same node JSON shape the GUI emits.
4. Submit through `/api/export`.
5. Treat backend validation as the gate. A graph is not successful unless validation passes.
6. Hugging Face upload happens only after validation passes.

## Human + Agent Collaboration

- The human uses the browser GUI to inspect grids, connect nodes, edit constants/attributes, and export.
- The agent may use `scripts/agent_export.py` for headless attempts.
- If the agent finds a useful graph, explain the node/edge recipe so the human can recreate or inspect it in the GUI.
- Do not bypass the compiler by writing ONNX directly except for diagnostic comparison.

## Headless Export

Color remap shortcut:

```bash
python3 scripts/agent_export.py --task task276 --color-remap '{"6":2}'
```

Graph JSON:

```bash
python3 scripts/agent_export.py --task task010 --graph graph.json
```

The helper posts to the same `/api/export` endpoint as the GUI.

## Safety

- Do not print `.env` values.
- Do not commit secrets, tokens, logs, build output, archives, or dependency folders.
- Keep runtime hosts and tunnels in local deployment notes, not public source docs.
- Run `python3 -m pytest -q` and `cd client && npm run build` before commits.

---
> Source: [goldbar123467/GolfWebGUI](https://github.com/goldbar123467/GolfWebGUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
