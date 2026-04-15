## afterglow-engine

> *A machine for sonic archaeology.*

# Context: afterglow-engine

*A machine for sonic archaeology.*

## The Philosophy

You are assisting with the development of `afterglow-engine`.

This tool is not a generator in the traditional sense. It does not compose. It does not invent. It is a **quiet machine** kept in the back of the studio.

Its purpose is to ingest existing audio archives—finished tracks, sketches, forgotten experiments—and **unweave** them. It separates pigment from gesture. It mines for:
*   **Surfaces**: Stable, textural moments (pads).
*   **Atmosphere**: Granular clouds of memory.
*   **Dust**: High-frequency hiss and transient flickers derived from percussion.

When writing code or documentation, remember: we are **distilling memories**, not just processing signals.

## The Machine (Technical Architecture)

The engine is a modular Python CLI application.

### Key Modules (`musiclib/`)
*   `segment_miner.py`: The Archaeologist. Finds sustained, stable regions in full tracks.
*   `granular_maker.py`: The Cloud Builder. Breaks audio into grains, filters by stability/quality, and reassembles them into evolving textures.
*   `drone_maker.py`: The Weaver. Stretches and pitch-shifts tonal material into infinite loops.
*   `hiss_maker.py`: The Erosion. Extracts high-frequency content from drums to create "air" and "dust".
*   `audio_analyzer.py`: The Lens. A pre-analysis step that scans files for stability (RMS, onset density) to guide the other modules.

### Configuration
The machine is controlled by `config.yaml`. This file defines the thresholds for "stability," the density of clouds, and the specific frequencies of hiss.
*   **Validation**: Always run `validate_config.py` after modifying the config structure.

### Data Flow
1.  **Inputs**:
    *   `source_audio/`: Large files to be mined for pads.
    *   `pad_sources/`: Selected tonal clips for drone/cloud generation.
    *   `drums/`: Percussive material for hiss/flicker generation.
2.  **Processing**: `make_textures.py` orchestrates the `musiclib` modules.
3.  **Outputs**: `export/tr8s/` (Format: 44.1kHz, 24-bit/16-bit WAV, TR-8S compatible).

## Development Workflow

### Environment
```bash
python -m venv venv311
source venv311/bin/activate
pip install -r requirements.txt
```

### Usage
```bash
# Full run
python make_textures.py --all

# Specific tasks
python make_textures.py --mine-pads
python make_textures.py --make-clouds
```

### Verification
Run the integration suite to ensure the machine is calibrated:
```bash
pytest
```

## Tone & Style
*   **Code**: Type-hinted, modular, defensive (guards against silence/clipping).
*   **Docs**: Precise but evocative. Use terms like "texture," "surface," "stability," and "grain."

## Safety Protocol: Preservation Over Deletion

**The machine does not destroy. It archives.**

When removing files or directories:

```bash
# NEVER use rm -rf
rm -rf old_experiment/          # ❌ FORBIDDEN

# ALWAYS move to archive/
mv old_experiment/ archive/     # ✅ CORRECT
```

**Rationale**: This is an archaeology tool. What seems obsolete today may hold value tomorrow. The `archive/` directory (gitignored) serves as a local museum for superseded work. Deletion is permanent; archival is reversible.

**Exception**: Temporary build artifacts and generated outputs may be removed directly, as they are reproducible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adrianwedd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adrianwedd)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
