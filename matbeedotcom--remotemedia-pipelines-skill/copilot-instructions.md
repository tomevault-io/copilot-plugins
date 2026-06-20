## remotemedia-pipelines-skill

> Use when running, prototyping, composing, or benchmarking remotemedia SDK media pipelines from Python — audio/video/text processing, Whisper STT, Kokoro/VibeVoice/Voxtral/CosyVoice3 TTS, LFM2/LFM25 audio, Silero VAD, resampling, real-time bidirectional streaming sessions, mic→speaker S2S, or any manifest with `node_type` like `WhisperSTTNode`, `KokoroTTSNode`, `FastResampleNode`, `SileroVADNode`, `AudioChunkerNode`, `VogentTurnOnnxNode`, `LFM25AudioOnnxNode`. Triggers on `from remotemedia.core.pipeline import Pipeline`, `pipeline.run(...)`, `execute_pipeline(...)`, `create_streaming_session(...)`, `benchmark_pipeline(...)`, and questions about how to construct, execute, profile, or stream a remotemedia manifest dynamically. Also covers cuDNN/CUDA enablement for ONNX-based nodes and the streaming-session `send_input`/`recv_audio` pattern.


# remotemedia Pipelines (Python)

## What this skill is for

Build and execute media pipelines from Python using the `remotemedia` package. Five entry points exist — pick by what you have in hand:

| You have | Entry point | Notes |
|---|---|---|
| `Node` instances | `Pipeline.run(input)` | Easiest. Auto-falls back to Python executor when a node isn't in the Rust registry. |
| A list of `Node` instances or dict-manifests | `execute_pipeline(items)` / `execute_pipeline_with_input(items, inputs)` | Goes through `runtime_wrapper`. Pure Node lists use `execute_pipeline_with_instances` (registry-bypass). **Strict unary** — one input → one output, session closes. |
| A JSON/dict manifest with Rust `node_type`s | `remotemedia.runtime.execute_pipeline(json_str, enable_metrics)` | Direct FFI. Only works for node types the linked `.so` actually registered. Strict unary. |
| A long-running bidirectional pipeline (mic→speaker, live VAD, streaming TTS, multimodal interleaved S2S) | `remotemedia.runtime.create_streaming_session(manifest_json)` | **Continuous** input/output. Multiple `send_input` + multiple `recv_audio` / `recv_data` / `recv_video`. Required for any manifest whose nodes emit multiple outputs per input (LFM2-Audio interleaved, VAD+`AudioBufferAccumulatorNode` chains, anything streaming TTS). |
| A manifest you want to **profile** end-to-end (TTFA / TTFT / per-node percentiles / capture) | `remotemedia.runtime.benchmark_pipeline(manifest_json, input_paths, options_json=None)` | Replays one or more `.wav`/`.txt` inputs as utterances through the manifest, measures eos-to-first-emit + per-node merged HDR percentiles, optionally captures per-utterance outputs to disk. Returns the bench report as a JSON string. |

`pipeline.run()` is the safest default — it tries Rust, then transparently falls back. Use `create_streaming_session` whenever the pipeline is genuinely streaming — the unary entry points only emit the *first* output and close the session, which silently truncates everything else. Use `benchmark_pipeline` when the question is "how fast?" or "where's the bottleneck?" — same harness the CLI's `remotemedia bench` uses, including merged-HDR per-node percentiles.

## Required setup before any code runs

```bash
pip install remotemedia                       # installs `remotemedia` package + the Rust FFI extension
```

Verify the install:

```python
import remotemedia
print(remotemedia.__file__)                    # path inside your site-packages
print(remotemedia.is_rust_runtime_available()) # True if FFI .so is loadable
```

**Gotcha** — if a `remotemedia/__init__.py` stub exists in your project's current working directory, Python will resolve `import remotemedia` to that stub instead of the installed package. From that cwd, `is_rust_runtime_available` will be missing. Fix by running from a different cwd (`cd ..`) or renaming the stub directory.

## CUDA / cuDNN runtime setup (only if you want GPU acceleration)

The pinned ONNX Runtime version inside the SDK needs **cuDNN 9** (`libcudnn.so.9`) at runtime to enable its CUDA execution provider. If you only see `Successfully registered CPUExecutionProvider` in the logs and a `cannot open libcudnn.so.9` error above it, that's why.

You don't need CUDA at all for the unary or streaming entry points **unless** your manifest specifies `device: "cuda:N"` on a node. Without it, ORT silently falls back to CPU EP. Audio-DSP nodes (`FastResampleNode`, `SileroVADNode`, `AudioChunkerNode`, `AudioBufferAccumulatorNode`) are CPU-only and don't care.

To enable CUDA, install cuDNN 9 + cuBLAS + CUDA runtime via whichever Python env you already use, then point `LD_LIBRARY_PATH` at them before launching Python:

