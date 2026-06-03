## runpod-serverless

> RunPod serverless deployment for [ivrit.ai](https://ivrit.ai) speech-to-text. Wraps the `ivrit` Python package in a Docker container for serverless GPU inference on RunPod.

# runpod-serverless

RunPod serverless deployment for [ivrit.ai](https://ivrit.ai) speech-to-text. Wraps the `ivrit` Python package in a Docker container for serverless GPU inference on RunPod.

## Key Files

- `infer.py` — RunPod handler: loads models via `ivrit.load_model()`, transcribes audio (streaming or batch), supports diarization.
- `Dockerfile` — Builds the container image: PyTorch base, ffmpeg, `ivrit[all]`, pre-downloaded models.
- `test_input.json` — Sample RunPod job input for local testing.

## Agent-Based Workflow

This project uses five specialized subagents in `.claude/agents/`. They are the canonical source of truth for how work is done here.

| Agent       | Purpose                                                            |
|-------------|--------------------------------------------------------------------|
| `visionary` | Strategic alignment for direction-changing decisions               |
| `architect` | Technical design when components/boundaries/dependencies change   |
| `coder`     | Implementation following project conventions                       |
| `reviewer`  | Final quality gate — NO HACK, NO OVER-ENGINEERING                 |
| `manager`   | Completion confirmation and credit attribution                     |

## Workflow

For any non-trivial change, dispatch the subagents in this order, skipping any whose "When to Invoke" criteria don't apply:

1. **visionary** — only for feature proposals, strategic pivots, or long-term architectural decisions.
2. **architect** — when adding components, changing boundaries, introducing dependencies, or changing interfaces.
3. **coder** — for all implementation work.
4. **reviewer** — mandatory before declaring any change done. Must return APPROVED.
5. **manager** — only after reviewer approves. Produces the completion summary.

If reviewer requests changes, push back to architect or coder as indicated and re-run the affected phases.

## Ensuring the Right Agent Runs

- **Trust the descriptions.** Each subagent's frontmatter includes `PROACTIVELY` / `MUST BE USED` triggers so Claude Code auto-dispatches them. Don't paraphrase their work inline — delegate.
- **reviewer is non-optional.** No change is complete without a `REVIEW` block returning APPROVED. If you're about to report "done" without one, stop and invoke reviewer.
- **manager closes every workflow.** After reviewer approves, invoke manager to emit the completion block.
- **One agent at a time.** Wait for each subagent's output block before dispatching the next; later agents depend on the previous block as input.
- **When unsure which agent applies**, default to the more senior one (visionary > architect > coder) and let its decision framework decide whether to proceed or downscope.
- **Manual override.** You can always force a specific agent with phrasing like "use the reviewer subagent on this change" — useful if auto-dispatch misses.

Run `/agents` to confirm all five subagents are registered.

---
> Source: [ivrit-ai/runpod-serverless](https://github.com/ivrit-ai/runpod-serverless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
