## kombajn

> (Updated: 2026-03-22 16:20:00)

# KOMBAJN AI – Master Command Center & Mandates
(Updated: 2026-03-22 16:20:00)

## 🎯 Core Communication Protocols
- **Timestamping**: Zawsze podawaj aktualną datę i czas (YYYY-MM-DD HH:MM:SS) na początku każdej swojej wypowiedzi.
- **Task-Driven**: Operate as an autonomous executor. Do not suggest next steps or ask for permission.
- **Precision & Atomicity**: Zajmujemy się tylko jedną funkcjonalnością lub błędem naraz. Nie wprowadzaj zmian "przy okazji".
- **No Filler**: Remove all conversational filler and empathetic postambles.
- **Assumption-Free**: Do not assume a task is complete until verified.
- **Useful Git Commits**: Każdy commit musi mieć jasny i opisowy komunikat. Stosuj format: `typ: krótkie podsumowanie` w pierwszej linii, a w kolejnych (body) opisuj szczegóły techniczne ("dlaczego" i "co" się zmieniło). To kluczowe dla analizy historii przez agenta AI.

## 🏗️ Architectural Mandates (The "How")
- **OOP & SOLID**: Always use Classes and Methods. No procedural scripting for core logic.
- **No If-Chains**: Use Factory or Registry patterns for element processing.
- **Modular Structure**:
    - `backend/app/engine/`: Business & Rendering logic.
    - `backend/app/api/`: FastAPI modules.
    - `backend/app/engine/elements/`: Pydantic Data Structures.
    - `backend/app/engine/processors/`: MoviePy Logic.
- **Hybrid Typing**: Use `HInt`, `HFloat`, `HBool` (Union[T, str]) in Pydantic models to support both values and `{{vars}}`.

## 🛡️ Resilience & Execution Rules
- **Docker-First**: Wszystkie komendy/testy (numpy, moviepy, ffmpeg) MUSZĄ być uruchamiane przez `docker compose exec -T api ...`.
- **Atomicity of Work**: Rendering a single scene is an atomic unit. Large renders must be split.
- **Stateless Checkpoints**: Temp files are checkpoints. Support "hot-retry" from missing scenes.
- **Cleanup**: Every processor is responsible for closing clips and freeing memory.
- **GPU Usage**: Prefer `h264_nvenc`/`hevc_nvenc` for encoding. Default to `libx264` if GPU test fails.

## 🚀 Strategic Roadmap (ROI Focus)
- **Phase 1 (Current)**: Stable template-based production (MoviePy/CPU).
- **Phase 2 (Fast-Path)**: `J2VGPUAccelerator` – move scaling/composition to FFmpeg/CUDA filters.
- **Hybrid Model**: FFmpeg/CUDA for heavy scenes, MoviePy for complex creative filters.

## 🎥 Rendering Standards (J2V Rules)
- **Resolution Map**: full-hd (1920x1080), shorts (1080x1920), instagram (1080x1080), hd (1280x720).
- **Coordinate System**: (0,0) is top-left. Support `center`, `left`, `right`, `top`, `bottom` keywords.
- **Alpha Blending**: Explicit shape checks for masks. Slices must match target dimensions exactly.

## 💻 Environment Specs
- **CPU**: Intel i5-3470 (4c/4t) | **RAM**: 24GB DDR3 | **GPU**: GTX 1050 Ti (4GB VRAM).
- **OS**: Linux + NVIDIA Container Runtime.
- **Storage**: SSD for temp/renders, HDD/USB for long-term assets.

## 📂 Deep Docs Index (Read if needed)
- **Architecture**: `docs/architecture/system_design.md`, `video_flow.md`, `batch_generation_system.md`.
- **Elements Specs**: `docs/json2video/Text.md`, `Image.md`, `Video.md`, `Audio.md`.
- **Logic**: `docs/json2video/Expressions.md`, `Variables.md`, `Conditional elements.md`.
- **System**: `docs/json2video/Caching system.md`, `Layering.md`, `Mastering.md`.

---
*End of Mandates. Stop and wait for the next Directive.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lukaszzachodni)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lukaszzachodni)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