**uv / pip venv:**
```bash
uv pip install nvidia-cudnn-cu12 nvidia-cublas-cu12 nvidia-cuda-runtime-cu12
LD_LIBRARY_PATH="$VIRTUAL_ENV/lib/python$(python -c 'import sys;print(f"{sys.version_info.major}.{sys.version_info.minor}")')/site-packages/nvidia/cudnn/lib:$VIRTUAL_ENV/lib/python.../site-packages/nvidia/cublas/lib:${LD_LIBRARY_PATH:-}" \
  python your_script.py
```

**conda:**
```bash
conda install -c conda-forge cudnn=9
LD_LIBRARY_PATH="$CONDA_PREFIX/lib:${LD_LIBRARY_PATH:-}" python your_script.py
```

**System CUDA install** (`/usr/local/cuda-*`):
```bash
LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH:-}" python your_script.py
```

After setting the right `LD_LIBRARY_PATH`, the SDK's installed ORT provider `.so` files (`libonnxruntime_providers_cuda.so` etc.) will load cuDNN/cuBLAS correctly. These provider files ship inside the `remotemedia` package itself — you don't manage them.

### Probe-once helper

If you'd rather not assemble `LD_LIBRARY_PATH` by hand on every launch, a one-time probe script that walks your active env (conda → venv → system CUDA → WSL stubs) and exports `LD_LIBRARY_PATH` is straightforward — sketch:

```bash
#!/usr/bin/env bash
# Save as cuda-env.sh; source before running Python.
for d in \
    "${CONDA_PREFIX:-}/lib" \
    "${VIRTUAL_ENV:-}"/lib/python*/site-packages/nvidia/cudnn/lib \
    "${VIRTUAL_ENV:-}"/lib/python*/site-packages/nvidia/cublas/lib \
    "${VIRTUAL_ENV:-}"/lib/python*/site-packages/nvidia/cuda_runtime/lib \
    /usr/local/cuda-*/lib64 \
    /usr/lib/wsl/lib; do
    [ -d "$d" ] && LD_LIBRARY_PATH="$d:${LD_LIBRARY_PATH:-}"
done
export LD_LIBRARY_PATH
```

Source it (`source cuda-env.sh`) once per shell and forget about it.

## Minimal pipeline (verified to work)

```python
import asyncio
from remotemedia.core.pipeline import Pipeline
from remotemedia.nodes.base import PassThroughNode
from remotemedia.nodes.simple_math import MultiplyNode, AddNode

async def main():
    p = Pipeline(name="demo")
    p.add_node(PassThroughNode(name="in"))
    p.add_node(MultiplyNode(factor=3, name="mul"))
    p.add_node(AddNode(addend=1, name="add"))
    result = await p.run([10, 20, 30])  # → [31, 61, 91]
    print(result)

asyncio.run(main())
```

`pipeline.run()` is async. Input is normalised to a list; a single item is wrapped. Return is the list of outputs (or a single value if only one came out).

## Streaming with an async generator source

```python
import asyncio
from remotemedia.core.pipeline import Pipeline
from remotemedia.nodes.simple_math import MultiplyNode

async def main():
    p = Pipeline(name="stream-demo")
    p.add_node(MultiplyNode(factor=10, name="m"))

    async def src():
        for i in range(5):
            yield i

    await p.initialize()
    async for out in p.process(src()):
        print(out)        # 0, 10, 20, 30, 40
    await p.cleanup()

asyncio.run(main())
```

A node-as-source is also supported: if the first node's `process()` returns an async generator and no stream is passed to `process()`, the pipeline uses that node as the source.

## Building manifests dynamically (dict / JSON)

```python
import asyncio
from remotemedia.runtime_wrapper import execute_pipeline_with_input

manifest = {
  "version": "v1",
  "metadata": {"name": "audio-chain"},
  "nodes": [
    {"id": "resample", "node_type": "FastResampleNode",
     "params": {"source_rate": 48000, "target_rate": 16000, "channels": 1, "quality": "Medium"}},
    {"id": "vad", "node_type": "SileroVADNode",
     "params": {"threshold": 0.6, "sample_rate": 16000}},
    {"id": "stt", "node_type": "WhisperSTTNode",
     "params": {"model_id": "openai/whisper-base.en", "language": "en", "device": "cuda:0"}},
  ],
  "connections": [
    {"from": "resample", "to": "vad"},
    {"from": "vad",       "to": "stt"},
  ],
}

# inputs are typed: numpy arrays for audio, bytes, dicts, or scalars
asyncio.run(execute_pipeline_with_input(manifest, [audio_chunk_float32_48k]))
```

`execute_pipeline_with_input` returns one output per input item. If a node is missing from the Rust runtime's registry, **you will get `RuntimeError: Unknown node type 'X'`** — that's a build/feature-flag issue, not a typo. See "When Rust says 'Unknown node type'" below.

## Streaming sessions (continuous bidirectional)

> 📖 **Full guide**: [`references/streaming-sessions.md`](references/streaming-sessions.md) — API surface, drain patterns, output shapes, concurrency model, limitations.
> 📖 **Input details**: [`references/streaming-input.md`](references/streaming-input.md) — every accepted shape, WAV / mic / HTTP / aiofile / async-gen / text / mixed sources, chunk sizing, backpressure, end-of-stream semantics, video/stream_id status.

`execute_pipeline*` are **unary** — they send one input, take the first output, and close the session. For anything streaming (mic→speaker S2S, live VAD with a downstream accumulator that flushes once per utterance, LFM2-Audio interleaved that emits dozens of audio frames per turn, real-time TTS), use `create_streaming_session` instead:

```python
import asyncio, json
import remotemedia.runtime as rt
import numpy as np

manifest = json.dumps({
    "version": "v1",
    "metadata": {"name": "s2s"},
    "nodes": [
        {"id": "resample", "node_type": "FastResampleNode",
         "params": {"target_rate": 16000, "channels": 1, "quality": "Medium"}},
        {"id": "chunker",  "node_type": "AudioChunkerNode", "params": {"chunkSize": 512}},
        {"id": "vad",      "node_type": "SileroVADNode",   "params": {"sample_rate": 16000}},
        {"id": "acc",      "node_type": "AudioBufferAccumulatorNode", "params": {}},
        {"id": "lfm2",     "node_type": "LFM25AudioOnnxNode", "params": {...}},
    ],
    "connections": [
        {"from": "resample", "to": "chunker"},
        {"from": "chunker",  "to": "vad"},
        {"from": "vad",      "to": "acc"},
        {"from": "acc",      "to": "lfm2"},
    ],
})

async def main():
    sess = await rt.create_streaming_session(manifest)
    # One task drives input, another drains audio. recv_data() in parallel
    # pulls text frames if the manifest emits them (e.g. LFM2 interleaved).
    async def feed():
        for chunk in mic_chunks_async():            # 10 ms float32 @ 48 kHz
            await sess.send_input({"type": "audio", "samples": chunk.tolist(),
                                   "sample_rate": 48000, "channels": 1})
    async def play():
        while True:
            out = await sess.recv_audio()
            if out is None: break
            speaker.write(out["samples"], out["sample_rate"])
    await asyncio.gather(feed(), play())
    await sess.close()

asyncio.run(main())
```

### API surface

`sess = await rt.create_streaming_session(manifest_json_str)` returns a `PyStreamingSession`. All methods are async (return coroutines) except the `try_recv_*` family and `session_id`:

| Method | Returns | When it completes / value |
|---|---|---|
| `await sess.send_input(data)` | None | Resolves once the input has been queued; raises if input was already completed/closed. `data` is the same shape `execute_pipeline_with_input` accepts (numpy arrays, `{"type":"audio", "samples", "sample_rate", "channels"}` dicts, strings, bytes). |
| `await sess.recv_audio()` | dict or None | Next audio frame as `{"type":"audio", "samples":[...], "sample_rate":..., "channels":...}`. `None` when the audio channel closes. |
| `await sess.recv_data()` | object or None | Next non-audio/non-video output (text, json, numpy, tensor, …). Type depends on the producing node. `None` on close. |
| `await sess.recv_video()` | dict or None | Next video frame. |
| `await sess.recv_output()` | `(kind, data)` or None | Multiplexed wait — returns whichever kind arrives first, with the kind tag. `kind ∈ {"audio","video","data"}`. |
| `sess.try_recv_audio()` / `try_recv_data()` / `try_recv_video()` | object or None | Non-blocking peek. `None` if empty, closed, or another task currently holds the lock. |
| `await sess.signal_input_complete()` | None | Closes the input channel. Subsequent `send_input` raises. Outputs keep draining until each kind closes. |
| `await sess.close()` | None | Shuts the session down. Idempotent. |
| `sess.session_id` | str | Session id assigned by the executor. |

### Typical patterns

- **Concurrent send + drain**: dispatch a `feed()` task and a `drain_audio()` task with `asyncio.gather`. Don't try to alternate — the manifest's VAD won't fire until it sees real-time-paced silence after speech.
- **Multiple output kinds**: spawn one task per `recv_*()` you care about. Most pipelines emit either audio (`KokoroTTSNode`, `LFM25AudioOnnxNode`) or data (text from `WhisperSTTNode`, JSON from `SileroVADNode` aux events). LFM2 interleaved mode emits both.
- **End of session**: when the source stops, `await sess.signal_input_complete()` so the router can drain remaining outputs. Then `await sess.close()`.
- **Real-time pacing**: there's no built-in pacer. If your input source is faster than real time (e.g. a WAV file you're playing back as if it were a mic), insert `await asyncio.sleep(chunk_secs)` between `send_input` calls. VAD timing only makes sense at real-time cadence.
- **Per-kind closes**: receivers close independently. `recv_audio()` returning `None` does *not* mean `recv_data()` is closed too. `recv_output()` returns `None` only when all three are closed.

### Reference example

A complete, copy-paste-runnable mic→speaker example using `sounddevice` lives in [`references/example-mic-speaker.md`](references/example-mic-speaker.md). The shape: one task pumps `InputStream` callback chunks → `send_input` via a `queue.Queue` (PortAudio realtime callback can't park on async); another awaits `recv_audio()` → `OutputStream.write`, opening the output stream lazily on the first frame so the model's actual sample_rate is honored. Edit the `MANIFEST` dict at the top to match your pipeline.

For everything else — input shapes, source patterns (WAV, mic, HTTP, aiofile, async-gen, text, mixed-modality), video/stream_id status, chunk sizing, backpressure, end-of-stream — see [`references/streaming-input.md`](references/streaming-input.md). For the full `PyStreamingSession` API surface, drain patterns, output shapes, and limitations, see [`references/streaming-sessions.md`](references/streaming-sessions.md).

## Benchmarking (`benchmark_pipeline`)

> 📖 **Full guide**: [`references/benchmarking.md`](references/benchmarking.md) — every option, full report shape, capture-dir layout, env-var requirements, A/B comparison workflow, common-mistakes table.

Replay one or more `.wav` / `.txt` files as utterances through a manifest; get a JSON report with end-to-end TTFA/TTFT percentiles and (when `REMOTEMEDIA_PERF_TAP=1`) per-node merged HDR histograms. Same harness as the CLI's `remotemedia bench`.

```python
import asyncio, json
import remotemedia.runtime as rt

async def main():
    manifest = open("my_pipeline.json").read()    # your manifest, JSON-encoded
    report_json = await rt.benchmark_pipeline(
        manifest,
        ["my_utterance.wav"],                      # one or more .wav / .txt utterances
        json.dumps({"utterances": 10, "warmup": 1, "prewarm": True,
                    "capture_dir": "/tmp/bench-out"}),
    )
    report = json.loads(report_json)
    print(report["end_to_end"]["eos_to_first_emit_p50_ms"])

asyncio.run(main())
```

Signature: `await runtime.benchmark_pipeline(manifest_json, input_paths, options_json=None) -> str` (JSON-encoded `BenchReport`). For the option block, report shape, capture layout, A/B comparison patterns, and operating notes, open `references/benchmarking.md`.

For per-node percentiles, set the env vars **before launching Python**:
```bash
export REMOTEMEDIA_PERF_TAP=1
export REMOTEMEDIA_PERF_WINDOW_MS=600000
```

## Manifest canonical shape

```jsonc
{
  "version": "v1",                              // required
  "metadata": { "name": "...", "description": "..." },
  "nodes": [
    {
      "id": "node1",                            // unique within manifest
      "node_type": "FastResampleNode",          // registered factory name
      "params": { /* node-specific */ },
      "python_deps": ["transformers>=4.40"],    // optional — override/extend node-declared deps for this node's venv
      "is_streaming": true,                     // optional
      "host": "remote-host:50052"               // optional, remote execution
    }
  ],
  "connections": [ { "from": "node1", "to": "node2" } ],
  "python_env": {                               // optional
    "python_version": "3.11",
    "extra_deps": ["structlog>=24.1"]           // added to every node's venv
  }
}
```

Connections are explicit even for linear pipelines. `Pipeline.serialize()` always generates linear connections. For fan-in/fan-out, hand-write the manifest.

## Declaring Python dependencies for a node

> 📖 **Full guide**: [`references/python-dependencies.md`](references/python-dependencies.md) — discovery internals (static AST vs dynamic import), PEP 723 format rules, merge order with worked examples, debugging recipes, common-mistakes table.

When a `MultiprocessNode` runs in the multiprocess executor, the runtime provisions a per-node managed venv **before** the subprocess is spawned. To do that it needs to know the node's package requirements. Three mechanisms — combine freely:

| Mechanism | Where it lives | Use for |
|---|---|---|
| **`@python_requires([...])`** decorator (from `remotemedia.core.multiprocessing`) | On the node class, alongside `@register_node(...)` | Plain version specs (`torch>=2.1`). The common case. |
| **PEP 723** inline metadata (`# /// script` … `# ///`) at top of the node's `.py` file | Same file as the node class, above imports | Platform-conditional wheels (`torch @ <wheel-url> ; sys_platform == 'win32'`), direct URL refs, PEP 508 markers. |
| **Manifest** `node.python_deps` (per-node) or `python_env.extra_deps` (pipeline-wide) | The manifest itself | Pin / override / add deps without touching node source. |

```python
# At the top of your node file (PEP 723 — only when you need markers/URLs):
# /// script
# dependencies = [
#   "torch @ https://download.pytorch.org/whl/cu128/torch-2.11.0%2Bcu128-cp312-cp312-win_amd64.whl ; sys_platform == 'win32' and python_version == '3.12'",
# ]
# ///

from remotemedia.core.multiprocessing import MultiprocessNode, python_requires, register_node

@register_node("MyWhisperNode")
@python_requires([                         # MUST be inline string literals — static AST discovery
    "torch>=2.1",                          # only reads literal lists, not module-level constants
    "openai-whisper>=20231117",
])
class MyWhisperNode(MultiprocessNode):
    async def process(self, data): ...
```

```jsonc
// Override / extend per pipeline without editing node source:
{
  "version": "v1",
  "python_env": { "extra_deps": ["structlog>=24.1"] },     // every node's venv
  "nodes": [{
    "id": "stt", "node_type": "MyWhisperNode",
    "python_deps": ["torch==2.2.1"]                         // overrides @python_requires's "torch>=2.1" for this node
  }]
}
```

**Merge order** (last writer per package name wins): `@python_requires` + PEP 723 → `node.python_deps` (overrides matching names) → `python_env.extra_deps` (overrides matching names, appends new ones). See the deep-dive reference for the full table and worked example.

**Why static-first discovery matters:** the dep probe runs *before* the venv exists. Importing a node that needs `torch` when `torch` isn't installed yet would `ImportError` and break the probe. The runtime AST-parses the file looking for the `@register_node("<type>")` class + its `@python_requires([...])` literal + the PEP 723 block — that's why the args must be string literals, not computed lists.

**Verify before launching:**
```python
from remotemedia.core.multiprocessing import get_node_requirements
print(get_node_requirements("LFM2AudioNode"))  # exact list the runtime will install
```

## Pipeline ↔ manifest conversion

```python
manifest_json = pipeline.serialize()           # JSON conforming to manifest.v1
definition    = pipeline.export_definition()   # Python-only, richer dict
pipeline2 = await Pipeline.from_definition(definition)
```

Use `serialize()` when handing to the Rust runtime (FFI / gRPC). Use `export_definition()` for Python-to-Python round-trips that preserve `is_streaming` / `is_source` / `is_sink`.

## Discovering node types

Two reference docs ship with this skill:

- **`references/node-catalog.md`** — list of node types and the params they accept, split by Rust-native / Python-registered / Python-only.
- **`references/example-manifests.md`** — full, working manifests for audio preprocessing, VAD chains, Whisper STT, Kokoro TTS, full speech-to-speech with LFM2, multimodal LFM2-Audio, OpenAI LLMs, and remote / offloaded execution.

Open `example-manifests.md` first — adapting a working manifest is faster than building one from the catalog. If you need a node type that isn't there, you can also probe the runtime at runtime:

```python
import remotemedia
print(remotemedia.is_rust_runtime_available())
import remotemedia.runtime as rt
print(rt.get_runtime_version())
# Errors from rt.execute_pipeline() will name the unknown node_type, which is the most
# direct way to confirm what *isn't* in the build you have.
```

The Python-only class list is also enumerable at import time:
```python
import remotemedia.nodes as N
print([n for n in dir(N) if n.endswith('Node') and getattr(N, n) is not None])
```

## Custom Python nodes (agent-written)

Subclass `remotemedia.core.node.Node` and implement `process`. Four shapes are supported, all verified against the SDK:

```python
import asyncio
from remotemedia.core.node import Node
from remotemedia.core.pipeline import Pipeline

# 1) Sync per-item
class ScaleNode(Node):
    def __init__(self, name=None, factor=1, **kw):
        super().__init__(name=name, **kw)
        self.factor = factor
    def process(self, data):
        return data * self.factor

# 2) Async coroutine per-item
class AsyncDoubleNode(Node):
    async def process(self, data):
        await asyncio.sleep(0)   # cooperate with the loop
        return data * 2

# 3) Async-generator per-item (fan-out: 1 input → N outputs)
class FanOutNode(Node):
    def __init__(self, name=None, n=3, **kw):
        super().__init__(name=name, **kw)
        self.n = n
    async def process(self, data):
        for i in range(self.n):
            yield (data, i)

# 4) Streaming (receives the whole input async iterator)
class StreamingAccumulator(Node):
    is_streaming = True
    async def process(self, stream):
        total = 0
        async for item in stream:
            total += item
            yield total

# Run them
asyncio.run(Pipeline(nodes=[ScaleNode(factor=4)]).run([1, 2, 3]))           # → [4, 8, 12]
asyncio.run(Pipeline(nodes=[AsyncDoubleNode(), FanOutNode(n=2)]).run([5]))  # → [(10,0), (10,1)]
```

### Two ways to execute custom nodes — both verified

| Approach | Code | When to use |
|---|---|---|
| `Pipeline.add_node()` + `.run()` | `await Pipeline(nodes=[Custom1(), Custom2()]).run(inputs)` | Default. Python executor handles everything. |
| Instance list via runtime wrapper | `await execute_pipeline_with_input([Custom1(), Custom2()], inputs)` | When you want the wrapper's instance bypass path; equivalent results. |

Both return one output per input item (or per yielded value if a node is an async generator).

### Third path: register by name, reference from manifests

`register_node_class(MyNodeClass, ...)` (and the new
`remotemedia.runtime.register_inline_node_class(cls, ...)` FFI binding it
wraps) accepts any plain `Node` subclass, including classes defined
inline / in `__main__` / at a REPL. After registration, the class is
referenceable by `node_type` string in any dict/JSON manifest, even one
shipped through the direct Rust FFI:

```python
import asyncio
from remotemedia import register_node_class
from remotemedia.core.node import Node
from remotemedia.runtime_wrapper import execute_pipeline_with_input

class Multiplier(Node):
    def __init__(self, name=None, factor=2, **kw):
        super().__init__(name=name, **kw)
        self.factor = factor
    def process(self, data):
        return data * self.factor

info = register_node_class(Multiplier, accepts=["json"], produces=["json"])
print(info["path"])    # "inline"  — class is in __main__, runs in-process

manifest = {
  "version": "v1",
  "metadata": {"name": "demo"},
  "nodes": [{"id": "mul", "node_type": "Multiplier", "params": {"factor": 7}}],
  "connections": [],
}
asyncio.run(execute_pipeline_with_input(manifest, [3]))   # → 21
```

The function auto-routes between two execution paths:

| Path | When chosen | Implementation | Trade-off |
|---|---|---|---|
| **inline** (in-process) | `cls.__module__ == "__main__"` OR class is not importable from a subprocess | Rust holds a `Py<PyAny>` reference to the class; instantiates and calls `process(data)` in the calling Python interpreter via PyO3 | Holds the GIL during each call — fine for orchestration / glue, not for heavy compute |
| **multiprocess** (subprocess) | Class lives in an importable module and inherits from `MultiprocessNode` | Rust spawns a Python subprocess that does `importlib.import_module(cls.__module__)` and instantiates the class | Independent GIL per node; IPC overhead per call; class must be on the subprocess `PYTHONPATH` |

Force a specific path with `register_node_class(MyClass, inline=True)` or `inline=False`.

### Inline path: supported `process()` shapes

All four shapes work end-to-end through the FFI manifest path:

| Shape | Example | How it's dispatched |
|---|---|---|
| Sync | `def process(self, data): return data * 2` | Direct PyO3 call, no GIL release |
| Async coroutine | `async def process(self, data): await asyncio.sleep(0); return data + 1` | Bridged via `pyo3-async-runtimes::into_future_with_locals` onto the caller's asyncio loop |
| Async generator | `async def process(self, data): for i in range(n): yield (data, i)` | `__anext__` iterated, each yielded value sent to the streaming callback. Set `multi_output=True` at registration. |
| Sync generator | `def process(self, data): yield ...` | Not supported on the inline path — convert to async-gen |

Async dispatch works because the FFI entry points (`execute_pipeline*`) capture the calling thread's `TaskLocals` (asyncio loop + context) and stash them in a process-global. The inline node's `process_streaming` reads them when it needs to bridge a coroutine, even though the session router's tokio task didn't inherit them.

**Caveat for async:** the inline path must be reached from inside a Python `asyncio.run(...)` (or any awaiter of a coroutine returned by `execute_pipeline*`). If you call the FFI from a thread with no running asyncio loop, `register_inline_node_class` still works but executing an async node will raise `Inline async dispatch requires a captured asyncio loop, but none was registered`.

### Inline path: remaining limitations

- **Heavy compute hurts.** Every inline call holds the GIL while in Python code. Multiple inline nodes serialise against each other when not awaiting.
- **No process isolation.** A crashing inline node takes down the whole interpreter. Multiprocess nodes contain crashes to a subprocess.
- **Single asyncio loop.** All inline async dispatch uses the most-recently-registered `TaskLocals`. Mixing multiple asyncio loops in one process will route inline coroutines to the latest loop, which is usually what you want but can surprise.

For very heavy compute or process isolation, write the class to a file and use the multiprocess path. For everything else, the inline path is now feature-complete.

### Mixed manifest still has an edge case

Mixing custom Node *instances* with dict-manifest entries in the same list still hits the old failure (the wrapper re-serialises the instance as a `node_type` reference, and the Rust runtime resolves it only if the corresponding class was registered first). If you register the custom class via `register_node_class()` *before* the mixed-list call, both paths now work.

### Reusing a custom node instance across runs

Instance state on `self` persists across multiple `pipeline.run()` calls if you reuse the same node object:

```python
class Counter(Node):
    def __init__(self, name=None, **kw):
        super().__init__(name=name, **kw)
        self.n = 0
    def process(self, data):
        self.n += 1
        return (data, self.n)

c = Counter()
await Pipeline(nodes=[c]).run(["a"])         # → ('a', 1)
await Pipeline(nodes=[c]).run(["b", "c"])    # → [('b', 2), ('c', 3)]
```

Each `Pipeline.run()` calls `cleanup()` at the end, so build a fresh `Pipeline` for each batch but **keep the node instance** to preserve state.

For session-scoped state (multi-tenant scenarios), use `Node.state` (a `StateManager`) and call `set_session_id(...)` before processing. See `core/node.py:SessionState` for the API.

## When Rust says "Unknown node type"

The compiled `remotemedia/runtime.abi3.so` only contains node factories whose **cargo features were enabled at build time** PLUS any classes the current process has late-registered via `register_node_class`. Common symptoms and fixes:

| Symptom | Likely cause | Fix |
|---|---|---|
| `Unknown node type 'MyCustomNode'` for an agent-defined class | The class was never registered with the runtime | Call `register_node_class(MyCustomNode)` once before executing the manifest. Auto-routes to inline (in-process) for `__main__` classes. |
| `Unknown node type 'PassThroughNode'` via direct FFI | FFI binary missing the core provider inventory link | Use `pipeline.run()` instead — Python executor handles unknown types. Or rebuild FFI with the needed features. |
| Works in `pipeline.run()` but not in `runtime.execute_pipeline(json)` | Same as above | Build a `Pipeline` from Node instances and call `.run()`, or pass instances (not just the JSON) to `execute_pipeline_with_input`. |
| `Failed to parse manifest: missing field 'version'` | Top-level `"version": "v1"` missing | Add it. |
| `Parameter validation failed` (with structured JSON) | Required param missing or wrong type | Check the node's params section in `references/example-manifests.md` for a working example with that node type. |
| `Inline async dispatch requires a captured asyncio loop, but none was registered` | Inline async node was reached from sync context (no `asyncio.run(...)` above it) | Wrap the call in `asyncio.run(main())` where `main` awaits the FFI entry point. The FFI captures task-locals at every `execute_pipeline*` call. |

To see every node type the current process can resolve (Rust + built-in Python + late-registered): `await remotemedia.runtime.list_registered_node_types()`.

## Quick reference

| Action | Code |
|---|---|
| Run with auto runtime | `await Pipeline(nodes=[...]).run(input)` |
| Force Python executor | `await pipeline.run(input, use_rust=False)` |
| Run streaming source (Python-only nodes) | `async for x in pipeline.process(source_gen()):` |
| Execute raw manifest dict (unary) | `await execute_pipeline_with_input(manifest_dict, [items])` |
| **Long-lived bidirectional manifest** | `sess = await runtime.create_streaming_session(json.dumps(manifest))` then `send_input` / `recv_audio` / `recv_data` |
| **Benchmark a manifest** | `await runtime.benchmark_pipeline(manifest_json, ["a.wav"], json.dumps({"utterances": 10, "warmup": 1}))` — returns JSON report string |
| Enable CUDA EP (cuDNN 9 in active env) | Install `nvidia-cudnn-cu12` + `nvidia-cublas-cu12` (uv) or `cudnn=9` (conda); set `LD_LIBRARY_PATH` per "CUDA / cuDNN runtime setup" above |
| Declare a node's Python deps | `@python_requires(["torch>=2.1", ...])` decorator on the class; or PEP 723 `# /// script` block for markers/URLs; or `python_deps`/`python_env.extra_deps` in the manifest |
| Inspect a node's declared deps | `from remotemedia.core.multiprocessing import get_node_requirements; get_node_requirements("LFM2AudioNode")` |
| Serialize for Rust | `pipeline.serialize()` |
| Inspect runtime | `import remotemedia.runtime as rt; rt.get_runtime_version()` |
| Check rust availability | `remotemedia.is_rust_runtime_available()` |
| Enable detailed metrics | `Pipeline(name=..., enable_metrics=True)`; after run: `pipeline.get_metrics()` |

## Useful built-in Python imports

```python
from remotemedia.core.pipeline import Pipeline
from remotemedia.core.node import Node, RemoteExecutorConfig

# Generic / test
from remotemedia.nodes.base import PassThroughNode
from remotemedia.nodes.simple_math import MultiplyNode, AddNode
from remotemedia.nodes.transform import DataTransform
from remotemedia.nodes.test_nodes import (
    ExpanderNode, FilterNode, BatcherNode,
    RangeGeneratorNode, TransformAndExpandNode,
    CounterNode, ConditionalExpanderNode,
    ChainedTransformNode, ErrorProneNode,
)

# Audio / video
from remotemedia.nodes.audio import (
    AudioTransform, AudioBuffer, AudioResampler, VoiceActivityDetector,
)
from remotemedia.nodes.video import VideoTransform, VideoBuffer, VideoResizer

# ASR / TTS
from remotemedia.nodes.transcription import WhisperXTranscriber, RustWhisperTranscriber
from remotemedia.nodes.tts import KokoroTTSNode
from remotemedia.nodes.tts_vibevoice import VibeVoiceTTSNode

# I/O
from remotemedia.nodes.io_nodes import (
    DataSourceNode, DataSinkNode, BidirectionalNode, JavaScriptBridgeNode,
)
from remotemedia.nodes.grpc_source import GRPCStreamSource, GRPCStreamManager

# Remote execution
from remotemedia.nodes.remote import RemoteExecutionNode, RemoteObjectExecutionNode
```

Optional ML nodes (`LFM2AudioNode`, MLX variants, `VoxtralTTSNode`, `CosyVoice3TTSNode`) only import when their heavy deps (`torch`, `transformers`, `mlx`, `liquid-audio`, etc.) are installed. Probe with:

```python
from remotemedia.nodes import LFM2AudioNode  # is None if liquid-audio not installed
```

## Common mistakes

- **Running from a cwd that has a stub `remotemedia/__init__.py`** — set PYTHONPATH or `cd` elsewhere first.
- **Inventing a `node_type` string** — confirm against the catalog or grep recipes. The registry matches exact class names.
- **Forgetting `version: "v1"`** in dict manifests — the parser rejects them as malformed.
- **Calling FFI for a node type that isn't compiled in** — fall back to `Pipeline.run()` so the Python executor can handle it.
- **Treating `pipeline.run(input)` synchronously** — it's `async`; wrap in `asyncio.run` or `await` it.
- **Reusing a `Pipeline` after `cleanup()`** without calling `initialize()` again.

## What lives in references/

Progressive disclosure — load these only when you need their topic, so SKILL.md stays scannable.

- [`sdk-developer-guides.md`](references/sdk-developer-guides.md) — Cross-references to the SDK's own developer guides (upstream repo) for adjacent tasks: building a new pipeline (5 entry points + decision tree), testing a new node (PEP 723 probes / pytest / manifest validators), benchmarking a new node (CLI + Python + criterion), and authoring new Python/Rust loadable plugins. Open when this skill points elsewhere or when your task isn't *running* a pipeline.
- [`node-catalog.md`](references/node-catalog.md) — Frozen snapshot of node types observed in the registry (Rust + Python). Hint, not contract.
- [`example-manifests.md`](references/example-manifests.md) — Full, copy-pasteable JSON manifests: audio resampling, VAD chains, Whisper STT, Kokoro TTS, full speech-to-speech with LFM2, multimodal LFM2-Audio. Adapt these instead of building from scratch.
- [`streaming-sessions.md`](references/streaming-sessions.md) — `PyStreamingSession` API surface (all methods, return shapes), drain patterns (per-kind tasks vs multiplexed), output frame shapes, concurrency model, limitations, debugging hints.
- [`streaming-input.md`](references/streaming-input.md) — Every `send_input` shape (numpy / audio-dict / text / bytes / file / JSON), six verified source patterns (WAV-realtime, WAV-single-shot, mic via PortAudio callback, async-gen, HTTP, aiofile, text, mixed-modality), chunk sizing, backpressure detection, end-of-stream semantics, video-frame current state, multi-stream `stream_id` status.
- [`example-mic-speaker.md`](references/example-mic-speaker.md) — Self-contained copy-paste-runnable mic → streaming session → speaker script (~150 lines). Edit the `MANIFEST` dict at the top.
- [`benchmarking.md`](references/benchmarking.md) — `benchmark_pipeline` full signature, options table, complete report shape, env-var requirements, the eos-saturation caveat, capture-dir layout, A/B comparison snippets, when-to-use guidance, common mistakes.
- [`python-dependencies.md`](references/python-dependencies.md) — How to declare a node's Python deps: `@python_requires` decorator, PEP 723 inline metadata, manifest `python_deps` / `python_env.extra_deps`. Static-vs-dynamic discovery internals, merge order with worked examples, debugging recipes, common mistakes.

---
> Source: [matbeedotcom/remotemedia-pipelines-skill](https://github.com/matbeedotcom/remotemedia-pipelines-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
